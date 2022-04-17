---
layout: article
title: 计算机网络-ping局域网网段
tags: 计算机网络 Shell Python
category: blog
date: 2022-03-08 00:00:00 +08:00
mermaid: true
---

```bash
#!/bin/bash
#这是父进程#
>used_ip.txt
>unused_ip.txt
#清空文件
for i in {1..255}
do
	#()&生成多个子进程并行，提高ping的速度效率	
	(ping 192.168.2.$i -c 1 -w 1 &>/dev/null
	#ping 1次，时长为1s必须完成，然后将结果丢进黑洞文件
	if(($? == 0))#如果ping通,ping通的返回值为1.
	then
		echo "192.168.0.$i is used"
		echo "192.168.0.$i" >>used_ip.txt
		#ping通后的ip地址写入used_ip.txt
	else
		echo "192.168.0.$i" >>unusedip.txt
		#ping不通的ip地址写入unused_ip.txt
	fi)& 		
done
wait #明显父进程清空文件简单快速，所以父进程运行完需等待子进程
echo '########[used ip detail]################'
cat used_ip.txt
echo '#######[arp buffer table]###############'
num=$(cat used_ip.txt|wc -l)
#统计used_ip.txt的行数
echo "一共有$num个ip地址"
arp -a|awk '{print $2,$4}'|grep -v "incomplete"|tr -d "()"
#输出MAC地址截取第二个到第四个字段

```



```python
#!/usr/bin/python3
import subprocess
used_ip = []
unused_ip = []

for i in range(1,11):
        ip = subprocess.run(f"ping -c 1 -w 1 192.168.2.{i}",shell=True,stdout=subprocess.PIPE,stderr=subprocess.PIPE)
        if ip.returncode == 0:
                used_ip.append(f"192.168.0.{i}")
        else:
                unused_ip.append(f"192.168.0.{i}")
print("正在使用的IP地址如下：")
print(used_ip)
print("没有使用的IP地址如下：")
print(unused_ip) 
```

