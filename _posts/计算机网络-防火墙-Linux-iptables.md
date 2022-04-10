### Linux防火墙概述
Linux系统的防火墙功能是由**内核**实现的，是一个 IP信息包过滤系统，它实际上由两个组件netfilter和iptables组成，工作在网络层。iptables是运行在用户空间的应用软件，通过控制Linux内核netfilter模块，来管理网络数据包的处理和转发。

**内核（Kernel）**：是操作系统的内部核心

Linux命令查看内核版本：

```bash
uname -r
```

**作用**

 - 对CPU进行调度管理
 - 对内存空间进行分配和管理
 - 对进程进行管理
 - 对磁盘里的文件系统进行管理
 - 对网络进行管理
 - 对其他硬件进行管理

**内核态:** 工作在内核空间里的进程的状态
**用户态:** 工作在用户空间里的进程
（运行在用户态的进程不能直接访问操作系统内核空间，内核态的进程可以访问用户空间。）
**用户态切换到内核态的方式**
 - 系统调用的时候
 - 中断的时候 ，硬件出现问题
 - IO的时候（进出读写的时候）
 
 **进程的三个基本状态**
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/aa2c4b1a3f664bd09411d6567033f4d0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

 - **就绪状态**：进程已获得除CPU外的所有必要资源，只等待CPU时的状态。一个系统会将多个处于就绪状态的进程排成一个就绪队列。
 - **执行状态**：进程已获CPU，正在执行。
 - **阻塞状态**：正在执行的进程由于某种原因而暂时无法继续执行，便放弃处理机而处于暂停状态，即进程执行受阻。（这种状态又称等待状态或封锁状态）
 
 **netfilter:** 在Linux内核中的包过滤防火墙功能体系
 **iptables:**运行在用户空间位于/sbin/iptables，用来管理防火墙的命令工具，真正实现防火墙功能的是netfilter，我们配置了iptables规则后netfilter通过这些规则来进行防火墙过滤等操作。
 
## iptables的四表五链
**规则链:** 防火墙规则策略的集合
默认的5种规则链
 - **INPUT:** 处理入站数据包
 - **OUTPUT:** 处理出站数据包
 - **FORWARD:** 处理转发数据包
 - **PREROUTING:** 路由选择之前
 - **POSTEROUTING:** 路由选择之后

**规则表:** 具有某一类相似用途的防火墙规则

默认的4个规则表
- **filter表:** 确定是否放行该数据包（过滤）
- **nat表:** 修改数据包中的源，目标IP地址或端口
- **raw表:** 确定是否对该数据包进行状态跟踪
- **mangle表:** 为数据包设置标记

查看表所在的链：iptables -t 表 -L

```bash
iptables -t filter -L 
```
规则表的优先顺序

	raw--->mangle--->nat--->filter

规则链内的匹配顺序
- 按顺序依次进行检查，找到相匹配的规则即停止（LOG策略会有例外）
- 若在该链内找不到相匹配的规则，则按该链的默认策略处理


![在这里插入图片描述](https://img-blog.csdnimg.cn/742ac72d9f64473290b15294cfb7b668.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

## iptables命令的管理选项
设置规则内容

 - **-A**：在链尾追加一条新的规则
 - **-I**：在指定位置（默认在规则链头部）插入一条新的规则
 - **-R**：修改替换指定位置或内容的规则
 - **-P**：设置指定链的默认策略
 - **-D** ：删除指定位置或内容的规则
- **-F** ：清空指定链的规则，默认清空所有链
- **-N** ：新建一条规则链
 - **-L**：列表查看各条规则信息
 - **--line-numbers**：查看规则信息时显示规则的行号
 - **-n**：以数字形式显示IP地址，端口等信息
 - **-v**：显示数据包个数，字节等详细信息
- **-t**：指定规则表，默认filter表
通用条件匹配

- **-p**：指定协议规则
- **-s**：指定IP地址
- **-i**：指定网络接口进
- **-o**：指定网络接口出
- **--sport**：源端口
- **--dport**：目标端口
(采用“端口1：端口2”的形式可以指定一个范围的端口)

-  **icmp --icmp-type**：icmp类型字段 8 是 Echo request回显请求，0是Echo reply——回显应答。

**多端口**
```bash
iptables -A INPUT -p tcp -m multiport --deport 9988,8877,6655 -j ACCCEPT 
```
(逗号表示分割，冒号表示范围)

**定时清除iptables防火墙规则脚本**

```bash
[root@centos71 ~]vim /root/clear_iptables.sh
```
```bash
#!/bin/bash
/usr/sbin/iptables -F  清楚防火墙规则
/usr/sbin/iptables -P INPUT ACCEPT 打开INPUT规则链
```

```bash
[root@centos71 ~]crontab -e
```
添加一条定时任务
```bash
*/1 * * * * bash /root/test/317/clear_iptables.sh
#每隔1分钟
```

**数据包控制**:
- ACCEPT：放行数据包
- DROP：丢弃数据包
- REJECT：拒绝数据包
- LOG：记录日志信息，并传递给下一条规则处理
- 用户自定义链名：传递给自定义链内的规则进行处理
**导入，导出防火墙规则**
导出规则：iptables-save，结合重定向“>”符号保存规则信息
导入规则：iptables-restore，结合重定向“<”符号恢复规则信息

## 开机自动加载iptables规则
**/etc/rc.d/rc.local** 这个文件里的命令开机自启
开机执行脚本

```bash
[root@centos71 317]vim /etc/rc.d/rc.local 
```


添加一行

```bash
bash /root/test/317/iptables.sh
```
给予可执行权限
```bash
chmod +x /etc/rc.d/rc.local
```

**iptables-restore恢复**

```bash
[root@centos71 317]iptables-save >/root/test/317/iptables.rule
```
```bash
[root@centos71 317]vim /etc/rc.d/rc.local 
```
#添加一行

```bash
iptables-restore < /root/test/317/iptables.rule
```

给予可执行权限
```bash
chmod +x /etc/rc.d/rc.local
```
---------------
 
