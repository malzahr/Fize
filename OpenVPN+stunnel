# OpenVPN+stunnel


因为众所周知的原因，小伙伴们不能愉快的玩耍，介绍下如何使用openVPN和stunnel来科学的上网。让我们愉快的观看欧美大片！我是酷炫的**小白兔** *（Malzahar）*.
<malzaharguo@gmail.com>


![](http://img4.imgtn.bdimg.com/it/u=1801838307,3746143499&fm=21&gp=0.jpg)

我准备了一台香港的云主机用来作为openVPN的服务端，然后在公司的机房搞了一台linux的服务器作为客户端网关。系统版本都是CentOS 6.5_64bit。



## Server端

### openvpn

**OpenVPN** 就不用过多介绍了，上干货，直接开整。

先来更新系统，安装vpn等相关软件  

1. `yum update`  
2. `yum install -y openvpn easy-rsa`

easy-rsa配置脚本默认是放在/usr/share/easy-rsa/这个路径，自行定义其位置。我的操作如下：

1. `mkdir -p /etc/openvpn/easy-rsa/keys`
2. `cp -rf /usr/share/easy-rsa/2.0/* /etc/openvpn/easy-rsa/`

编辑vars文件内容如下**(/etc/openvpn/easy-rsa/vars)**：

```
......
export KEY_COUNTRY="CN"
export KEY_PROVINCE="BJ"
export KEY_CITY="BJ"
export KEY_ORG="CY"
export KEY_EMAIL="mail@localhost.com"
export KEY_OU="server"
......
```

然而，以上内容你直接使用默认的，其实也TM能用。 

到这里/etc/openvpn/easy-rsa/来初始化证书的授权中心


```
cp openssl-1.0.0.cnf openssl.cnf
source ./vars
./clean-all
./build-ca									创建CA证书和秘钥
./build-key-server server					创建服务端证书和秘钥
./build-key client							创建客户端证书和秘钥
./build-dh									创建迪菲霍尔曼密钥交换参数
```

到这里基本就算完事了，然后需要一个服务端的配置文件

```
cp /usr/share/doc/openvpn-2.3.8/sample/sample-config-files/server.conf /etc/openvpn/
```

下面贴下我用的配置文件：

```
local 127.0.0.1    监听本地接口
port 8888          监听端口
proto tcp          协议
dev tap                
ca /etc/openvpn/keys/ca.crt            
cert /etc/openvpn/keys/server.crt    
dh /etc/openvpn/keys/dh1024.pem        
server 10.8.50.0 255.255.255.0    
client-to-client
duplicate-cn
keepalive 10 120
comp-lzo
persist-key
persist-tun
status openvpn-status.log        状态日志
log-append  openvpn.log          执行日志
verb 3
```

为毛要这样配置，因为要使用**stunnel**这个东西。

启动vpn服务喽  

`service openvpn start`

### stunnel

先到stunnel官网的[下载中心](http://www.stunnel.org/downloads.html)上找到最新的版本，然后下载下来，编译安装。

```
wget http://www.stunnel.org/downloads/stunnel-5.26.tar.gz
tar -zvxf stunnel-5.26.tar.gz
cd stunnel-5.26
./configure
make
make install
```

吗丹，要是出错了就装必要的依赖包，openssl-devel记得很重要。

在这个目录（*/usr/local/etc/stunnel/*）生成stunnel.pem秘钥文件

`openssl req -new -x509 -days 365 -nodes -out stunnel.pem -keyout stunnel.pem`

接着给它生成Diffie-Hellman部分：

`openssl gendh 512 >> stunnel.pem`

在/usr/local/etc/stunnel/目录下有一个stunnel.conf.simple文件，可以cp一份为stunnel.conf或是新建一个stunnel.conf，这里使用新建

配置如下：

```
cert = /usr/local/etc/stunnel/stunnel.pem
CAfile = /usr/local/etc/stunnel/stunnel.pem
socket = l:TCP_NODELAY=1
socket = r:TCP_NODELAY=1

pid = /tmp/stunnel.pid
verify = 3

setuid = stunnel
setgid = stunnel

compression = zlib
delay = no
sslVersion = TLSv1
fips=no

debug = 7
syslog = no
output = /usr/local/etc/stunnel/stunnel.log

[s-openvpn]    
accept = 13579                 监听端口
connect = 127.0.0.1:4443       OpenVPN端口
```

然后启动stunnel，使用以下命令

`# stunnel`

## Router端

### stunnel

记得同步时间
然后把服务器端的stunnel.pem证书传过来以便验证

`scp root@server-ip:/usr/local/etc/stunnel.pem root@router-ip:/usr/local/etc/stunnel.pem
`

编写stunnel的配置文件（*/usr/local/etc/stunnel.conf*）

```
pid = /tmp/stunnel.pid
cert = /usr/local/etc/stunnel/stunnel.pem
socket = l:TCP_NODELAY=1
socket = r:TCP_NODELAY=1
verify = 3
CAfile = /usr/local/etc/stunnel/stunnel.pem
client=yes
compression = zlib
ciphers = AES256-SHA
delay = no
failover = prio
sslVersion = TLSv1

output = /root/bin/logs/stunnel.log

[s-openvpn]
accept = 127.0.0.1:4443
connect = server.ip.address:13579
```

然后启动stunnel，和服务端一样的命令

### openvpn

安装openvpn，使用yum即可。
然后编写客户端配置文件如下：

```
dev tap                
port 65530           本地监听端口
proto tcp            协议
client               
tls-client           加密客户端
ns-cert-type server    
remote 127.0.0.1 8888     Server端口, 由于使用Stunnel加密透传, 所以连接本地端口
ca /etc/openvpn/ca/ca.crt            
key /etc/openvpn/ca/client1.key       
cert /etc/openvpn/ca/client1.crt    
persist-key
persist-tun
comp-lzo
status /etc/openvpn/openvpn-status.log       
log-append  /etc/openvpn/ca.log                
verb 3
```

启动，链接VPN

`openvpn --daemon --config /etc/openvpn/client.conf`

至此只需要添加一条默认路由就可以自由驰骋了，然而我们要做的是一台网关，所以不能加默认路由。因为*DNS*污染的问题，SO我们要使用 *8.8.8.8* 作为*DNS*的地址。但是有些国内网站会被解析到国外的地址，这就使得访问出国绕了一圈，实在没有必要啊。所以要使用 *DNSmasq* 来作为内网其他主机的*DHCP*服务和*DNS*服务。使得解析的时候当配置中没有的域名时使用 *8.8.8.8*，而解析国内地址时使用国内的*DNS*服务器，同时开启 *DNSmasq* 的*DHCP*服务。然后开启Router的ipv4转发功能，并配置iptables规则，使其成为真正意义上的路由网关服务器。为了方便，我只添加了经常访问的被墙屌的国外地址。如下：

```
ip route add 8.8.8.8 via 10.8.20.1
ip route add 8.8.4.4 via 10.8.20.1
ip route add 23.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 24.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 54.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 63.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 64.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 65.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 66.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 67.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 68.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 69.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 70.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 71.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 72.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 73.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 74.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 75.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 76.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 96.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 97.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 98.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 99.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 104.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 108.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 173.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 174.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 184.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 199.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 204.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 205.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 206.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 207.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 208.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 209.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 216.0.0.0/255.0.0.0 via 10.8.20.1
ip route add 103.245.222.0/255.255.255.0 via 10.8.20.1
```

有闲人们随意刷个脚本就好。
关于 *DNSmasq* 的配置问题，上网找去吧！ ![hoho](http://t10.baidu.com/it/u=2104728270,3839818969&fm=56)

