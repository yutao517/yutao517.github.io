---
layout: article
title: Zabbix-自定义脚本监控nginx+微信告警
tags: Zabbix
category: blog
date: 2022-04-16 15:00:00 +08:00
mermaid: true
---

## 项目描述
本项目的目的是构建一个能实现微信告警的zabbix监控系统，方便的监控整个NGINX集群，同时能批量的去部署和管理整个集群。
## 项目步骤
- zabbix服务端（192.168.2.138）安装好zabbix server，nginx端安装好zabbix agent（192.168.2.58），配置好Nginx负载均衡集群，并打开状态统计。
- nginx端编写监控脚本，取到nginx的状态。
- 服务端web添加监控项，出图。
- 注册企业微信，配置好微信接口。
- zabbix服务端添加接口推送脚本，测试接口正常。
- 在web端配置报警媒介，群组和用户，添加相关触发器和动作。
- nginx关停服务，测试是否能通过微信告警。

## 项目心得
在测试接收消息过程中，返回值一直是参数{ALTER.MESSAGE}，排查脚本问题，版本问题，最后发现是参数{ALERT.MESSAGE}，所以打字一定要细心，认真。同时对监控也有了一定的认识，运维人员不可能7*24小时盯着zabbix看，所以做到及时告警是非常必要的，对之前的高可用web集群项目叶可以进行完善。

## 详细步骤
### zabbix-agent客户端nginx配置监控
**nginx打开状态统计功能**

nginx配置增加stub_status模块
```bash
location = /wyt_status{
stub_status;}
```
测试状态统计功能是否打开

> http://192.168.2.58/wyt_status\

**nginx端(zabbix客户端)编写监控脚本**

```bash
cd /etc/zabbix/zabbix_agentd.d 
#在zabbix_agentd.d目录下编写监控脚本
vim zabbix-nginx_status.sh
```
```bash
#!/bin/bash
case $1 in
        active)
                curl http://192.168.2.58:80/wyt_status 2>/dev/null|awk '/Active/ {print $NF}'
                ;;
        accepts)
                curl http://192.168.2.58:80/wyt_status 2>/dev/null|awk 'NR==3 {print $1}'
                ;;
        handled)
                curl http://192.168.2.58:80/wyt_status 2>/dev/null |awk 'NR==3 {print $2}'
                ;;
        requests)
                curl http://192.168.2.58:80/wyt_status 2>/dev/null |awk 'NR==3 {print $3}'
                ;;
        reading)
                curl http://192.168.2.58:80/wyt_status 2>/dev/null |awk 'NR==4 {print $2}'
                ;;
        writing)
                curl http://192.168.2.58:80/wyt_status 2>/dev/null |awk 'NR==4 {print $4}'
                ;;
        waiting)
                curl http://192.168.2.58:80/wyt_status 2>/dev/null |awk 'NR==4 {print $NF}'
                ;;
           ping)
                pidof nginx |wc -l
                #通过查询进程PID值，测试nginx存活状态
                ;;

esac  
```

```bash
vim userparameter_nginx.conf
#在zabbix_agentd.d目录下自定义参数配置文件
UserParameter=nginx.status[*],/etc/zabbix/zabbix_agentd.d/zabbix-nginx_status.sh $1
#指定动作
chmod +x zabbix-nginx_status.sh
#授予可执行权限
service zabbix-agent restart
#刷新服务
zabbix_get -k nginx.status[ping] -s 192.168.2.58
#去服务端测试是否返回参数1
```

### zabbix-server-web配置监控
先创建nginx主机master-nginx

