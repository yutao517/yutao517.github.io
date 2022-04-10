## 限制速度
添加  

```bash
   limit_rate 50k;限制速度50K
   limit_rate_after 1000k;当下载超过1000k时
```

```bash
 location  /mp3 {
         root   html/wyt;
         limit_rate 50k;
         limit_rate_after 1000k;
         auth_basic "wyt";
         auth_basic_user_file htpasswd;
         autoindex on;
          }
```

```bash
dd if=/dev/zero of=100M.dd bs=1M count=100 #建立一个100M的数据文件提供下载
```



## 限制连接数
**限制并发连接数**
```bash
http {
limit_conn_zone $binary_remote_addr zone=perip:10m; 
#放到http
server {
    ...
    limit_conn perip 1;#只限制一个IP地址访问
}
```
建立第二个请求
报503错误
![在这里插入图片描述](https://img-blog.csdnimg.cn/c67336a7a449441ab795350c87186b2d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

说明限制，服务器临时不可用

**突发限制访问频率** 

```bash
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s; #1s一次

    ...

    server {

        ...

        location /mp3/ {
            limit_req zone=one burst=2;
        }
```
Nginx的限流都是基于漏桶算法

**漏桶算法和令牌桶算法**

漏桶算法：水先进入到漏桶里，漏桶以一定的速度出水，当水流入速度过大会直接溢出
令牌桶算法：以一个恒定的速度桶会产生令牌，而如果请求需要被处理，则需要先从桶里获取一个令牌，当桶里没有令牌可取时，则拒绝服务。
