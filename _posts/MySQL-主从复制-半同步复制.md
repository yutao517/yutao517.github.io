**半同步复制**(semi sync)：master每commit一个事务(简单来说就是做一个改变数据的操作）,要确保slave接受完主服务器发送的二进制日志文件并写入到自己的中继日志relay log里，然后会给master ACK确认信号，告诉对方已经接收完毕，这样master才能把事物成功commit。
解决了异步模式里主从可能存在数据不一致的问题，主库有数据，而从库没有接收到二进制日志，保证了master-slave的数据绝对的一致（但是以牺牲master的性能为代价)，等待ACK确认时间也是可以调整的，默认10s。
![在这里插入图片描述](https://img-blog.csdnimg.cn/9b04a896c3844dcea80e32cd4d1a5fe3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/517b85bd10f04c34b27348e515ef988b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_19,color_FFFFFF,t_70,g_se,x_16)

**步骤**

前提：配置好异步操作
主服务器
```bash
mysql>INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so'; #安装插件
mysql>SET GLOBAL rpl_semi_sync_master_enabled = 1; #临时开启半同步
```
从服务器
```bash
mysql>INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';
mysql>SET GLOBAL rpl_semi_sync_slave_enabled = 1; #临时开启半同步
mysql>stop slave;
mysql>start slave;
```
也可以加到配置/etc/my.cnf
![在这里插入图片描述](https://img-blog.csdnimg.cn/7be74a63e6f64c5784d418b198545b2e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_18,color_FFFFFF,t_70,g_se,x_16)
