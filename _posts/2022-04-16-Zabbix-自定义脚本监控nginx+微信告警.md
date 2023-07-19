---
layout: article
title: Zabbix-自定义脚本监控nginx+微信告警
tags: Zabbix Nginx Project
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

> http://192.168.2.58/wyt_status

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

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/fd38b70d-9434-40aa-9c46-d34c52b05414)


添加应用集nginx

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/5a22a919-e0b2-45f7-b8ee-0b34b7d87359)


在应用集nginx增加监控项

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/7fefb9ef-56d7-4a90-aa80-787447a060ee)


自定义触发器nginx-up-down，监控项为nginx-ping，正常为1，每5s监控一次，若为0，严重警告。

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/09cd94c2-70c2-4825-a599-331a7aab8b69)

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/792ad9f2-ec53-4389-9701-4585aada74f6)


除nginx-ping，nginx-accepts外，需要添加所有状态监控项，只有nginx-ping创建触发器，这里不一一举例。

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/00fe1af4-da25-4e66-9dc0-a7b34a82ed59)

添加好所有监控项，下一步制图，图中包含所有监控项

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/40e3a3c4-df6d-4220-a17b-e6ae3324b076)


因为我们监控实际上就是在一直请求，所以看到nginx-requests在不断增加。

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/3fe74953-319c-4402-acbe-b55af69c38ce)


### 注册企业微信接口
注册成功之后创建一个运维部门

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/499ce58a-d846-48ed-beda-05d35d0d32de)


记住自己的企业ID

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/269361dd-fead-46f2-b256-ac3a44c90ae4)

自建应用

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/58a9426a-f362-4067-ac7e-1e735f1d8fd3)

应用名称为zabbix监控

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/75319c7c-ea85-43dd-ad66-b6b2bb913c67)

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/2dbe4bc3-1d77-4b66-a12a-b7bd54269ce7)


创建成功后，查看信息记住自己的AgentId和Secret

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/a8d2443d-bd0f-47f9-8dbe-c9be0122ce68)


微信扫码企业微信插件就可以在微信接收消息

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/2d71225f-c6d6-436a-94c9-0cc13e6066e3)

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
```bash

#!/bin/bash
corpid="wwaa6fb8ff1b81aa77"	#企业id
corpsecret="CxydMDAEscOL7Hqbqphf77rY3HpVsJDu4cizLwg-YMI"	#你的corpsecret
url="https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=$corpid&corpsecret=$corpsecret"
#echo $url
token=$(curl -s $url|awk -F'"' '{print $10}')	#截取token
#echo $token
#echo $url2
body(){
        printf '{'
        printf '"touser": "@all",'
        printf '\t"msgtype": "text",'
        printf '\t"agentid": "1000002",'    #你的agentid
        printf '"text": {'
        printf '"content": '"$1"
        printf '},'
        printf '"safe":"0"'
        printf '}'
} 
curl -d "$(body $1)" https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=$token -s
```
测试脚本能否正常接收消息

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/68548968-ce2b-4691-b37c-05f4dd2a43b2)


可以看到正常接收。

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/eebf22e9-56d1-4cc0-b1ee-f5ee26855cfe)

然后我们回到zabbix-server-web配置

### zabbix-server-web配置告警

管理->报警媒介类型->创建媒体类型

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/fd00f124-120f-462e-8d5a-d7206d0ab850)


创建用户群组

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/61b7d208-a421-4a33-b16f-624e9afbae34)

创建用户


![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/3308f2e6-1bdf-4a0e-8d92-0f43768da24e)

报警媒介

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/ba61e2b4-914c-468b-9ba0-5c03d29ddb66)


![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/282ac6ef-0f73-4441-b6ec-68591ee9d539)

权限->超级管理员

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/af67a7a1-4fc2-408c-84dc-e3bd92626c65)

配置->动作

添加触发器

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/35fd1b80-d326-493a-bdb2-4710418a723d)

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

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/b06f50df-c0ec-490e-a118-15612b23dfc5)

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

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/d5444fa1-3775-417a-a2a9-f8ea52afd202)


### zabbix-agent关停nginx服务测试

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/64447777-e5b5-44c3-b4ad-811d7e40c9d8)


可以看到推送成功

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/bf1e5259-48d0-44ea-951c-d9181c946122)


#### Python webhook机器人脚本
新建一个测试群聊，在群里添加机器人，记住webhook地址

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/aa58cd07-868d-4c56-9f3d-b2dcaf3ecbb9)


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
![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/a9136ca7-bbb5-4dbe-bfde-ab36b02e4ca1)

