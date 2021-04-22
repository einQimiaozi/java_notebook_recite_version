## BIO

阻塞同步通信模型，一个线程处理一个请求

![bio](https://pcsdata.baidu.com/thumbnail/8a9257fd0k700cf216d47bfc4ad8737b?fid=1508469986-16051585-733691224096521&rt=pr&sign=FDTAER-yUdy3dSFZ0SVxtzShv1zcMqd-mxUIvv59fM0Lg%2BONaWLpAwPTQ7A%3D&expires=2h&chkv=0&chkbd=0&chkpc=&dp-logid=1275034477&dp-callid=0&time=1619085600&size=c1600_u1600&quality=100&vuk=-&ft=video)

客户端同过tcp三次握手和服务器连接，acceptor用来处理客户端的请求(io直接在stream中操作)，将每个请求分派给一个线程，线程顺序执行，因为频繁new Thread会造成性能问题，后期加入了线程池用于减少线程对象的创建

特点：
  - 同步，不存在线程安全问题
  - 阻塞，一次只能处理一个客户端请求

## NIO

非阻塞同步通信模型，一个线程处理多个请求

![nio](https://pcsdata.baidu.com/thumbnail/d9000ce1aq06a9782c479101c29f6e66?fid=1508469986-16051585-254446546933029&rt=pr&sign=FDTAER-yUdy3dSFZ0SVxtzShv1zcMqd-%2F%2BwD%2FY5%2Bo1hExLJI7%2B%2BgGczCwlQ%3D&expires=2h&chkv=0&chkbd=0&chkpc=&dp-logid=1401519158&dp-callid=0&time=1619085600&size=c1600_u1600&quality=100&vuk=-&ft=video)

客户端请求通过Selector统一管理，Selector另外管理多个在其上注册的channel(双向通道，读写可同时进行，一般分为两类，网络io和文件io，io在缓冲区中操作，不直接写入到流中了)，Selector不断轮询channel集合，一旦轮询到当前的channel处于就绪(空闲)状态，则分派任务给channel执行，所以服务端这边只需要提供一个线程管理Selector就可以，做到非阻塞同步通信

细节：
  - 1.io操作使用一个Buffer数组作为缓冲区，不直接在stream中操作
  - 2.io模型通过ScoketChannel过河ServerSocketChannel(一个阻塞一个非阻塞)来完成套接子通道的实现，不需要tcp三次握手建立链接
  - 3.NIO从其设计思想上来看，目的是为了尽可能多的处理请求，而不是加快处理请求的速度

适用场景：短连接，小数据

## AIO

非阻塞异步

简单来说AIO这个东西是使用系统api提供的read和write实现的，它会返回一个回调函数，执行外读写之后会直接调用回调函数，其socket，channel和文件都是异步实现

## select

select是java中对NIO的一种实现框架

概念:
  - fd_set:文件描述符集
  - 等待队列：进程想要监听一个文件描述符，需要将自己加入到该文件描述符的等待队列下，该操作需要将文件描述符从用户空间拷贝到内核空间

select实现：
  - 每次将fd_set作为参数传入
  - 将fd_set从用户空间拷贝到内核空间，并注册系统的回调函数poll_wait
  - 遍历fd集，使用每个fd的poll方法(这个poll方法和后面的poll不是一个！！！)调用回调函数poll_wait将当前进程挂载(注意是挂载，不是拷贝)到该fd的等待队列下对该fd进行监听并阻塞该进程
  - 多路复用器不断轮询fd集，若有任何fd处于就绪状态，则将其等待队列中的进程移出唤醒并从该fd的缓冲区中取出数据开始执行任务，注意，这里还会遍历整个fd集，把当前要执行的进程从它监听的其他fd中也移出
  - 执行完毕后将fd集从内核空间拷贝回用户空间

缺点：
  - 1.每次拷贝fd集都需要切换内核态和用户态
  - 2.轮询fd的时间复杂度为o(n)，较慢，所以select最大监听描述符为1024个

## poll

poll和select差不多，唯一的区别就是对fd集的描述不同，poll使用pollfd,select使用fd_set

## epoll




