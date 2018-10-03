Cloudflare API v4 Dynamic DNS Update in Bash, without unnecessary requests



# 以下教程来自mrkevin博客的文章

> 文章地址：https://www.mrkevin.net/code/1444.html

最近Nathosts商家突然宣布开始取消免费[DDNS服务](https://www.mrkevin.net/tag/ddns%e6%9c%8d%e5%8a%a1)，后台仅提供当前服务器IP查看，对于用户来说，不可能每时每刻去检测母鸡的IP是否改变。

对于商家来说，最近的HKT/HKBN受攻击事件，让商家运维成本大大增加，所以取消DDNS服务，可在一定程度上保证母鸡免被攻击。

而在NAT VPS上[自建DDNS](https://www.mrkevin.net/tag/%e8%87%aa%e5%bb%baddns)，能实时更新域名指向的母鸡IP，减少用户查看IP的次数。

本次教程以Nathosts家的HKBN NAT VPS为测试机型，来进行搭建，VPS系统为Debian9 X64

### 前提条件

要自建DDNS服务，首先必须要有自己的域名，另外就是使用阿里云解析、DNSPOD云解析、[Cloudflare](https://www.mrkevin.net/tag/cloudflare)云解析等服务，教程以Cloudflare云解析为例子。

### 获取CFKEY

打开网页：<https://dash.cloudflare.com/profile>

在页面下方找到【Global API Key】，点击右侧的View查看Key，并保存下来

### 设置用来DDNS解析的二级域名

以Nathosts为例，先在控制面板找到目前母鸡的IP，然后到Cloudflare中新建一个A记录，如：ddns.yourdomain.com，指向母鸡当前的IP

### 下载脚本

※脚本来自Github

















| 1    | wget https://gist.github.com/larrybolt/6295160/raw/c634c48c001a411240fc78147949a6a32e1de370/cf-ddns.sh |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

2018/07/01更新脚本

这几天CF貌似停用了之前脚本的API，下面这个脚本修改了API的相关接口，设置依然可按照本教程的来设置

















| 1    | wget https://raw.githubusercontent.com/yulewang/cloudflare-api-v4-ddns/master/cf-v4-ddns.sh |
| ---- | ------------------------------------------------------------ |
|      |                                                              |



### 获取WANIPSITE网址

这里需要填写一个能返回当前母鸡IP的网页地址，下面提供一些

















| 123456789 | $ curl ifconfig.me$ curl icanhazip.com$ curl ident.me$ curl ipecho.net/plain$ curl whatismyip.akamai.com$ curl tnx.nl/ip$ curl myip.dnsomatic.com$ curl ip.appspot.com$ curl -s checkip.dyndns.org \| sed 's/.*IP Address: \([0-9\.]*\).*/\1/g' |
| --------- | ------------------------------------------------------------ |
|           |                                                              |

自己在SSH里看看哪个网址能返回母鸡IP的，而且是仅返回IP，不能有其他内容，我自己测试了myip.dnsomatic.com这个域名能使用。

### 填写相关信息



















| 123456789101112131415161718192021222324 | # API key, see https://www.cloudflare.com/a/account/my-account,# 上一步获取的CFKEYCFKEY= #输入你需要解析用来DDNS解析的根域名 eg: example.comCFZONE= # 暂时空着CFID= # 登陆CF的Username, eg: user@example.comCFUSER= # 填写用来DDNS解析的二级域名，与上面设置的要一致, eg: ddns.yourdomain.comCFHOST= # Cloudflare TTL for record, between 120 and 86400 secondsCFTTL=3600# Get domain ID from Cloudflare using awk/sed and python json.toolGETID=true# Ignore local file, update ip anywayFORCE=false# 填写上一步能正确返回母鸡IP的网址, other examples are: bot.whatismyipaddress.com, https://api.ipify.org/ ...WANIPSITE="myip.dnsomatic.com" |
| --------------------------------------- | ------------------------------------------------------------ |
|                                         |                                                              |



### 运行脚本

















| 12   | chmod +x cf-ddns.sh./cf-ddns.sh |
| ---- | ------------------------------- |
|      |                                 |



### 记录CFID

第一次运行后，会显示你用于DDNS解析的二级域名的CFID，记录下来

[![『教程』在NAT VPS中自建DDNS[更新CF脚本\]-Mr.KevinH](https://i.imgur.com/hl4zBw6.png)](https://i.imgur.com/hl4zBw6.png)

将CFID填入到配置文件中的CFID处

### 再次运行脚本

















| 1    | ./cf-ddns.sh |
| ---- | ------------ |
|      |              |

不出意外的话，会显示当前母鸡的IP

### 设置定时任务

设置定时任务后，就完全自动化更改域名指向的IP















| 123  | crontab -e#不知道这样写对不对0 * * * * /root/cf-ddns.sh >/dev/null 2>&1 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

教程结束-0-有什么问题欢迎留言交流或加入我们的[TG群组](https://t.me/mrkevinh)