![在这里插入图片描述](https://img-blog.csdnimg.cn/aa6daf8e5db3416781d1d2a100823e7d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

添加应用集nginx

![在这里插入图片描述](https://img-blog.csdnimg.cn/5ddee8f005ee416fbae3ac3939265ab6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

在应用集nginx增加监控项

![在这里插入图片描述](https://img-blog.csdnimg.cn/1827d71f54824cb58dd482ebb6649b5c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

自定义触发器nginx-up-down，监控项为nginx-ping，正常为1，每5s监控一次，若为0，严重警告。

![在这里插入图片描述](https://img-blog.csdnimg.cn/3f7de86b8db44a47b74fe10bc7a60571.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/0d03478416bb4f30a3b1463f53d5b7ae.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

除nginx-ping，nginx-accepts外，需要添加所有状态监控项，只有nginx-ping创建触发器，这里不一一举例。

![在这里插入图片描述](https://img-blog.csdnimg.cn/3871000c1574457f849af703ecb4f016.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
添加好所有监控项，下一步制图，图中包含所有监控项

![在这里插入图片描述](https://img-blog.csdnimg.cn/f551b20c829744bd803cc8f5b6c376b3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

因为我们监控实际上就是在一直请求，所以看到nginx-requests在不断增加。

![在这里插入图片描述](https://img-blog.csdnimg.cn/6a3a0b5cd27e46138c1670228146c571.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

### 注册企业微信接口
注册成功之后创建一个运维部门

![在这里插入图片描述](https://img-blog.csdnimg.cn/84dc3eb218584473ba8916139c2f987b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

记住自己的企业ID

![在这里插入图片描述](https://img-blog.csdnimg.cn/9b43e8505f0c477093661019a56985ef.png)

自建应用

![在这里插入图片描述](https://img-blog.csdnimg.cn/ec1b6a3ab5db4e7bbab37ec4f4fbb7d1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

应用名称为zabbix监控

![在这里插入图片描述](https://img-blog.csdnimg.cn/429781b8cfc740b3b63b8927590c9e9e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_19,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/b6f6224055694779bcb38b285238b890.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

创建成功后，查看信息记住自己的AgentId和Secret

![在这里插入图片描述](https://img-blog.csdnimg.cn/ff466621447f4c31a0569927a16e2644.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

微信扫码企业微信插件就可以在微信接收消息

![在这里插入图片描述](https://img-blog.csdnimg.cn/0649b330e8a5472295873dbcdd7ebaa6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

### zabbix-server接口脚本配置告警
**Shell脚本**
```bash
cd /usr/lib/zabbix/alertscripts
进入脚本配置文件夹
vim weixin.sh
```

```bash
#!/bin/bash

CorpID="wwaa6fb8ff1b81aa77"     # 你的企业id
Secret="Cxyd*****"    #你的SecretID
GURL="https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=$CorpID&corpsecret=$Secret"
Token=$(/usr/bin/curl -s -G $GURL |awk -F\": '{print $4}'|awk -F\" '{print $2}')
# echo $Token
PURL="https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=$Token"

function body(){
        local int agentid=1000002               # 你的agentdid
        local UserID="@all"                 # 发送的用户ID
        local PartyID=1                  # 部门ID
        local Msg=$(echo "$@" | cut -d" " -f3-) # 发送给所有人
        printf '{\n'
        printf '\t"touser": "'"$UserID"\"",\n"
        printf '\t"toparty": "'"$PartyID"\"",\n"
        printf '\t"msgtype": "text",\n'
        printf '\t"agentid": "'"$agentid"\"",\n"
        printf '\t"text": {\n'
        printf '\t\t"content": "'"$Msg"\""\n"
        printf '\t},\n'
        printf '\t"safe":"0"\n'
        printf '}\n'
}
/usr/bin/curl --data-ascii "$(body $1 $2 $3)" $PURL
```
测试脚本能否正常接收消息

![在这里插入图片描述](https://img-blog.csdnimg.cn/0afcf63df19a428fa7c3b61fd7e2dbcd.png)

可以看到正常接收。

![在这里插入图片描述](https://img-blog.csdnimg.cn/329403b827f04c4087235e4315b898b2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

然后我们回到zabbix-server-web配置

### zabbix-server-web配置告警

管理->报警媒介类型->创建媒体类型

![在这里插入图片描述](https://img-blog.csdnimg.cn/897ee6b21c4e4c5abc5e008a6b5da1cc.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

创建用户群组

![在这里插入图片描述](https://img-blog.csdnimg.cn/acb48d9d6bfe42ff8e7172fc4ea5016e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

创建用户

![在这里插入图片描述](https://img-blog.csdnimg.cn/0df624972fb8430bb1e1dfa357bed27a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

报警媒介

![在这里插入图片描述](https://img-blog.csdnimg.cn/d4885af06d814c1492078a7005b22109.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/f70e97941482419cb5186b0ee05ed7fc.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

权限->超级管理员

![在这里插入图片描述](https://img-blog.csdnimg.cn/e5d9532fa8c047f89ad092ae4946cc21.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
配置->动作

添加触发器

![在这里插入图片描述](https://img-blog.csdnimg.cn/ef671d03aa39425784d8c773397c83d0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
操作

> 故障{TRIGGER.STATUS},服务器:{HOSTNAME1}发生{TRIGGER.NAME}故障!
> 告警主机:{HOSTNAME1}
告警时间:{EVENT.DATE} {EVENT.TIME}
告警等级:{TRIGGER.SEVERITY}
告警信息: {TRIGGER.NAME}
告警项目:{TRIGGER.KEY1}
问题详情:{ITEM.NAME}:{ITEM.VALUE}
当前状态:{TRIGGER.STATUS}:{ITEM.VALUE1}
事件ID:{EVENT.ID}

![在这里插入图片描述](https://img-blog.csdnimg.cn/7e15057e31eb431992c77c720a91169b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

恢复操作

> 恢复{TRIGGER.STATUS}, 服务器:{HOSTNAME1}: {TRIGGER.NAME}已恢复!
> 告警主机:{HOSTNAME1}
告警时间:{EVENT.DATE} {EVENT.TIME}
告警等级:{TRIGGER.SEVERITY}
告警信息: {TRIGGER.NAME}
告警项目:{TRIGGER.KEY1}
问题详情:{ITEM.NAME}:{ITEM.VALUE}
当前状态:{TRIGGER.STATUS}:{ITEM.VALUE1}
事件ID:{EVENT.ID}

![在这里插入图片描述](https://img-blog.csdnimg.cn/28715fe509a544398c50a9cf98fcc0bb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

### zabbix-agent关停nginx服务测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/f8fd28f8bfa84658bfeda8b21370d0b6.png)

可以看到推送成功

![在这里插入图片描述](https://img-blog.csdnimg.cn/bfa00a728a1a4208b45cb4db5f072d11.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

#### Python webhook机器人脚本
新建一个测试群聊，在群里添加机器人，记住webhook地址
![在这里插入图片描述](https://img-blog.csdnimg.cn/92a6332fd2394ec38c71333b25222e79.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

除了使用Shell脚本配置接口之外，还可以使用Python脚本

```python
#!/usr/bin/python
#-*- coding: utf-8 -*-
import requests
import json
import sys
import os

headers = {'Content-Type': 'application/json;charset=utf-8'}
api_url = "https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=****”
#填写自己的webhook地址
def msg(text):
    json_text= {
     "msgtype": "text",
        "text": {
            "content": text
        },
    }
    print requests.post(api_url,json.dumps(json_text),headers=headers).content

if __name__ == '__main__':
    text = sys.argv[1]
    msg(text)
              
```

测试

```bash
python weixin.py test
```
web配置同上

![在这里插入图片描述](https://img-blog.csdnimg.cn/e0e7950ce72b4f998f176524493392b6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
