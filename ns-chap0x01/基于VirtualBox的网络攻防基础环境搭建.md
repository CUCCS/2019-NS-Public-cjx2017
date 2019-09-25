# chap0x01实验报告
## 基于 VirtualBox 的网络攻防基础环境搭建

### 一、实验要求
* 掌握 VirtualBox 虚拟机的安装与使用；
* 掌握 VirtualBox 的虚拟网络类型和按需配置；
* 掌握 VirtualBox 的虚拟硬盘多重加载；

### 二、实验过程
#### 1.虚拟硬盘多重加载
* 安装完虚拟机后，将系统更新并安装增强功能
* 将虚拟机关机，此时显示的是已关闭 
* 打开“管理——虚拟介质管理”，将虚拟机的类型选为多重加载，提示需要释放，确定即可，并应用
![pic1](pic/多重加载1.png)
* 打开虚拟机设置-存储，没有控制器也没有盘片，选择添加虚拟硬盘，使用现有的虚拟盘，选择刚才的vdi即可
* 通过多重加载创建新的操作系统，只需要使用现有虚拟盘，括号内显示类型为多重加载
![pic2](pic/多重加载2.png)

#### 2.网卡以及网络配置
* 靶机（ip：172.16.111.141）
![pic3](pic/靶机1.png)
* 网关（ip：10.0.2.15）
![pic4](pic/网关1.png)
* 攻击者（ip：10.0.2.6）  
![pic5](pic/攻击者1.png)
##### 网关NatNetwork网卡和攻击者的NatNetwork网卡处于一个网段
     ifconfig查看没启用的端口
     利用vim修改/etc/network/interfaces
     重新开启网卡
     再次查看，此时已经得到IP地址了
##### 靶机intnet网卡和网关的intnet网卡处于同一网段
![pic6](pic/靶机网关.png)
##### 靶机配置默认网关
     在setting-network中手动配置默认网关为网关eth1（内部网络）的IP
![pic7](pic/靶机配置网关.png)

#### 3.网络连通性测试
* 靶机可以直接访问攻击者主机
    * 靶机可以ping攻击者主机ip（攻击者ip：10.0.2.6）
    ![pic8](pic/靶机访问攻击者.png)
* 攻击者主机无法直接访问靶机
    * 攻击者ping靶机ip失败（靶机ip：172.16.111.141）
    ![pic9](pic/攻击者无法访问靶机.png)
* 网关可以直接访问攻击者主机和靶机
    * 网关ping靶机ip和攻击者ip成功
    ![pic10](pic/网关访问靶机和攻击者.png)
* 靶机的所有对外上下行流量必须经过网关  
     利用tcpdump -n -i enp0s9 icmp命令查看流量
    ![pic11](pic/流量经过.png)
* 所有节点均可以访问互联网  
    ping baidu.com 均成功
    ![pic12](pic/所有结点访问互联网.png)


### 三、实验总结
* 网络拓朴图
    ![pic13](pic/网络拓扑.png)
* 相关配置
    * 攻击者
        * 开启dhcp服务
        ```
        vi /etc/network/interfaces 
        "auto eth0"  
        "iface eth0 inet dhcp"
        /etc/init.d/networking restart
        ```
    * 网关
        * 开启转发服务
        ```
        echo 1 > /proc/sys/net/ipv4/ip_forward  
        iptables -t nat -A POSTROUTING -o eth0 -s 192.168.56.105/24 -j MASQUERADE
        ```
        * 保存iptables表
        ```
        iptables-save -c > iptables.rules
        ```
        * 查看网关流量
        ```
        tcpdump -n -i enp0s9 icmp
        ```

- - -
* *参考资料*

    kali Linux 没有ip解决办法
    https://blog.csdn.net/valecalida/article/details/88569791