# 基于scapy编写扫描器

## 实验目的

- 掌握网络扫描之端口状态探测的基本原理

## 实验环境

- python + scapy

## 实验要求

- 禁止探测互联网上的 IP ，严格遵守网络安全相关法律法规
- 完成以下扫描技术的编程实现 
  - TCP connect scan / TCP stealth scan
  - TCP Xmas scan / TCP fin scan / TCP null scan
  - UDP scan
- 上述每种扫描技术的实现测试均需要测试端口状态为：开放、关闭和过滤状态时的程序执行结果
- 提供每一次扫描测试的抓包结果并分析与课本中的扫描方法原理是否相符？如果不同，试分析原因
- 在实验报告中详细说明实验网络环境拓扑、被测试 IP 的端口状态是如何模拟的
- （可选）复刻 nmap 的上述扫描技术实现的命令行参数开关

## 实验准备

### 搭建实验拓扑结构

本机IP 

![image-20191113165837402](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191113165837402.png)

虚拟机IP

![image-20191113170039236](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191113170039236.png)

 ![image-20191113162926017](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191113162926017.png)

![image-20191113170204578](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191113170204578.png)

端口控制指令：

```
 systemctl start apache2 #开启80端口
 systemctl stop apache2 #关闭80端口
 iptables -A INPUT -p tcp --dport 80 -j DROP #不接收80端口的数据包
 iptables -A INPUT -p tcp --dport 80 -j ACCEPT #接收80端口的数据包
 netstat -anop | grep LISTEN | grep -v unix #查看开启状态的端口
```

### 实验步骤

### TCP connect scan 

客户端与服务器建立 TCP 连接要进行一次三次握手，如果进行了一次成功的三次握手，则说明端口开放。 

客户端想要连接服务器80端口时，会先发送一个带有 SYN 标识和端口号的 TCP 数据包给服务器。如果端口是开放的，则服务器会接受这个连接并返回一个带有 SYN 和 ACK 标识的数据包给客户端。随后客户端会返回带有 ACK 和 RST 标识的数据包，此时客户端与服务器建立了连接。如果完成一次三次握手，那么服务器上对应的端口肯定就是开放的。 

当客户端发送一个带有 SYN 标识和端口号的 TCP 数据包给服务器后，如果服务器端返回一个带 RST 标识的数据包，则说明端口处于关闭状态。 

实验代码

```python
`\#! /usr/bin/python

import logging
logging.getLogger("scapy.runtime").setLevel(logging.ERROR)
from scapy.all import 
dst_ip = "192.168.56.101"
src_port = RandShort()
dst_port=80
tcp_connect_scan_resp = sr1(IP(dst=dst_ip)/TCP(sport=src_port,dport=dst_port,flags="S"),timeout=10)

if(str(type(tcp_connect_scan_resp))==""):
  print("Closed")
elif(tcp_connect_scan_resp.haslayer(TCP)):
  if(tcp_connect_scan_resp.getlayer(TCP).flags == 0x12):
​    send_rst = sr(IP(dst=dst_ip)/TCP(sport=src_port,dport=dst_port,flags="AR"),timeout=10)
​    print("Open")
elif (tcp_connect_scan_resp.getlayer(TCP).flags = 0x14):
  print("Closed")
```

端口关闭和开启时的状态：

![image-20191113165346963](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191113165346963.png)

1、开启状态：运行scan.py

![image-20191113170400993](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191113170400993.png)

![image-20191113170559140](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191113170559140.png)

客户端与服务器建立 TCP 连接要进行一次三次握手，如果进行了一次成功的三次握手，则说明端口开放。 

2、关闭状态：

![image-20191114090853658](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191114090853658.png)

![image-20191113212752928](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191113212752928.png)

3、过滤状态：

![image-20191113213745650](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191113213745650.png)

![](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191113213710698.png)

### TCP Xmas scan

在Xmas扫描中，客户端会向服务器发送带有 PSH,FIN,URG 标识和端口号的数据包给服务器。如果目标端口是开放的，那么不会有任何来自服务器的回应。 

如果服务器返回了一个带有 RST 标识的 TCP 数据包，那么说明端口处于关闭状态。 

如果服务器返回了一个 ICMP 数据包，其中包含 ICMP 目标不可达错误类型3以及 ICMP 状态码为1，2，3，9，10或13，则说明目标端口被过滤了无法确定是否处于开放状态。 

实验代码：

```python
#! /usr/bin/python

