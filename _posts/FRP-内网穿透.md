## 1.frp介绍

   frp 是一种快速反向代理，可帮助您将 NAT 或防火墙后面的本地服务器暴露给 Internet。目前，它支持TCP和UDP，以及HTTP和HTTPS协议，可以通过域名将请求转发到内部服务。

## **2.为什么使用frp**

 - 如果公司的内网不给提供外网访问，或者没有给分配外网可以访问的IP，我们又需要访问SSH登录内网的服务器，远程桌面、远程文件
- 远程桌面使用TeamViewer，但需要访问端也拥有TeamViewer软件，不方便。且TeamViewer不易实现远程文件访问。
- 使用蒲公英相关的拨号软件进行组网，可用，但免费版本网络速度极慢，体验不佳，几乎无法正常使用。
- 使用花生壳软件进行DNS解析，可用，但同第二点所述，免费版本有带宽限制，无法实际使用。

## 3.服务器配置
1.首先需要1台云服务器（Linux和Windows都可以，配置大同小异，下面先介绍Linux服务端配置)
2.[https://github.com/bttb520/SakuraFrp/releases/download/v0.28.2/frp_0.28.2_linux_amd64.tar.gz](https://github.com/bttb520/SakuraFrp/releases/download/v0.28.2/frp_0.28.2_linux_amd64.tar.gz)
下载好后上传到服务器上，我使用Xftp软件上传拖动，强烈推荐使用Xftp，功能强大的传输软件,能同时适应初级用户和高级用户的需要。它采用了标准的Windows风格的向导，它简单的界面能与其他windows应用程序紧密地协同工作，此外它还为高级用户提供了众多强劲的功能特性。
3.上传到linux服务器的root目录下

![在这里插入图片描述](https://img-blog.csdnimg.cn/358c6d2b63e24e308e153021e977cc9c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_17,color_FFFFFF,t_70,g_se,x_16)
4.登录Linux服务器（我使用Xshell和Xftp配套连接服务器）

解压缩命令

```bash
tar -zxvf frp_0.28.2_linux_amd64.tar.gz
```

解压后进入目录运行frps

```bash
cd frp_0.28.2_linux_amd64
./frps
```

我们不用管其他设置，端口默认7000（默认服务器7000端口关闭，必须保证服务器7000端口开放，进入云服务器管理平台打开安全组配置策略开放7000端口），服务器我们看到成功打开frps
![在这里插入图片描述](https://img-blog.csdnimg.cn/521f7457b15e4afc866b1867bd805540.png)
## 4.客户端配置
从[https://github.com/bttb520/SakuraFrp/releases/download/v0.28.2/frp_0.28.2_linux_amd64.tar.gz](https://github.com/bttb520/SakuraFrp/releases/download/v0.28.2/frp_0.28.2_linux_amd64.tar.gz)下载的安装包，我们解压到本地的D盘frp文件下面
打开frpc.ini进行编辑如下

```bash
[common]
server_addr = 服务器ip地址
server_port = 7000
[80]
type = tcp
local_ip = 127.0.0.1
local_port = 80
remote_port = 8080
[RDP]
type = tcp
local_ip = 127.0.0.1
local_port = 3389
remote_port = 8081
```
sever_adder = 后面改为自己购买的具有公网的服务器ip地址

 - 上面80配置可以使本机的80web端口投射到服务器的8080端口（默认服务器8080端口关闭，必须保证服务器8080端口开放，进入云服务器管理平台打开安全组配置策略开放8080端口）
 - 上面RDP配置可以使本机的3389远程控制桌面端口投射到服务器的8081端口（默认服务器8081端口关闭，必须保证服务器8081端口开放，进入云服务器管理平台打开安全组配置策略开放8081端口）
 
 cmd 打开命令指示符
![在这里插入图片描述](https://img-blog.csdnimg.cn/255542cfe71444d791f2edeebcc7cbb3.png)
看到start proxy success提示成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/ec79de79ac3a462e801ea2bcb9d2be67.png)

```bash
D:
cd \frp\frp_0.28.2_windows_amd64
frpc.exe
```

为了快速打开frpc，我们可以简单写一个脚本，新建文本，复制上面命令保存，修改txt后缀为bat，下次双击快速打开。

