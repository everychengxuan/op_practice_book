## Linux 安全
 
* [禁止ping](#禁止ping)
* [禁止密码登陆](#禁止密码登陆)
* [ssh 防暴力破解及提高 ssh 安全](#ssh-防暴力破解及提高-ssh-安全)
* [运维操作审计](#运维操作审计)
* [双因子认证](#双因子认证)
	* [安装及配置篇](#安装及配置篇)
		* [环境](#环境)
		* [查看系统时间](#查看系统时间)
		* [安装google_authenticator](#安装google_authenticator)
		* [为SSH服务器用Google认证器](#为ssh服务器用google认证器)
		* [生成验证密钥](#生成验证密钥)
	* [使用](#使用)
		* [环境](#环境-1)
		* [在安卓设备上运行Google认证器](#在安卓设备上运行google认证器)
		* [终端设置google 二次身份验证登陆](#终端设置google-二次身份验证登陆)
	* [原理](#原理)
		* [前世今生](#前世今生)
		* [TOTP中的特殊问题](#totp中的特殊问题)
* [iptables命令](#iptables命令)
	* [iptables是什么](#iptables是什么)
	* [iptables示例](#iptables示例)
		* [filter表INPUT链](#filter表input链)
		* [filter表OUTPUT链](#filter表output链)
		* [filter表的FORWARD链](#filter表的forward链)
	* [nat表](#nat表)
		* [nat表PREROUTING链](#nat表prerouting链)
		* [nat表POSTROUTING链](#nat表postrouting链)
		* [nat表做HA的实例](#nat表做ha的实例)
		* [nat表为虚拟机做内外网联通](#nat表为虚拟机做内外网联通)
	* [iptables管理命令](#iptables管理命令)
		* [查看iptables规则](#查看iptables规则)
		* [清除iptables规则](#清除iptables规则)
		* [保存iptables规则](#保存iptables规则)
	* [其他内容](#其他内容)

# 禁止ping

禁止系统响应任何从外部/内部来的ping请求攻击者一般首先通过ping命令检测此主机或者IP是否处于活动状态
，如果能够ping通某个主机或者IP，那么攻击者就认为此系统处于活动状态，继而进行攻击或破坏。如果没有人能ping通机器并收到响应，那么就可以大大增强服务器的安全
性，linux下可以执行如下设置，禁止ping请求：
```
[root@localhost ~]#echo "1"> /proc/sys/net/ipv4/icmp_echo_ignore_all
```
默认情况下"icmp_echo_ignore_all"的值为"0"，表示响应ping操作。

可以加上面的一行命令到/etc/rc.d/rc.local文件中，以使每次系统重启后自动运行

# 禁止密码登陆

# ssh 防暴力破解及提高 ssh 安全

# 运维操作审计

# 双因子认证

海上升明月，天涯共此时！

## 安装及配置篇

### 环境

server：Centos 6.5

### 查看系统时间

使用外网机器时，创建的时候有可能不是北京时间

```
[root@centos ~]#date
Sun Aug 14 23:18:41 EDT 2011
[root@centos ~]# rm -rf /etc/localtime 
[root@centos ~]# ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
### 安装google_authenticator

安装EPEL源并安装 google_authenticator

```
#curl -o epel-release-6-8.noarch.rpm "https://raw.githubusercontent.com/BillWang139967/BillWang139967.github.io/master/doc/safety/epel-release-6-8.noarch.rpm"
#rpm -ivh epel-release-6-8.noarch.rpm
#yum install google-authenticator
```
### 为SSH服务器用Google认证器

***配置/etc/pam.d/sshd***

在"auth       include      password-auth"行前添加如下内容
```
auth       required pam_google_authenticator.so
```
即先google方式认证再linux密码认证
***修改SSH服务配置/etc/ssh/sshd_config***

ChallengeResponseAuthentication no->yes

```
sed -i 's#^ChallengeResponseAuthentication no#ChallengeResponseAuthentication yes#' /etc/ssh/sshd_config
```
***重启SSH服务***

```
#service sshd restart
```
### 生成验证密钥
在Linux主机上登陆需要认证的用户运行Google认证器(我这是使用root用户演示的)
```
#google-authenticator
```
直接一路输入yes即可，询问内容如下，想了解的可以看下

```
Do you want me to update your "/root/.google_authenticator" file (y/n):y  
应急码的保存路径

Do you want to disallow multiple uses of the same authentication token? This restricts you to one login about every 30s, but it increases your chances to notice or even prevent man-in-the-middle attacks (y/n)
是否禁止一个口令多用，自然也是答 y

By default, tokens are good for 30 seconds and in order to compensate for possible time-skew between the client and the server, we allow an extra token before and after the current time. If you experience problems with poor time synchronization, you can increase the window from its default size of 1:30min to about 4min. Do you want to do so (y/n)
问是否打开时间容错以防止客户端与服务器时间相差太大导致认证失败。这个可以根据实际情况来。如果一些 Android平板电脑不怎么连网的，可以答 y 以防止时间错误导致认证失败。

If the computer that you are logging into isn't hardened against brute-force login attempts, you can enable rate-limiting for the authentication module. By default, this limits attackers to no more than 3 login attempts every 30s.Do you want to enable rate-limiting (y/n)
选择是否打开尝试次数限制（防止暴力攻击），自然答 y
```
这里需要记住的是

```
#cat /root/.google_authenticator       手机密钥和应急码保存路径
密钥
Your emergency scratch codes are: 一些生成的5个应急码，每个应急码只能使用一次
```
## 使用

### 环境

> * 密钥(一次性输入即可)
> * 安卓手机
> * linux or windows

### 在安卓设备上运行Google认证器

***安装google 身份验证器***

我的方法是在UC中搜索的 google身份验证器进行的安装

***输入密钥***

选择"Enter provided key"选项，使用键盘输入账户名称和验证密钥

### 终端设置google 二次身份验证登陆

***windows xshell***

打开xshell(其他终端类似)，选择登陆主机的属性。设置登陆方法为Keyboard Interactive

登陆时输入用户名后，接着输入手机设备上的数字，然后输入密码

***linux***

linux下直接输入

```
#ssh 用户名@IP
```
连接比较慢时可以修改本机的客户端配置文件ssh_config，注意，不是sshd_config

GSSAPIAuthentication yes -->no

```
#sed -i 's#GSSAPIAuthentication yes#GSSAPIAuthentication no#' /etc/ssh/ssh_config

```

## 原理

 基于时间的一次性密码（Time-based One-time Password，简称TOTP），只需要在手机上安装密码生成应用程序，就可以生成一个随着时间变化的一次性密码，用于帐户验证，而且这个应用程序不需要连接网络即可工作。仔细看了看这个方案的实现原理，发现挺有意思的。

### 前世今生

***HOTP***
 
 Google的两步验证算法源自另一种名为HMAC-Based One-Time Password的算法，简称HOTP。HOTP的工作原理如下：

 客户端和服务器事先协商好一个密钥K，用于一次性密码的生成过程，此密钥不被任何第三方所知道。此外，客户端和服务器各有一个计数器C，并且事先将计数值同步。
进行验证时，客户端对密钥和计数器的组合(K,C)使用HMAC（Hash-based Message Authentication Code）算法计算一次性密码，公式如下：

> HOTP(K,C) = Truncate(HMAC-SHA-1(K,C))

上面采用了HMAC-SHA-1，当然也可以使用HMAC-MD5等。HMAC算法得出的值位数比较多，不方便用户输入，因此需要截断（Truncate）成为一组不太长十进制数（例如6位）。计算完成之后客户端计数器C计数值加1。用户将这一组十进制数输入并且提交之后，服务器端同样的计算，并且与用户提交的数值比较，如果相同，则验证通过，服务器端将计数值C增加1。如果不相同，则验证失败。

这里的一个比较有趣的问题是，如果验证失败或者客户端不小心多进行了一次生成密码操作，那么服务器和客户端之间的计数器C将不再同步，因此需要有一个重新同步（Resynchronization）的机制。

***TOTP***

介绍完了HOTP，Time-based One-time Password（TOTP）也就容易理解了。TOTP将HOTP中的计数器C用当前时间T来替代，于是就得到了随着时间变化的一次性密码。非常有趣吧！

一句话概括就是

> 海上升明月，天涯共此时！

### TOTP中的特殊问题

***时间T的选取(30秒作为时间片)***

首先，时间T的值怎么选取？因为时间每时每刻都在变化，如果选择一个变化太快的T（例如从某一时间点开始的秒数），那么用户来不及输入密码。如果选择一个变化太慢的T（例如从某一时间点开始的小时数），那么第三方攻击者就有充足的时间去尝试所有可能的一次性密码（试想6位数字的一次性密码仅仅有10^6种组合），降低了密码的安全性。除此之外，变化太慢的T还会导致另一个问题。如果用户需要在短时间内两次登录账户，由于密码是一次性的不可重用，用户必须等到下一个一次性密码被生成时才能登录，这意味着最多需要等待59分59秒！这显然不可接受。综合以上考虑，

Google选择了30秒作为时间片，T的数值为从Unix epoch（1970年1月1日 00:00:00）来经历的30秒的个数。

***网络延时处理***

第二个问题是，由于网络延时，用户输入延迟等因素，可能当服务器端接收到一次性密码时，T的数值已经改变，这样就会导致服务器计算的一次性密码值与用户输入的不同，验证失败。解决这个问题个一个方法是，服务器计算当前时间片以及前面的n个时间片内的一次性密码值，只要其中有一个与用户输入的密码相同，则验证通过。当然，n不能太大，否则会降低安全性。
事实上，这个方法还有一个另外的功能。我们知道如果客户端和服务器的时钟有偏差，会造成与上面类似的问题，也就是客户端生成的密码和服务端生成的密码不一致。但是，如果服务器通过计算前n个时间片的密码并且成功验证之后，服务器就知道了客户端的时钟偏差。因此，下一次验证时，服务器就可以直接将偏差考虑在内进行计算，而不需要进行n次计算。

以上就是Google两步验证的工作原理，推荐大家使用，这确实是保护帐户安全的利器。

# iptables命令

## iptables是什么
iptables是与 Linux 内核集成的 IP 信息包过滤系统，该系统有利于在 Linux 系统上更好地控制 IP 信息包过滤和防火墙配置。

## iptables示例
### filter表INPUT链
怎么处理发往本机的包。

```
# iptables {-A|-D|-I} INPUT rule-specification
# iptables -A INPUT -s 10.1.2.11 -p tcp --dport 80 -j DROP
# iptables -A INPUT -s 10.1.2.11 -p tcp --dport 80 -j REJECT --reject-with tcp-reset
# iptables -A INPUT -s 10.1.2.11 -p tcp --dport 80 -j ACCEPT
```

以上表示将从源地址10.1.2.11访问本机80端口的包丢弃（以tcp-reset方式拒绝和接受）。

-s 表示源地址（--src,--source），其后面可以是一个IP地址（10.1.2.11）、一个IP网段（10.0.0.0/8）、几个IP或网段（192.168.1.11/32,10.0.0.0/8，添加完成之后其实是两条规则）。  
-d 表示目的地址（--dst,--destination），其后面可以是一个IP地址（10.1.2.11）、一个IP网段（10.0.0.0/8）、几个IP或网段（10.1.2.11/32,10.1.3.0/24，添加完成之后其实是两条规则）。  
-p 表示协议类型（--protocol），后面可以是tcp, udp, udplite, icmp, esp, ah, sctp, all，其中all表示所有的协议。  
--sport 表示源端口（--source-port），后面可以是一个端口（80）、一系列端口（80:90，从80到90之间的所有端口），一般在OUTPUT链使用。  
--dport 表示目的端口（--destination-port），后面可以是一个端口（80）、一系列端口（80:90，从80到90之间的所有端口）。  
-j 表示iptables规则的目标（--jump），即一个符合目标的数据包来了之后怎么去处理它。常用的有ACCEPT, DROP, REJECT, REDIRECT, LOG, DNAT, SNAT。


```
# iptables -A INPUT -p tcp --dport 80 -j DROP
# iptables -A INPUT -p tcp --dport 80:90 -j DROP
# iptables -A INPUT -m multiport -p tcp --dports 80,8080 -j DROP
```

以上表示将所有访问本机80端口（80和90之间的所有端口，80和8080端口）的包丢弃。

-m 匹配更多规则（--match），可以指定更多的iptables匹配扩展。可以是tcp, udp, multiport, cpu, time, ttl等，即你可以指定一个或多个端口，或者本机的一个CPU核心，或者某个时间段内的包。

### filter表OUTPUT链
怎么处理本机向外发的包。

```
# iptables -A OUTPUT -p tcp --sport 80 -j DROP
```

以上这条规则意思是不允许访问本机80端口的包出去。即你可以向本机80端口发送请求包，但是本机回应给你的包会被该条规则丢弃。

INPUT链与OUTPUT链用法一样，但是表示的意思不同。

### filter表的FORWARD链
For packets being routed through the box（不知道怎么解释）。

其用法与INPUT链和OUTPUT链类似。

## nat表
nat表有三条链，分别是PREROUTING, OUTPUT, POSTROUTING。

### nat表PREROUTING链
修改发往本机的包。

```
# iptables -t nat -A PREROUTING -p tcp -d 202.102.152.23 --dport 80 -j DNAT --to-destination 10.67.15.23:8080
# iptables -t nat -A PREROUTING -p tcp -d 202.102.152.23 -j DNAT --to-destination 10.67.15.23
```

以上这两条规则的意思是将发往IP地址202.102.152.23和端口80的包的目的地址修改为10.67.15.23，目的端口修改为8080。将发往202.102.152.23的其他非80端口的包目的地址修改为10.67.15.23。第二条规则中的-p tcp是可选的，也可以指定其他协议。

其实类似这样的规则一般在路由器上做，路由器上有个公网IP（202.102.152.23），其中有个用户的内网IP（10.67.15.23）想提供外网的web服务，而路由器又不想将公网IP地址绑定到用户机器上，因此就出来了以上的蛋疼规则。

### nat表POSTROUTING链
修改本机向外发的包。

```
# iptables -t nat -A POSTROUTING -p tcp -s 10.67.15.23 --sport 8080 -j SNAT --to-source 202.102.152.23:80
# iptables -t nat -A POSTROUTING -p tcp -s 10.67.15.23 -j SNAT --to-source 202.102.152.23
```

以上两条规则的意思是将从IP地址10.67.15.23和端口8080发出的包的源地址修改为202.102.152.23，源端口修改为80。将从10.67.15.23发出的非80端口的包的源地址修改为202.102.152.23。

这两条正好与以上两条PREROUTING共同完成了内网用户想提供外网服务的功能。

其中的--to-destination和--to-source都可以缩写成--to，在DNAT和SNAT中会分别被理解成--to-destination和--to-source。

注： 之所以将内网服务的端口和外网服务的端口写的不一致是因为二者其实真的可以不一致。另外，是否将PREROUTNG中的-d改为域名就可以使用一个公网IP为不同用户提供服务了呢？这个需要哥哥我稍后验证。

### nat表做HA的实例
有两台服务器和三个IP地址，分别是10.1.2.21, 10.1.2.22, 10.1.5.11。假设他们提供的是相同的WEB服务，现在想让他们做HA，而10.1.5.11是他们的VIP。

* 10.1.2.21这台的NAT规则如下：

```
# iptables -t nat -A PREROUTING -p tcp -d 10.1.2.11 --dport 80 -j DNAT --to-destination 10.1.2.21:80
# iptables -t nat -A POSTROUTING -p tcp -s 10.1.2.21 --sport 80 -j SNAT --to-source 10.1.2.11:80
```

* 10.1.2.22这台的NAT规则如下：

```
# iptables -t nat -A PREROUTING -p tcp -d 10.1.2.11 --dport 80 -j DNAT --to-destination 10.1.2.22:80
# iptables -t nat -A POSTROUTING -p tcp -s 10.1.2.22 --sport 80 -j SNAT --to-source 10.1.2.11:80
```

默认可以认为VIP在10.1.2.21上挂着，那么当这台机器发生故障不能提供服务时，我们可以及时将VIP挂到10.1.2.22上，这样就可以保证服务不中断了。当然我们可以写一个简单的SHELL脚本来完成VIP的检测及挂载，方法非常简单。

注： LVS的实现中貌似有这么一项，还没有深入去研究LVS。

### nat表为虚拟机做内外网联通
宿主机内网IP是10.67.15.183(eth1)，外网IP是202.102.152.183(eth0)，内网网关是10.67.15.1，其上面的虚拟机IP是10.67.15.250(eth1)。

目前虚拟机只能连接内网，其路由信息如下：

```
# ip r s
10.67.15.0/24 dev eth1  proto kernel  scope link  src 10.67.15.250
169.254.0.0/16 dev eth1  scope link  metric 1003 
192.168.0.0/16 via 10.67.15.1 dev eth1 
172.16.0.0/12 via 10.67.15.1 dev eth1 
10.0.0.0/8 via 10.67.15.1 dev eth1 
default via 10.67.15.1 dev eth1
```

若要以NAT方式实现该虚拟机即能连接公网又能连接内网，则该虚拟机路由需要改成以下：

```
# ip r s
10.67.15.0/24 dev eth1  proto kernel  scope link  src 10.67.15.250
169.254.0.0/16 dev eth1  scope link  metric 1003 
192.168.0.0/16 via 10.67.15.1 dev eth1 
172.16.0.0/12 via 10.67.15.1 dev eth1 
10.0.0.0/8 via 10.67.15.1 dev eth1 
default via 10.67.15.183 dev eth1
```
虚拟机连接内网的网关地址也可以写成宿主机内网IP地址。

宿主机上面添加如下NAT规则：

```
# iptables -t nat -A POSTROUTING -s 10.67.15.250/32 -d 10.0.0.0/8 -j SNAT --to-source 10.67.15.250
# iptables -t nat -A POSTROUTING -s 10.67.15.250/32 -d 172.16.0.0/12 -j SNAT --to-source 10.67.15.250
# iptables -t nat -A POSTROUTING -s 10.67.15.250/32 -d 192.168.0.0/16 -j SNAT --to-source 10.67.15.250
# iptables -t nat -A POSTROUTING -s 10.67.15.250/32 -j SNAT --to-source 202.102.152.183
```

以上四条规则的意思是将从源地址10.67.15.250发往内网机器上的数据包的源地址改为10.67.15.250。将从源地址10.67.15.250发往公网机器上的数据包的源地址修改为202.102.152.183。

## iptables管理命令
### 查看iptables规则

```
# iptables -nL
# iptables -n -L
# iptables --numeric --list
# iptables -S
# iptables --list-rules
# iptables -t nat -nL
# iptables-save
```
-n代表--numeric，意思是IP和端口都以数字形式打印出来。否则会将127.0.0.1:80输出成localhost:http。端口与服务的对应关系可以在/etc/services中查看。  
-L代表--list，列出iptables规则，默认列出filter链中的规则，可以用-t来指定列出哪个表中的规则。  
-t代表--tables，指定一个表。  
-S代表--list-rules，以原命令格式列出规则。  
iptables-save命令是以原命令格式列出所有规则，可以-t指定某个表。

### 清除iptables规则

```
# iptables -F
# iptables --flush
# iptables -F OUTPUT
# iptables -t nat -F
# iptables -t nat -F PREROUTING
```

-F代表--flush，清除规则，其后面可以跟着链名，默认是将指定表里所有的链规则都清除。

### 保存iptables规则

```
# /etc/init.d/iptables save
```

该命令会将iptables规则保存到/etc/sysconfig/iptables文件里面，如果iptable有开机启动的话，开机时会自动将这些规则添加到机器上。

## 其他内容
iptables命令中的很多选项前面都可以加"!"，意思是“非”。如"! -s 10.0.0.0/8"表示除这个网段以外的源地址，"! --dport 80"表示除80以外的其他端口。

