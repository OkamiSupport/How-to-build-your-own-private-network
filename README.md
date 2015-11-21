# How-to-setup-your-private-network
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


##VPN接入：
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
>wget &nbsp;&nbsp;   ***~~Removed according to regulations.~~***  
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

##访问控制（使用Toughradius）：
* [Toughradius项目地址](https://github.com/talkincode/ToughRADIUS)
* [Toughradius安装文档](http://docs.toughradius.net/index.html)

###VPN和ToughRadius之间的对接：
***（其实就是和普通Radius对接差不多，记得在BAS管理那一栏添加服务器网卡IP，DOCKER的和eth0都要添加）***
* [PPTP](http://docs.toughradius.net/toughradius/pptp.html)
* [OpenVPN（只需要看Radius对接部分）](http://blog.csdn.net/xiaoxinghehe/article/details/8253100)
* [Cisco IPsec VPN](https://gist.github.com/OkamiSupport/4892f251e837ee708131)
* *~~IKEv2~~* (Not support)

