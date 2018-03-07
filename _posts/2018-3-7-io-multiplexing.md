---
layout: post
title:  "I/O复用模型之select、poll、epoll对比"
tag:
- Linux
- 操作系统
comments: true
---

在网络服务中，I/O复用的目的是使一个进程得以同时处理多个socket。一共有三种常见的机制可以实现这一模型：`select`、`poll`、`epoll`。

## select
`select`函数的原型为
```c
int select (int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```
`select`的过程简单地概括起来是这样的：首先注册要监控的socket及相应的事件（读、写或异常，分别对应函数参数中的`readfds`、`writefds`、`exceptfds`三个文件描述符集合`fd_set`），然后不断地对这些socket进行轮询，一旦某个socket发生了对应的事件，或者设置的等待时间（`timeout`）耗尽了，`select`函数就会修改对应的`fd_set`并返回，然后用户进程可以通过遍历`fd_set`来确定哪些socket发生了相应的事件。

这样的机制存在一些问题：

- 每次调用`select`都需要把被监控的文件描述符从用户空间拷贝到内核空间，为了减少数据拷贝带来的性能损坏，内核对被监控的文件描述符集合大小做了限制，并且这个是通过宏控制的，大小不可改变。
- 需要在返回后通过遍历文件描述符来获取已经就绪的socket。事实上，同时连接的大量客户端在一时刻可能只有很少处于就绪状态，因此随着监视的描述符数量的增长，其效率也会线性下降。

## poll
`poll`函数的原型为
```c
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```
虽然号称是`select`的改进版，但其实大同小异，只是改用`pollfd`数组来表示文件描述符，解决了文件描述符集合大小限制的问题，性能方面并没有多少提升，显得有些鸡肋。

## epoll
是`select`和`poll`的增强版本。相对于`select`和`poll`来说，`epoll`更加灵活，没有描述符限制。`epoll`使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的复制只需一次。

在`select`/`poll`中，进程只有在调用一定的方法后，内核才对所有监视的文件描述符进行扫描，而`epoll`事先通过`epoll_ctl()`来注册一个文件描述符，一旦基于某个文件描述符就绪时，内核会采用类似callback的回调机制，迅速激活这个文件描述符，当进程调用`epoll_wait()`时便得到通知。`epoll`的IO效率不会随着监视描述符的数量的增长而下降：`epoll`不同于`select`和`poll`轮询的方式，而是通过每个描述符定义的回调函数来实现的，只有就绪的描述符才会执行回调函数。
因此，如果没有大量的idle connection或者dead connection，`epoll`的效率并不会比`select`/`poll`高很多，但是当遇到大量的idle connection，就会发现`epoll`的效率大大高于`select`/`poll`。

下面是实际使用`epoll`的Python代码，相关说明见注释：
```python
import socket, select

EOL1 = b'\n\n'
EOL2 = b'\n\r\n'
response  = b'HTTP/1.0 200 OK\r\nDate: Mon, 1 Jan 1996 01:01:01 GMT\r\n'
response += b'Content-Type: text/plain\r\nContent-Length: 13\r\n\r\n'
response += b'Hello, world!'

serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
serversocket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
serversocket.bind(('0.0.0.0', 8880))
serversocket.listen(1)
serversocket.setblocking(0)     # use nonblocking mode
print("listening")

epoll = select.epoll()
epoll.register(serversocket.fileno(), select.EPOLLIN)   # register `read` events

try:
    connections = {}; requests = {}; responses = {}      # `connection` dictionary maps fd to their corresponding connection objects
    while True:
        # query the epoll object to find out if any events of interest may have occurred. 
        events = epoll.poll(1)    # `1` means waiting up to 1 second for such an event to occur.
        for fileno, event in events:
            if fileno == serversocket.fileno():    # read event on the server socket
                connection, address = serversocket.accept()
                connection.setblocking(0)   # set new socket to non-blocking mode
                epoll.register(connection.fileno(), select.EPOLLIN)
                connections[connection.fileno()] = connection 
                requests[connection.fileno()] = b''
                responses[connection.fileno()] = response
            elif event & select.EPOLLIN:   # read event on connection socket
                requests[fileno] += connections[fileno].recv(1024)
                if EOL1 in requests[fileno] or EOL2 in requests[fileno]:
                    epoll.modify(fileno, select.EPOLLOUT)    # register interest in write(EPOLLOUT) events when the complete request has been received
                    print('-'*40 + '\n' + requests[fileno].decode()[:-2])
            elif event & select.EPOLLOUT:  # write event on a connection socket
                byteswritten = connections[fileno].send(responses[fileno])
                responses[fileno] = responses[fileno][byteswritten:]
                if len(responses[fileno]) == 0:     # the complete response has been sent
                    epoll.modify(fileno, 0)  # disable interest in further events
                    connections[fileno].shutdown(socket.SHUT_RDWR)
            elif event & select.EPOLLHUP:  # hang-up(EPOLLHUP) event indicates disconnected client socket
                epoll.unregister(fileno)
                connections[fileno].close()
                del connections[fileno]
finally:
    epoll.unregister(serversocket.fileno())
    epoll.close()
    serversocket.close()
```

