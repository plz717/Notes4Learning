#Socket编程
Socket是进程通讯的一种方式，即调用这个网络库的一些API函数实现分布在不同主机的相关进程之间的**数据交换**。
几个定义：
-   IP地址：即依照TCP/IP协议分配给本地主机的网络地址，两个进程要通讯，任一进程首先要知道通讯对方的位置，即对方的IP。
-   端口号：用来辨别本地通讯进程，一个本地的进程在通讯时均会占用一个端口号，不同的进程端口号不同，因此在通讯前必须要分配一个没有被访问的端口号。
-   连接：指两个进程间的通讯链路。
-   半相关：网络中用一个三元组可以在全局唯一标志一个进程：（协议，本地地址，本地端口号）这样一个三元组，叫做一个半相关,它指定连接的每半部分。
-   全相关：一个完整的网间进程通信需要由两个进程组成，并且只能使用同一种高层协议。也就是说，不可能通信的一端用TCP协议，而另一端用UDP协议。因此一个完整的网间通信需要一个五元组来标识：（协议，本地地址，本地端口号，远地地址，远地端口号）这样一个五元组，叫做一个相关（association），即两个协议相同的半相关才能组合成一个合适的相关，或完全指定组成一连接。


##客户/服务器模式过程
-   ####**服务器端**
其过程是首先**服务器方要先启动**，并根据请求提供相应服务：
（1）打开一通信通道并告知本地主机，它愿意在**某一公认地址上的某端口**接收客户请求；
（2）等待客户请求到达该端口；
（3）接收到客户端的服务请求时，处理该请求并发送应答信号。接收到并发服务请求，要**激活一新进程**来处理这个客户请求（如UNIX系统中用fork、exec）。新进程处理此客户请求，并不需要对其它请求作出应答。服务完成后，关闭此新进程与客户的通信链路，并终止。
（4）返回第（2）步，等待另一客户请求。
（5）关闭服务器

-   ####**客户端**
（1）打开一通信通道，并连接到**服务器所在主机的特定端口**；
（2）向服务器发服务请求报文，等待并接收应答；继续提出请求......
（3）请求结束后关闭通信通道并终止。


##socket如何使用？    
> **服务端**：建立监听套接字(socket())，绑定地址和端口号(bind())，然后开始监听(listen())，等待客户机连接(accept())，生成一个响应套接字，负责处理这个连接请求，然后读写信息(read/write)，关闭响应套接字，关闭监听套接字(close())。 
**客户端**：需要先建立套接字(socket())，然后申请连接服务器(connect())，需要知道服务端的ip和端口号，而不需要指定客户端的（客户端会由os分配），连接上服务器后开始读写信息(read/write)，最后关闭套接字(close())。       

下图显示了使用流套接字的面向连接的通信：
![使用流套接字的面向连接的通信](https://docs.oracle.com/cd/E38902_01/html/E38880/figures/7099.png)

##常用函数介绍

-   **int s = socket(int af, int type, int protocol);**
此调用创建流套接字。
1) af 为地址族（Address Family），也就是 IP 地址类型，常用的有 AF_INET 和 AF_INET6。AF_INET 表示 IPv4 地址，例如 127.0.0.1；AF_INET6 表示 IPv6 地址，例如 1030::C9B4:FF12:48AA:1A2B。
在LINUX系统中，AF等价于PF
2) type就是socket的类型，对于AF_INET协议族而言有流套接字(SOCK_STREAM)、数据包套接字(SOCK_DGRAM)、原始套接字(SOCK_RAW)。
3) protocol 表示传输协议，常用的有 IPPROTO_TCP 和 IPPTOTO_UDP，分别表示 TCP 传输协议和 UDP 传输协议。当protocol设置为0时,代表让系统自动根据af和type推演出协议类型,一般情况下有了 af 和 type 两个参数, 操作系统会自动推演出协议类型，除非遇到这样的情况：**有两种不同的协议支持同一种地址类型和数据传输类型**。此时,如果我们不指明使用哪种协议，操作系统是没办法自动推演的。

-  **bind (SOCKET s, name, namelen);**
将套接字绑定到地址（包含ip和端口号）后，远程进程才能引用此套接字。
用法：
		struct sockaddr_in6 sin6;//初始化一个表示网络地址的结构体
		sin6.sin6_family = AF_INET6;//所用协议
		sin6.sin6_addr.s6_addr = in6addr_arg;//绑定的ip地址
		sin6.sin6_port = htons(MYPORT);//绑定的端口号
		bind(s, (struct sockaddr *) &sin6, sizeof sin6);

-  **listen(SOCKET s, 5);     ** 
要接收客户机的连接，服务器必须在绑定其套接字之后执行两个步骤。第一步是说明可以排队多少连接请求。