import logging
logging.getLogger("scapy.runtime").setLevel(logging.ERROR)
from scapy.all import *

dst_ip = "192.168.56.101"
src_port = RandShort()
dst_port=80

xmas_scan_resp = sr1(IP(dst=dst_ip)/TCP(dport=dst_port,flags="FPU"),timeout=10)
if (str(type(xmas_scan_resp))==""):
    print "Open|Filtered"
elif(xmas_scan_resp.haslayer(TCP)):
    if(xmas_scan_resp.getlayer(TCP).flags == 0x14):
        print "Closed"
elif(xmas_scan_resp.haslayer(ICMP)):
    if(int(xmas_scan_resp.getlayer(ICMP).type)==3 and int(xmas_scan_resp.getlayer(ICMP).code) in [1,2,3,9,10,13]):
        print "Filtered"
```

1、开启状态

遇到的问题：开放了80端口但是一直无法收到回复包

解决过程

![image-20191114091008984](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191114091008984.png)

优先执行上方的指令，在ACCEPT的指令之前会先DROP，所以无法回复包。

解决方法：清除了DROP的指令

![image-20191114091139811](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191114091139811.png)

处理完成后重新抓包：还是没收到回复……

2、关闭状态

![image-20191114091722345](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191114091722345.png)

![image-20191114091835510](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191114091835510.png)

3、过滤状态

问题同开启状态

### UDP scan

TCP 是面向连接的协议，而UDP则是无连接的协议。

面向连接的协议会先在客户端和服务器之间建立通信信道，然后才会开始传输数据。如果客户端和服务器之间没有建立通信信道，则不会有任何产生任何通信数据。

无连接的协议则不会事先建立客户端和服务器之间的通信信道，只要客户端到服务器存在可用信道，就会假设目标是可达的然后向对方发送数据。

客户端会向服务器发送一个带有端口号的 UDP 数据包。如果服务器回复了 UDP 数据包，则目标端口是开放的。 

如果服务器返回了一个 ICMP 目标不可达的错误和代码3，则意味着目标端口处于关闭状态。 

如果服务器返回一个 ICMP 错误类型3且代码为1，2，3，9，10或13的数据包，则说明目标端口被服务器过滤了。 

但如果服务器没有任何相应客户端的 UDP 请求，则可以断定目标端口可能是开放或被过滤的，无法判断端口的最终状态。 

实验代码

```python
#! /usr/bin/python

import logging
logging.getLogger("scapy.runtime").setLevel(logging.ERROR)
from scapy.all import *

dst_ip = "192.168.56.101"
src_port = RandShort()
dst_port=53
dst_timeout=10

def udp_scan(dst_ip,dst_port,dst_timeout):
    udp_scan_resp = sr1(IP(dst=dst_ip)/UDP(dport=dst_port),timeout=dst_timeout)
    if (str(type(udp_scan_resp))==""):
        retrans = []
        for count in range(0,3):
            retrans.append(sr1(IP(dst=dst_ip)/UDP(dport=dst_port),timeout=dst_timeout))
        for item in retrans:
            if (str(type(item))!=""):
                udp_scan(dst_ip,dst_port,dst_timeout)
        return "Open|Filtered"
    elif (udp_scan_resp.haslayer(UDP)):
        return "Open"
    elif(udp_scan_resp.haslayer(ICMP)):
        if(int(udp_scan_resp.getlayer(ICMP).type)==3 and int(udp_scan_resp.getlayer(ICMP).code)==3):
            return "Closed"
        elif(int(udp_scan_resp.getlayer(ICMP).type)==3 and int(udp_scan_resp.getlayer(ICMP).code) in [1,2,9,10,13]):
            return "Filtered"

print udp_scan(dst_ip,dst_port,dst_timeout)
```

1、开启状态

![image-20191114095001208](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191114095001208.png)（我也不知道为什么这么多包） 

2、关闭状态

![image-20191114095358578](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191114095358578.png)

![image-20191114095521867](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191114095521867.png)

3、过滤状态

要先打开53端口

![image-20191114100151761](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191114100151761.png)

![image-20191114100120265](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191114100120265.png)

