## 语法规则

```
Syntax:	location [ = | ~ | ~* | ^~ ] uri { ... }
location @name { ... }
Default:	—
Context:	server, location
```
## 优先级

自上而下

location = /uri 精准匹配
---
location ^~ /uri  前缀匹配
---
location ~ 区分大小写的正则匹配
---
location ~* 不区分大小写的正则匹配
---
location /uri 不带修饰的前缀匹配
---
location / 通用匹配
---
