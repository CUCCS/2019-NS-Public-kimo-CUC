# 基于 VirtualBox 的网络攻防基础环境搭建
## 实验目的

    掌握 VirtualBox 虚拟机的安装与使用；
    掌握 VirtualBox 的虚拟网络类型和按需配置；
    掌握 VirtualBox 的虚拟硬盘多重加载；

## 实验环境

以下是本次实验需要使用的网络节点说明和主要软件举例：

    VirtualBox 虚拟机
    攻击者主机（Attacker）：Kali Rolling 2109.2
    网关（Gateway, GW）：Debian Buster
    靶机（Victim）：From Sqli to shell / xp-sp3 / Kali

## 实验要求

    1、虚拟硬盘配置成多重加载
![](\image\3.JPG)
* 使用已有的虚拟硬盘文件可以实现多重加载虚拟硬盘的复用。
* 遇到的一个问题：在安装Debian时不能直接挂载，要重新添加磁盘才可。

## 
    2、搭建满足如下拓扑图所示的虚拟机网络拓扑；
![](\image\4.JPG)
* 根据实验宿主机的性能条件，可以适度精简靶机数量。本次实验中使用了两个靶机、一个网关、一个Attacker 。
* 图示中有内部网络intnet1、intnet2和Nat网络，内部网络intnet1和intnet2分别有两个victim主机，Nat网络中有Gateway和Attacker。内部网络和Nat网络通过网关进行通信。

3、靶机、网关和Attacker的网络设置

* 靶机1：PCnet-FAST III 内部网络 名称为intnet1
![](\image\5.JPG)
* 靶机2：PCnet-FAST III 内部网络 名称为intnet2
![](\image\6.JPG)
！这里在设置时出现的错误：设置内部网络时，默认的是Intel PRO/1000 MT桌面，而不是PCnet-FAST III，我忘记了更改这里，导致ipconfig出现Windows IP Configuration。
* 网关：共四张网卡

网卡1：Intel PRO/1000 MT 桌面 NAT网络 名称为NatNetwork

网卡2：Intel PRO/1000 MT 桌面 仅主机（Host-Only）网络 名称为VirtualBox Host-Only Ethernet Adapter

网卡3：Intel PRO/1000 MT 桌面 内部网络 名称为intnet1

网卡4：Intel PRO/1000 MT 桌面 内部网络 名称为intnet2
![](\image\7.JPG)
！奇怪，这里的网卡3、4也使用了Intel PRO/1000 MT，可是没有出现问题。。
* Attacker:Intel PRO/1000 MT 桌面 NAT网络 名称为NatNetwork
![](\image\8.JPG)

4、查看四台电脑的默认网关
![](\image\9.JPG)

5、完成网络连通性测试
* 靶机可以访问Attacker

Attacker的IP地址为10.0.4.15，两个靶机都可以ping通
![](\image\10.JPG)
* Attacker不可以访问靶机

①靶机1、2的IP地址如图，Attacker都没有ping通，100%丢包
![](\image\11.JPG)
由此可以说明，攻击者主机和两个靶机在网络层具有不连通性。

②通过arp表，可以判断攻击者主机和两个靶机不在同一个局域网中。
但是我的两个主机都看不到arp
![](\image\12.JPG)

* 网关可以直接访问Attacker和靶机

两个靶机都能ping通，可是靶机2ping不完了……
![](\image\13.JPG)
ctrl+z结束，发现是我忘了加-c 3
![](\image\14.JPG)
ping attacker

* 靶机的所有对外上下行流量必须经过网关

通过靶机ping网关，同时网关开始抓包，过滤只剩下icmp的包，发现icmp包中的request包中源地址是靶机的IP地址，目的地址是网关的IP地址，reply包的源地址是网关的IP地址，目的地址是靶机的IP地址。

*  所有节点均可以访问互联网

四台电脑均能ping通百度
![](\image\15.JPG)