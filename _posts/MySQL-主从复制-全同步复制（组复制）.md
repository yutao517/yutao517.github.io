**全同步复制（组复制）**

复制组由多个 server成员构成，并且组中的每个 server 成员可以独立地执行事务；但所有读写（RW）事务只有在冲突检测成功后才会提交。只读（RO）事务不需要在冲突检测，可以立即提交。

MySQL 组复制提供了高可用性，高弹性，可靠的 MySQL 服务
但是组复制的效率很低当master节点写数据的时候，会等待所有的slave节点完成数据的复制，然后才继续往下进行；组复制的每一个节点都可能是slave。

- 单主模式（Single-Primary Mode）
在这种模式下，组有一个设置为读写模式的单主服务器。组中的所有其他成员都设置为只读模式。从复制组中众多个MySQL节点中自动选举一个master节点，只有master节点可以写，其他节点自动设置为read only当master节点故障时，会自动选举一个新的master节点，选举成功后，它将设置为可写，其他slave将指向这个新的master。
- 多主模式（ Multi-Primary Mode）
复制组中的任何一个节点都可以写，因此没有master和slave的概念，只要突然故障的节点数量不太多，（容忍n个故障所需的server数量2n+1）这个多主模型就能继续可用。

![在这里插入图片描述](https://img-blog.csdnimg.cn/af1cdb81dadd482d82f92ace10aabf08.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)


