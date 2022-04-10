## Git的安装使用

### 1. Git客户端安装
* Windows安装
	* 下载地址: [https://git-scm.com/download/win](https://git-scm.com/download/win)
* Linux安装
     * yum install git


### 2. Git基本配置查看命令
* 新建Git项目

* 克隆项目

```bash
$ git clone https://gitee.com/yutao_517/pirate-king-2022.git
```
* 配置用户信息

```bash
(master) $ cd E:/Git/pirate-king-2022
(master) $ git config user.name 'yutao_517'
(master) $ git config user.email '1271030564@qq.com'
```

* 查看分支-当前分支前面会有个星号（`*`）

```bash
(master) $ git branch -a
```

* 查看远程源
```bash
(master) $ git remote 
```
* 添加远程源
```bash
(master) $ git remote add teacher https://gitee.com/yutao_517/*****.git
```

* 再次查看远程目录的位置
```bash
(master) $ git remote  
```

### 3. Git基本操作命令


* 从指定remote upstream拉取最新的代码合并到自己本地master分支上.

```bash
git pull teacher master
```

* 新建分支

```bash
$ git branch test
```

* 切换分支

```bash
$ git checkout test
```
* 修改文件
```bash
$ echo 'xxx' >> README.md
```

* 添加待提交文件到区

```bash
$ git add README.md
```

* 拉取最新的代码合并到自己本地当前分支

```bash
$ git pull orign master
```
* 提交到本地仓库
```bash
$ git commit -m 'wyt'
```
* 提交到远程仓库
```bash
$ git push
```
