## 产品功能

VPN网关是一款基于Internet，通过加密通道将企业数据中心、企业办公网络、或Internet终端和各公有云提供商专有网络（VPC）安全可靠连接起来的服务。其中IPSec提供站点（企业数据中心、办公网络）到站点（VPC）的安全连接。


GFW - 一道隐形的墙

这个墙并非是完全隔离的，而是选择性的，如果是不希望你上的网站就直接阻断。
每一个网络请求都是有数据特征的，不同的协议具备不同的特征，比如HTTP/HTTPS这类请求，会明确的告诉GFW它们要请求哪个域名；再比如TCP请求，它只会告诉GFW它要请求哪个IP。

GFW封锁包含多种方式，最容易操作的也是最基础的方式便是域名黑白名单，在黑名单内的域名不让通过，IP黑白名单也是这个原理。
SSH Tunnel是比较具有代表性的防窃听通讯隧道，通过SSH与境外服务器建立一条加密通道，此时的通讯GFW会将其视作普通的连接。
但是由于大家都这么玩，GFW就着急了，于是它通过各种流量特征分析，渐渐的能够识别哪些连接是SSH隧道，同尝试性地对隧道做干扰，结果就是GFW技高一筹，众多隧道纷纷不通。

而##Shadowsocks##的原理是这样的：它所发出的TCP包，没有明显包特征，GFW分析不出来，当作普通流量放过了。

## 1. 基本原理

具体的说，Shadowsocks将原来SSH创建的Socks5协议拆开成Server端和Client端，两个端分别安装在境外服务器和境内设备上。

+------+     +------+     +=====+     +------+     +-------+
| 设备  | <-> |Client| <-> | GFW | <-> |Server| <-> | 服务器 |
+------+     +------+     +=====+     +------+     +-------+

Client和Server之间可以通过多种方式加密，并要求提供密码确保链路的安全性。

## 2. 服务器端部署

Shadowsocksfeng封装后对用户而言就是一个程序指令，以Ubuntu为例，首先安装pip，

apt-get install python-pip
pip install shadowsocks

（pip现在要求python的版本大于等于2.6）。启动shadowsocks有两种方式，一种是通过一行命令直接启动：

ssserver -p PORT - k PASSWORD -m rc4-md5 --log-file /tmp/ss.log -d start

另一种是使用config文件启动，如先配置好文件(/etc/shadowsocks.jason)

{
  "server": "YOUR_SERVER_IP",
  "server_port": 8388,  
  "local_address": "127.0.0.1",  
  "local_port": 1080,  
  "password": "PASSWORD",
  "timeout": 300,  
  "method":"aes-256-cfb",  
}
然后通过ssserver启动：

ssserver -c /etc/shadowsocks.jason -d start 

