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
根据实验宿主机的性能条件，可以适度精简靶机数量。本次实验中使用了两个靶机。

    3、靶机、网关和victim的网络设置
    