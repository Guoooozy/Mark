# Java网络编程

## 1.计算机网络

### 1.1.概述

#### 1.1.1.计算机网络

计算机网络是指将`地理位置不同`的具有独立功能的`多台计算机及其外部设备`，通过通信线路连接起来，在`网络操作系统`，`网络管理软件`以及**网络通信协议**的管理和协调下，实现`资源共享`和信息传递的计算机系统

#### 1.1.2.网络编程的目的

传播交流信息，数据交换，通信

> 想要达到这个效果需要什么？

1.如何准确地定位网络上的一台主机   192.168.16.124：端口 ，定位到这个计算机上的某个资源（进程）
2.找到了这个主机，如何传输数据呢？物理设备，网线
JavaWeb 网页编程 B/S
网络编程  TCP/IP C/S

### 1.2.网络通信的要素

通信双方的地址：1.IP   2.端口号  
规则L网络通信的协议
http，ftp，smtp，tcp，tdp

> 小结

1. 网络编程中两个主要的问题
   - 如何准确地定位到网络上的一台或多台主机
   - 找到主机后如何进行通信
2. 网络编程中的要素
   - IP和端口号 IP
   - 网络通信协议  udp  tcp
3. 万物皆对象

### 1.3.IP

ip地址：inetAddress

- 唯一定位一台网络上计算机
- 127.0.0.1：本机localhost
- ip地址的分类
  - ipv4/ipv6
    - `ipv4` 127.0.0.1  ，  4个字节组成。0~255,42亿，30亿个都在北美，亚洲4亿。2011年就用尽
    - `ipv6`  2409:8a44:3052:9110:3cfc:b136:b146:4af1  128位，由8个无符号整数
  - 公网（互联网）-私网（局域网）
    - ABCD类地址
    - 192.168.xx.xx 专门给组织内部使用
- 域名：记忆ip问题

```java
public static void main(String[] args) throws Exception{
        InetAddress inetAddress1= InetAddress.getByName("127.0.0.1");
        System.out.println(inetAddress1);
        InetAddress inetAddress3=InetAddress.getByName("localhost");
        System.out.println(inetAddress3);
        InetAddress inetAddress = InetAddress.getByName("www.baidu.com");
        System.out.println(inetAddress);
        System.out.println("==============");
        System.out.println(inetAddress.getAddress());
        System.out.println(inetAddress.getCanonicalHostName());
        System.out.println(inetAddress.getHostAddress());
        System.out.println(inetAddress.getHostName());
    }
```

### 1.4.端口

端口表示计算机上的一个程序的进程

- 不同的进程有不同的端口号，用来区分软件
- 被规定只有0~65535
- TCP，UDP  ：65535*2  Tcp：80  ，单个协议下，端口号不能冲突
- 端口分类：
  - 公有端口0~1023
    - HTTP：80
    - HTTPS：443
    - FTP：21
    - Telent：23
  - 程序注册端口：1024~49151
    - Tomcat：8080
    - Mysql：3306
    - Oracle：1521
  - 动态，私有：49152~65535

```bash
netstat -ano#查看所有端口
netstat - ano|findstr ""#查看某个具体的端口
```

### 1.5.通信协议

协议：约定，好比我们现在说的是普通话
网络通信协议：速率，码率传输，代码机构，传输控制
问题：非常的复杂（分层）
TCP/IP协议：（实际上是一组协议）

- TCP：用户传输协议
- 用户：用户数据报协议
- IP：网络互联协议

TCP udp 对比
TCP：打电话

- 连接，稳定

- 三次握手，四次挥手

  ```tex
  三次握手：最少需要三次，才能保证稳定连接！（三次握手建立连接）
  A:你愁啥
  B:瞅你咋地
  A：干一场？
  四次挥手：四次挥手结束
  A：我要走了
  B：你真的要走了吗？
  B：你真的真的要走了吗？
  A：我真的真的要走了
  ```

  

- 客户端，服务端（区分分明）

- 传输完成，释放连接，效率低

DUP：发短信

- 不连接，不稳定
- 客户端，服务端（没有明确的界限）
- 不管有没有准备好，都可以发给你
- 导弹攻击
- DDOS：洪水攻击，饱和攻击

