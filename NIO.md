## BIO

阻塞同步通信模型，一个线程处理一个请求

![bio](https://github.com/einQimiaozi/java_notebook_recite_version/blob/main/img/0034.png)

客户端同过tcp三次握手和服务器连接，acceptor用来处理客户端的请求(io直接在stream中操作，由acceptor顺序处理)，io如果没有就绪将阻塞，所以acceptor每次只能处理一个连接并且当这个连接执行时间过长或者阻塞时整个通信模型也将阻塞，io就绪后将每个请求分派给一个线程，可以通过多线程改善阻塞问题，即一个线程处理一个socket链接，线程顺序执行，因为频繁new Thread会造成线程数量爆炸，而且当某个socket套接字建立连接后长期没有io事件就绪则该线程会长时间空闲，所以后期加入了线程池用于减少线程对象的创建

特点：
  - 同步，不存在线程安全问题
  - 阻塞，一个线程一次只能处理一个客户端请求
  - 消耗大，当客户端请求较多时，会开启大量线程等待客户端的读事件，如果读事件不到来则线程阻塞，所以会有大量线程等待造成cpu占用率巨大

## NIO

非阻塞同步通信模型，一个线程处理多个请求

![nio](https://github.com/einQimiaozi/java_notebook_recite_version/blob/main/img/0035.png)

通过一个用户线程轮询多个channel，一个channel和一个socket连接绑定，channel是一个双工数据传输通道(和stream差不多)，不存储数据，只搬运数据，所以需要配合buffer使用，channel有数据则会放入buffer并交给一个线程进行处理，如果没有数据也不会阻塞，本质上在io部分依然是单线程

缺点：
  - 建立连接请求后客户端长时间没有数据到来会使得服务器端一直处于轮询状态造成cpu空转

## io多路复用

非阻塞同步通信模型，将线程进行拆分

![duiolufuyong](https://github.com/einQimiaozi/java_notebook_recite_version/blob/main/img/0036.png)

客户端请求通过Selector统一管理，事件驱动，即如果channel没有读写事件，那么Selector会阻塞，一旦任何一个或多个channel发生事件，Selector才会释放阻塞并处理该事件，所以可以避免cpu空转，是一种高效的nio模型

细节：
  - 1.NIO从其设计思想上来看，目的是为了尽可能多的处理请求，而不是加快处理请求的速度
  - 2.nio中的selector一般由一个内核线程担任，所以速度更快

适用场景：短连接，小数据

## AIO

非阻塞异步

![aio](https://github.com/einQimiaozi/java_notebook_recite_version/blob/main/img/0037.png)

当进行读写操作时，只须直接调用API的read或write方法即可。这两种方法均为异步的，对于读操作而言，当有流可读取时，操作系统会将可读的流传入read方法的缓冲区，并通知应用程序；对于写操作而言，当操作系统将write方法传递的流写入完毕时，操作系统主动通知应用程序。  即可以理解为，read/write方法都是异步的，完成后会主动调用回调函数，所以aio完成一次事件，需要用到两个线程，一个用于执行用户的read/write方法，另一个负责处理事件并执行回调函数将结果主动交回给用户线程

## select

select是java中对NIO的一种实现方法

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
  - 1.每次wait时需要拷贝fd集，所以需要频繁切换内核态和用户态
  - 2.轮询fd的时间复杂度为o(n)，较慢，所以select最大监听描述符为1024个

## poll

poll和select差不多，唯一的区别就是对fd集的描述不同，poll使用pollfd,select使用fd_set

## epoll

epoll提供了三个方法，而不是一个，epoll_create,epoll_ctl和epoll_wait
  - epoll_create:epoll中使用红黑树结构存储文件描述符和对应的事件，这样可以实现o(logn)级别的文件描述符快速查找，该方法就是用来创建该红黑树的，即epoll的文件描述符，参数为文件描述符的最大监听数量，不使用时必须使用close()方法关闭，不然会一直占用资源
  - epoll_ctl:该方法将感兴趣的事件和对应的文件描述符注册到红黑树中，当事件就绪时，将其节点的引用放入rdlist，每次注册会把整个fd集合一次性拷贝到内核空间，而不是每次在wait的时候拷贝(这意味着进程也只需要挂到eventpoll中一次，不需要每次都挂)
  - epoll_wait:轮询红黑树，等待内核发来的rdlist，执行其中所有的请求

实现：
  - epoll使用eventpoll作为中介，eventpoll管理fd和进程之间的关系
  - 进程监听某个fd时，将自己注册到eventpoll下，这样不需要每次遍历整个fd集，这一步是epoll_ctl完成
  - 当某个fd感兴趣的事件就绪后，会把红黑树中对应的节点的引用放入一个rdlist中，这样每次不需要遍历整个红黑树轮询就绪状态的fd，只需要轮询rdlist是否为空即可，不为空就分派rdlist中就绪的fd给监听它的进程，这一步由epoll_wait完成

优点：
  - 由于直接从rdlist中取文件描述符，其复杂度为O(1)
  - 无监听最大数量限制
  - 不需要多次切换内核态和用户态

## 水平触发(LT)和边缘触发(ET)

LT：当读写事件未全部完成时，每次epoll_wait都会返回通知，它会降低关心的fd的效率

ET：当读写事件未全部完成时，只有第一次epoll_wait会返回通知，之后如果没有新的读写事件到来，则不会再次通知，会提高关心fd的效率，但可能造成读写事件无法全部执行完毕

## Reactor

reactor是一种类似nio的设计模式，分为单线程单reactor，多线程单reactor，多线程多reactor

1.单线程单reactor

![1](https://pic3.zhimg.com/80/v2-bbb065565f6e53aa35b722d0fed9e9d2_720w.jpg)

Reactor用于处理socket，连接请求交给Accptor，io请求交给多个Handler

缺点：由于Handler是单线程，一旦一个Handler阻塞，后续其他Handler就都会阻塞

2.多线程单reactor

![2](https://pic2.zhimg.com/80/v2-e85e1cef0b6908ae13f84b76e81f1d85_720w.jpg)

使用线程池多线程处理多个读写请求替换Handler单线程处理，由reactor统一处理socket回写，这个和nio就比较相似了，而且使用多线程的话性能还好点，解决了Handler阻塞的问题

缺点：socket的io仍然由一个reactor单线程处理，短时间内高并发抗不住

3.多线程多reactor

![3](https://pic3.zhimg.com/80/v2-477021613f3539fe3d1b5023fc7974be_720w.jpg)

reactor的io和连接分离，mainireactor专门处理连接，将连接建立的socketChannel注册到subreactor上，subreactor根据socketChannel进行读写分离，其他交给线程池处理

## Netty

Netty是一个异步事件驱动的网络应用程序框架，用于快速开发可维护的高性能协议服务器和客户端，结构类似多线程多Reacotor

![netty](https://pic4.zhimg.com/80/v2-7eefba893a65706eb6bbe4115cbd0b83_720w.jpg)

1.NioEventGroup:一个轮询事件状态并处理的nio，每个NioEventGroup就是一个线程，Selector就是nio里的那个，TaskQeueu是一个异步队列，selector如果某个任务阻塞，就放入taskqueue中防止selector不能继续轮询

2.BossGroup：等同于mainReactor，本质是个线程池，里面有多个NioEventGroup

3.WorkerGroup：等同于subReactor，本质也是个线程池，NioEventGroup会把就绪的事件分发给pipeline，pipeline再顺序进行Handler处理

4.NioEventLoop：相当与一个本地java线程的抽象，循环轮询多个Channel处理io事件(也就是说一个NioEventLoop可能服务于多个channel)，其线程是基于concurrent包实现的，事件的执行顺序是按照FIFO的原则

netty的使用中一般要手动初始化workergroup和bossgroup，这俩东西在netty里实际上都是NioEventGroup

netty常用的一个场景就是对长连接进行心跳检测