### 水平触发模式与边缘触发模式
`epoll`对文件描述符的操作有两种模式：水平触发（LT，level trigger）和边缘触发（ET，edge trigger）。

- LT模式：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，**应用程序可以不立即处理该事件**。下次调用epoll_wait时，会再次响应应用程序并通知此事件。
- ET模式：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，**应用程序必须立即处理该事件**。如果不处理，下次调用epoll_wait时，不会再次响应应用程序并通知此事件。

默认情况下是LT模式，如上述代码，下面是边缘触发的代码：

```python
import socket, select

EOL1 = b'\n\n'
EOL2 = b'\n\r\n'
response  = b'HTTP/1.0 200 OK\r\nDate: Mon, 1 Jan 1996 01:01:01 GMT\r\n'
response += b'Content-Type: text/plain\r\nContent-Length: 13\r\n\r\n'
response += b'Hello, world!'

serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
serversocket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
serversocket.bind(('0.0.0.0', 8880))
serversocket.listen(1)
serversocket.setblocking(0)

epoll = select.epoll()
# set ET mode `(select.EPOLLET)`
epoll.register(serversocket.fileno(), select.EPOLLIN | select.EPOLLET)

try:
    connections = {}; requests = {}; responses = {}
    while True:
        events = epoll.poll(1)
        for fileno, event in events:
            if fileno == serversocket.fileno():
                try:
                    # use a loop with aim to process all the events at a single time, since they won't be notified again
                    while True:  
                        connection, address = serversocket.accept()
                        connection.setblocking(0)
                        epoll.register(connection.fileno(), select.EPOLLIN | select.EPOLLET)
                        connections[connection.fileno()] = connection
                        requests[connection.fileno()] = b''
                        responses[connection.fileno()] = response
                except socket.error:
                    pass
            elif event & select.EPOLLIN:
                try:
                    while True:
                        requests[fileno] += connections[fileno].recv(1024)
                except socket.error:
                    pass
                if EOL1 in requests[fileno] or EOL2 in requests[fileno]:
                    epoll.modify(fileno, select.EPOLLOUT | select.EPOLLET)
                    print('-'*40 + '\n' + requests[fileno].decode()[:-2])
            elif event & select.EPOLLOUT:
                try:
                    while len(responses[fileno]) > 0:
                        byteswritten = connections[fileno].send(responses[fileno])
                        responses[fileno] = responses[fileno][byteswritten:]
                except socket.error:
                    pass
                if len(responses[fileno]) == 0:
                    epoll.modify(fileno, select.EPOLLET)
                    connections[fileno].shutdown(socket.SHUT_RDWR)
            elif event & select.EPOLLHUP:
                epoll.unregister(fileno)
                connections[fileno].close()
                del connections[fileno]
finally:
    epoll.unregister(serversocket.fileno())
    epoll.close()
    serversocket.close()
```
可以看到，与LT模式代码最大的不同在于多了许多“try+while”的结构，为的是一次性把就绪的描述符处理完毕，因为相关的通知只会触发一次。这样的设计在很大程度上减少了`epoll`事件被重复触发的次数，因此效率要比LT模式高。

## 参考
- <https://segmentfault.com/a/1190000003063859>
- <http://scotdoyle.com/python-epoll-howto.html>
- <https://cloud.tencent.com/developer/article/1005481>
