# How-to-build-your-own-private-network
一个完整的虚拟网络是由以下几部分组成的：
* 接入部分（OpenVPN，PPTP/L2TP，Cisco IPsec，IKEv2等VPN协议）
* 访问控制部分（常用Radius控制，toughradius是个不错的选择）
* 路由部分（Linux IP Route，Quagga等）
* 防火墙部分（Linux IPtables）

这个repo讲解关于Linux IP Route的用法和用Quagga来实现路由控制，以及一些VPN协议和Radius对接的方法。

拓扑图如下：
![](http://i.imgur.com/k9h5Egw.png)  
图中的Host是指接入用户，pop是接入点，company router是公司核心交换服务器。  
路由默认走ISP1，特殊路由通过特定的server走ISP2。
<br></br>
*Please assume that the router in this figure is a regular server.*


##接入部分
<br></br>
###用户——服务器端
* OpenVPN [Setup And Configure OpenVPN Server On CentOS 6.5](http://www.unixmen.com/setup-openvpn-server-client-centos-6-5/)
* Cisco IPsec VPN /IKEv2 VPN [使用Strongswan搭建IPSEC VPN](https://hjc.im/shi-yong-strongswanda-jian-ipsecikev2-vpn/)
* PPTP [How To Setup Your Own VPN With PPTP](https://www.digitalocean.com/community/tutorials/how-to-setup-your-own-vpn-with-pptp)
* L2TP [IPSEC L2TP VPN on CentOS 6](https://raymii.org/s/tutorials/IPSEC_L2TP_vpn_on_CentOS_-_Red_Hat_Enterprise_Linux_or_Scientific_-_Linux_6.html)

常用的VPN教程已给出，本文不再做赘述。  
<br></br>

###服务器到服务器端
[VPN Traffic Redirect](https://github.com/OkamiSupport/VPN-traffic-redirect-to-another-vpn-tunnel)

>项目地址：&nbsp;&nbsp;   ***~~Removed according to regulations.~~***  

>```
>wget     ***~~Removed according to regulations.~~***  
>tar zxvf shadowvpn-0.1.6.tar.gz  
>./configure --enable-static --sysconfdir=/etc  
>make && sudo make install  
>```
>
>记得打开ShadowVPN用的端口，不然隧道起不来。  
>
>记得清除掉ShadowVPN client_up.sh中的一段命令：  
>```
>echo changing default route  
>if [ pppoe-wan = "$old_gw_intf" ]; then  
>route add $server $old_gw_intf  
>else  
>route add $server gw $old_gw_ip  
>fi  
>route del default  
>route add default gw 10.7.0.1  
>echo default route changed to 10.7.0.1  
>```
>不然ShadowVPN up后，你所有的流量都从ShadowVPN隧道走了。  
>修改完后，执行
>```
>shadowvpn -c /etc/shadowvpn/client.conf -s start
>```
>
>测试隧道通信是否成功：  
>```
>sudo route add -host 8.8.8.8 dev tunX （tunX是你ShadowVPN的interface）  
>```
>然后
>```
>nslookup twitter.com 8.8.8.8  
>```
>如果返回的地址是无污染的IP，说明隧道已经UP了。  
<br></br>

##访问控制部分（使用Toughradius）
* [Toughradius项目地址](https://github.com/talkincode/ToughRADIUS)
* [Toughradius安装文档](http://docs.toughradius.net/index.html)

###VPN和ToughRadius之间的对接：
*（其实就是和普通Radius对接差不多，记得在BAS管理那一栏添加服务器网卡IP，DOCKER的和eth0都要添加）*
* [PPTP](http://docs.toughradius.net/toughradius/pptp.html)
* [OpenVPN（只需要看Radius对接部分）](http://blog.csdn.net/xiaoxinghehe/article/details/8253100)
* [Cisco IPsec VPN](https://gist.github.com/OkamiSupport/4892f251e837ee708131)
* *~~IKEv2~~* (Not supported)  
ToughRadius是个很强大的东西，控制账号接入极其的方便。

##路由部分  
**由于员工所在的ISP连接公司服务器非常卡，所以公司希望能在员工所在的ISP部署多线服务器来实现加速访问。**  
**这一环节讲解如何实现员工拨入就近pop点，网络流量被无条件转发到公司服务器上。**  
** 假设你的服务器之间的ShadowVPN已经建立好且流量通信正常 **  
<br></br>
首先需要在pop点上创建一张私有路由表
```
echo "200 vpnredirect" >> /etc/iproute2/rt_tables
```
规定哪些VPN网段使用这张路由表
```
ip rule add from [your vpn subnet] table vpnredirect
```
给这张路由表添加默认路由（不然不走数据）
```
ip route add default dev [your shadowvpn interface]
```
最后，刷新缓存
```
ip route flush table vpnredirect cache
```
这样，接入pop的VPN流量就被重定向到公司服务器了。

##防火墙部分
一般来说，当shadowvpn起来后，脚本会自动为你添加相应的iptables规则，所以不用关心iptables规则。  
但如果你用的是其他的VPN协议做的site to site，则需要手动添加一些iptables规则，不多，也就几条。  
```
//国内pop点
iptables -t nat -A POSTROUTING -s [remote VPN subnet] -o [site to site tunnel interface] -j MASQUERADE
//国外pop点
iptables -t nat -A POSTROUTING -s 0.0.0.0/0 -o [default route interface] -j MASQUERADE
```
写完后保存即可。

##高级路由部分（使用BGP协议动态控制流量走向）
虽然说员工到公司的网络连通性问题解决了，但是某天老总脑子抽了，突然想玩CSGO，结果发现连接东南亚CSGO服务器卡的不行，  
老板大为光火，命令技术去解决这个操蛋的问题。（技术：关我屁事，买个加速器不就行了）  
悲催的技术被逼去做这个倒霉的事情。最后发现某机房的机子链接CSGO服务器质量非常好，而且连接公司路由器质量也不错，决定采购这家机房的服务器做流量二次跳转。  
但购买后，发现不能把公司的流量全盘导入到这个机器中，而且，CSGO服务器的IP经常变动，写静态路由不太现实***~~（这里只是假设）~~***，而且steam上还有其他好多的东南亚服的游戏，写一大堆静态路由非常难以维护，于是在服务器上跑一个quagga启动动态路由协议，并与公司路由器对接起来。  

**首先，两边服务器先安装quagga**
```
yum -y install quagga && cp /etc/quagga/bgpd.conf.sample /etc/quagga/bgpd.conf
```
**然后，启动BGP进程和模拟思科路由的进程，并设置开机启动**
```
service bgpd start && service zebra start && chkconfig zebra on && chkconfig bgpd on
```

BGP是有AS的概念，你需要把两台服务器划分到不同的AS中。  
假定公司服务器的AS是100，机房服务器AS是101，  
而且机房机器和公司路由器（服务器）用shadowvpn做好了隧道链接，且流量通信正常，防火墙也做好了规则。  
*~~（反正在内网路由，和公网没半毛钱关系，用公网AS也没关系嘛）~~*  
*~~（严谨的同学可以换成私有的AS来做路由）~~*  

**在公司路由器（服务器）上进行操作**  

```
vtysh  //进入zebra的进程
conf t  //进入思科特权模式
no router bgp 7675  //关掉模板config自带的bgp as
router bgp 100  //自定义自己的bgp as
neighbor [对端VPN interface IP] remote-as 101  //和对端bgp 做对接
neighbor [对端VPN interface IP] ebgp-multihop  //开启ebgp-multihop，防止因为ebgp数据包ttl不够导致邻居无法建立
end //退回普通模式
wr //保存
```
**在机房服务器上进行操作**  
```
vtysh  //进入zebra的进程
conf t  //进入思科特权模式
no router bgp 7675  //关掉模板config自带的bgp as
router bgp 101  //自定义自己的bgp as
neighbor [对端VPN interface IP] remote-as 100  //和对端bgp 做对接
neighbor [对端VPN interface IP] ebgp-multihop  //开启ebgp-multihop，防止因为ebgp数据包ttl不够导致邻居无法建立
redistribute  static //重分发通过zebra(quagga)写入的静态路由
end //退回普通模式
wr //保存
```
**然后看一下邻居关系**  
```
show ip bgp summary
```
**正常情况下应该成功建立了邻居的**  
```
okami# show ip bgp summary  
BGP router identifier 172.16.35.2, local AS number 101
RIB entries 8, using 768 bytes of memory
Peers 1, using 4560 bytes of memory

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.35.1     4   100     162     167        0    0    0 00:24:55        0

Total number of neighbors 1

```
首先，获取机房那台机器的eth0网卡的网关地址。为什么不能写interface呢？
== 因为有的机房禁止了ProxyARP，写interface会导致出网路由得不到正确的结果，会显示"Destination Unreachable." ==  
进入机房服务器的configure  terminal模式下，输入CSGO的路由，并分发。  
[点击这里下载CSGO路由表](https://gist.github.com/OkamiSupport/5becd7e0fa94e2ee9dc6)  
<br></br>
***使用文本编辑器把后面的关键字替换成你special-server的eth0网卡的网关地址。***
<br></br>
**进入机房服务器的zebra进程中进行操作**
```
ip route [你要去的网段/子网掩码长度] [eth0网卡的网关地址]

//或者这么写

ip route [你要去的网段] [子网掩码] [eth0网卡的网关地址]

//按照这样把csgo的路由添加进去。
//添加完后，进入enable模式保存即可。

en
wr
```
这样就完成了。CSGO的流量会自动走到机房服务器，被服务器转发到csgo服务器上。
而且只是局部路由，公司的整体网络不会被影响。
这样做也方便维护，只要在机房的服务器上写命令就可以维护整个大网络的路由表。
（拓展性更强的意味~）

##做完后的结果

**邻居状态**  
![](http://i.imgur.com/3zDTdYD.png)  
可见邻居收到了468条Prefix  
<br></br>
**部分路由表**  
![](http://i.imgur.com/nW1pjlM.png)  
<br></br>
Success!
<br></br>
###附部分BGP常用命令
```
show ip bgp  //查看所有bgp前缀
show ip bgp summary  //查看bgp邻居摘要信息
show ip bgp neighbors  //查看bgp邻居详细信息
show ip bgp neighbor [邻居IP] advertised-routes  //查看本机给邻居发送了哪些路由前缀
clear ip bgp * soft  //软清BGP进程（不撤销邻居）
clear ip bgp *  //硬清BGP进程（断开邻居并重连，suppress-map需要硬清）
```

##文档授权协议
![](https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png)
本作品采用[知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。任何人不得使用本文档中内容进行商业活动.
<br></br>
This work is licensed under a [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-nc-sa/4.0/).  You may not use the material for commercial purposes.