-  **accept(SOCKET s, (struct sockaddr *) &SockClient_addr, &sizeof(SockClient_addr));**
第二步接受客户机连接。此函数返回一个连接到请求客户机的**新套接字**描述符。它仅用于与这个特定的客户通信，而命名套接字s则被保留下来继续处理来自其他客户的连接。
用法：
struct sockaddr_in6 SockClient_addr;
clientSock = accept(s, (struct sockaddr *) &SockClient_addr, &sizeof(SockClient_addr));

-  **ssize_t read(int fd,void *buf, sizeof nbytes)**
从fd中读取nbytes字节内容并存入buf所指的空间中.当读成功 时,read返回实际所读的字节数,如果返回的值是0, 表示已经读到文件的结束了,小于0表示出现了错误.

-  **ssize_t write(int fd, const void*buf,size_t nbytes);**
write函数将buf中的nbytes字节内容写入文件描述符fd.成功时返回写的字节数.失败时返回-1.

-  **int send( SOCKET s, const char FAR *buf, int len, int flags );  **
第一个参数指定**发送端**套接字描述符:**send a message from a socket s**
第二个参数指明一个存放应用程序要**发送数据**的缓冲区；
第三个参数指明实际要发送的数据的字节数；
第四个参数一般置0

-  **int recv( SOCKET s, char FAR *buf, int len, int flags);**
第一个参数指定数据**发送端**套接字描述符:**receive a message from a socket s**
第二个参数指明一个缓冲区，该缓冲区用来存放recv函数**接收到的数据**；
第三个参数指明buf的长度；
第四个参数一般置0。


-  **close(SOCKET s)**
关闭套接字。

- **connect(SOCKET s, (struct sockaddr *)&server, sizeof server);**
客户机启动到服务器套接字的连接，向服务器请求服务。
用法：（客户机端）
		struct sockaddr_in6 server;
		...
		connect(s, (struct sockaddr *)&server, sizeof server);

##socket缓冲区
每一个socket在被创建之后，系统都会给它分配两个缓冲区，即输入缓冲区和输出缓冲区。 
![socket缓冲区](http://img.blog.csdn.net/20170723170103391?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWFuZ2t1bnFpYW5rdW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

-  send函数并不是直接将数据传输到网络中，而是负责将数据写入输出缓冲区，数据从输出缓冲区发送到目标主机是由TCP协议完成的。**数据写入到输出缓冲区之后，send函数就可以返回了，数据是否发送出去，是否发送成功，何时到达目标主机，都不由它负责了，而是由协议负责。**

-  recv函数也是一样的，它并不是直接从网络中获取数据，而是从输入缓冲区中读取数据。

##socket数据发送与接收问题
数据的发送和接收是独立的，并不是发送方执行一次send，接收方就执行一次recv。recv函数不管发送几次，都会从输入缓冲区尽可能多的获取数据。**如果发送方发送了多次信息，接收方没来得及进行recv，则数据堆积在输入缓冲区中，**取数据的时候会都取出来。换句话说，recv并不能判断数据包的结束位置。

###send函数： 
在数据进行发送的时候，需要先检查输出缓冲区的可用空间大小，如果可用空间大小小于要发送的数据长度，则send会被阻塞，直到缓冲区中的数据被发送到目标主机，有了足够的空间之后，send函数才会将数据写入输出缓冲区。

要写入的数据大于输出缓冲区的最大长度的时候，要分多次写入，直到所有数据都被写到缓冲区之后，send函数才会返回。

###recv函数： 
函数先检查输入缓冲区，如果输入缓冲区中有数据，读取出缓冲区中的数据，否则的话，recv函数会被阻塞，等待网络上传来数据。如果读取的数据长度小于输出缓冲区中的数据长度，没法一次性将所有数据读出来，需要多次执行recv函数，才能将数据读取完毕。

#####接收BuffSize >= 发送BuffSize >= 实际发送Size

> nbytes = recv(bBuffer,iBufferLen,0); //也有可能无法收到全部数据！ 
必须要考虑0 < nbytes < iBufferLen的情况==>继续接收iBufferLen - ret字节，然后合并
例如：服务器在循环recv，recv的缓冲区大小为100byte，客户端在循环send，每次send 6byte数据，则recv每次收到的数据可能为6byte，12byte，18byte，这是随机的，编程的时候注意正确的处理!!


###资料
linux socket 编程（C语言实例):http://blog.csdn.net/piaojun_pj/article/details/5920888
OpenCV Mat数据类型及位数总结:http://blog.sina.com.cn/s/blog_662c7859010105za.html
深入理解TCP网络编程中的send和recv:http://blog.csdn.net/yusiguyuan/article/details/21439719
Opencv之<Vec3b>:http://blog.csdn.net/qq_29540745/article/details/52517269
cv::Mat 之初始化:http://blog.sina.com.cn/s/blog_79bb01d00101ao58.html