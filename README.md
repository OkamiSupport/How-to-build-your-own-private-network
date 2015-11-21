# How-to-setup-your-private-network
一个完整的虚拟网络是由以下几部分组成的：
* 接入部分（OpenVPN，PPTP/L2TP，Cisco IPsec，IKEv2等VPN协议）
* 访问控制部分（常用Radius控制，toughradius是个不错的选择）
* 路由部分（Linux IP Route，Quagga等）
* 防火墙部分（Linux IPtables）

这个repo讲解关于Linux IP Route的用法和用Quagga来实现路由控制，以及一些VPN协议和Radius对接的方法。

##VPN接入的方法：
###用户——服务器端
* OpenVPN [Setup And Configure OpenVPN Server On CentOS 6.5](http://www.unixmen.com/setup-openvpn-server-client-centos-6-5/)
* Cisco IPsec VPN /IKEv2 VPN [使用Strongswan搭建IPSEC VPN](https://hjc.im/shi-yong-strongswanda-jian-ipsecikev2-vpn/)
* PPTP [How To Setup Your Own VPN With PPTP](https://www.digitalocean.com/community/tutorials/how-to-setup-your-own-vpn-with-pptp)
* L2TP [](
