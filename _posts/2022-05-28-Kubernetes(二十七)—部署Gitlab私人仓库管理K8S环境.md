---
layout: article
title: Kubernetes(二十七)—部署Gitlab私人仓库管理K8S环境
tags: Kubernetes Project
category: blog
date: 2022-05-28 20:43:00 +08:00
mermaid: true
---
## Git、Github、Gitlab的区别
Git是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。
Github是在线的基于Git的代码托管服务。 GitHub是2008年由Ruby on Rails编写而成。GitHub同时提供付费账户和免费账户。这两种账户都可以创建公开的代码仓库，只有付费账户可以创建私有的代码仓库。
Gitlab解决了这个问题, 可以在上面创建免费的私人repo。 

git是一套软件,可以做本地私有仓库。
github本身是一个代码托管网站 ，公有和私有仓库(收费) ，不能做本地私有仓库。
gitlab本身也是一个代码托管的网站，功能上和github没有区别，公有和私有仓库（免费），可以部署本地私有仓库。

## Git 与 SVN 区别
1. Git是分布式的，svn不是：这是GIT和其它非分布式的版本控制系统，例如SVN，CVS等，最核心的区别。
2. GIT把内容按元数据方式存储，而SVN是按文件：所有的资源控制系统都是把文件的元信息隐藏在一个类似.svn,.cvs等的文件夹里。
3. GIT分支和SVN的分支不同：分支在SVN中一点不特别，就是版本库中的另外的一个目录。
4. GIT没有一个全局的版本号，而SVN有：目前为止这是跟SVN相比GIT缺少的最大的一个特征。
5. GIT的内容完整性要优于SVN：GIT的内容存储使用的是SHA-1哈希算法。这能确保代码内容的完整性，确保在遇到磁盘故障和网络问题时降低对版本库的破坏。
6. Git是分布式的版本控制器，没有客户端和服务器端的概念。SVN它是C/S结构的版本控制器，有客户端和服务器端 ，服务器如果宕机而且代码没有备份的情况下，完整代码就会丢失。

## 部署Git服务

```bash
yum install git git-core gitweb -y
useradd git
passwd git
mkdir /git-root/
cd /git-root/
git init --bare code.git
#初始化
chown -R git:git code.git
# 权限
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/388922b725f246279728c307e4bbea33.png)

## git仓库测试

```bash
ssh-keygen
ssh-copy-id git@121.36.40.218
#配置免密连接git仓库
```

```bash
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```
```bash
git config --list
# 查看配置
```

```bash
git clone git@121.36.40.218:/git-root/code.git
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/39eae7a91b624a459cd27735ef13ff8d.png)

```bash
cd code
vim weixin.sh
git add weixin.sh
git commit -m 'first commit'
git push origin master
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/54e66065598f4ad69cc5d98c81cd2c98.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/5ed45d76d0244feebe5d57674b4e42f9.png)

## Git工作流程
一般工作流程如下：

- 克隆 Git 资源作为工作目录。
- 在克隆的资源上添加或修改文件。 
- 如果其他人修改了，你可以更新资源。
- 在提交前查看修改。
- 提交修改。
- 在修改完成后，如果发现错误，可以撤回提交并再次修改并提交。

## 配置更改
**远程仓库配置**

![在这里插入图片描述](https://img-blog.csdnimg.cn/85f06f6c6d4b4888b393e2f82ad24a84.png)

可以看到origin url，所以直接push origin master

**邮件名字配置**

![在这里插入图片描述](https://img-blog.csdnimg.cn/8399a0752b01477382263dca2322ce2c.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/f1e9cf58023a4a86898d6c2148558354.png)

## Git操作
1、如果你没有最新的代码，希望从头开始

```shell
git clone git@XXX.git      # 这里是项目的地址（可从项目主页复制），将远程服务器的内容完全复制过来 
cd BGBInspector_V01        # clone 之后进入该项目的文件夹 
touch　README.md           # 新建readme文件 
git add README.md          # 将新的文件添加到git的暂存区 
git commit -m ‘Its note：add a readme file’ # 将暂存区的文件提交到某一个版本保存下来，并加上注释 
git push -u origin master  # 将本地的更改提交到远程服务器
```

2、如果你已经有一个新版代码，希望直接把本地的代码替换到远程服务器

```shell
cd existing_folder          #进入代码存在的文件夹，或者直接在该文件夹打开
git init           # 初始化 
git remote add origin git@master:/git-test/code.git  #添加远程项目"code"库的地址（可从项目主页复制） ,前提是事先需要先在git远程服务器上创建相应的裸库"shell"
git add .                   #添加该文件夹中所有的文件到git的暂存区 
git commit -m ‘note’        #提交所有代码到本机的版本库 
git push -u origin master   #将本地的更改提交到远程服务器
git log           #查看提交日志
```

- git 中 clone过来的时候，git 不会对比本地和服务器的文件，也就不会有冲突，

- 建议确定完全覆盖本地的时候用 clone，不确定会不会有冲突的时候用 git pull，将远程服务器的代码download下来

![在这里插入图片描述](https://img-blog.csdnimg.cn/97f8e17693004ef7ae2c0fb60f021cdc.png)

```bash
git init                      # 初始化 
git add main.cpp              # 将某一个文件添加到暂存区 
git add .                     # 将文件夹下的所有的文件添加到暂存区 
git commit -m ‘note‘          # 将暂存区中的文件保存成为某一个版本 
git log                       # 查看所有的版本日志 
git status                    # 查看现在暂存区的状况 
git diff                      # 查看现在文件与上一个提交-commit版本的区别 
git reset --hard HEAD^        # 回到上一个版本 
git reset --hard XXXXX        # XXX为版本编号，回到某一个版本 
git pull origin master        # 从主分支pull到本地 
git push -u origin master     # 从本地push到主分支 
git pull                      # pull默认主分支 
git push                      # push默认主分支 ...
```

## 版本控制

```bash
git show #查看当前版本
git reflog #查看历史版本
git reset a02ab39 #切换版本
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/e2358a0a2feb4898bf2e4a7946174a53.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/a16222b9ce1e447abdb27ecda596830a.png)

## 分支管理
**创建切换分支**
```bash
git checkout -b dev  # #创建dev分支，然后切换到dev分支，-b等于以下两条命令
git branch dev
git checkout dev
```

```bash
git branch #列出所有分支
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/13abe962eac143f0bdaf0f4a1402c6ff.png)

**合并分支**

```bash
git merge dev 		
#把dev分支的工作成果合并到master分支上
git merge               
#命令用于合并指定分支到当前分支。
```
**删除分支**

```bash
git branch -d dev 
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f2d73d8570254b4badd647250211b68a.png)

**解决冲突**

```bash
git checkout -b feature1        # 创建新的feature1分支
# 修改readme.txt最后一行，改为：
Creating a new branch is quick AND simple.

git add readme.txt              # 在feature1分支上提交
git commit -m "AND simple"
git checkout master             #切换到master分支

Switched to branch 'master' Your branch is ahead of 'origin/master' by 1 commit.
#Git还会自动提示我们当前master分支比远程的master分支要超前1个提交。

#在master分支上把readme.txt文件的最后一行改为：
Creating a new branch is quick & simple.
git add readme.txt 
git commit -m "& simple"

#现在，master分支和feature1分支各自都分别有新的提交这种情况下，Git无法执行“快速合并”，只能试图把各自的修改合并起来，但这种合并就可能会有冲突，我们试试看：
git merge feature1 Auto-merging readme.txt CONFLICT (content): 
Merge conflict in readme.txt Automatic merge failed; 
fix conflicts and then commit the result.

#readme.txt文件存在冲突，必须手动解决冲突后再提交。
git status   #可以显示冲突的文件;
#直接查看readme.txt的内容：
Git is a distributed version control system.
Git is free software distributed under the GPL. 
Git has a mutable index called stage. 
Git tracks changes of files. 
<<<<<<< HEAD Creating a new branch is quick & simple. ======= Creating a new branch is quick AND simple. >>>>>>> feature1
#Git用<<<<<<<，=======，>>>>>>>标记出不同分支的内容，我们修改后保存再提交：
git add readme.txt  
git commit -m "conflict fixed" 
[master 59bc1cb] conflict fixed
#最后，删除feature1分支：
git branch -d feature1 
Deleted branch feature1 (was 75a857c).
```

## 部署Gitlab
**安装 gitlab 依赖包**
```bash
yum install -y curl openssh-server openssh-clients postfix cronie policycoreutils-python
```
**添加yum源**

```bash
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
```
官方源太慢，可以使用国内清华yum源，配置如下

```bash
vim /etc/yum.repos.d/gitlab-ce.repo
```
```bash
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1
```

**安装Gitlab**

```bash
yum -y install gitlab-ce
```
因为下载太慢我使用rpm包安装

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-10.5.7-ce.0.el7.x86_64.rpm
```
```bash
rpm -ivh gitlab-ce-10.5.7-ce.0.el7.x86_64.rpm
```

```bash
head -1 /opt/gitlab/version-manifest.txt
#查看gitlab版本
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/7a184c0bf7e549569665b8527c870b19.png)


**初始化 Gitlab**
gitlab要求语言环境为英文环境，必须切换，切换方法如下

```bash
cat <<EOF >/etc/profile.d/locale.sh
export LANG=en_US.UTF-8
export LANGUAGE=en_US.UTF-8
export LC_COLLATE=C
export LC_CTYPE=en_US.UTF-8
EOF
source /etc/profile.d/locale.sh
```

```bash
vim /etc/gitlab/gitlab.rb
# 修改为自己的IP地址
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/32cf2e6a794740bcbef4de475b4201ea.png)

```bash
gitlab-ctl reconfigure   
#初始化，需要一定时间
gitlab-ctl restart
#启动gitlab
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/81aab84a8b4c4000951a960dd51b6b72.png)


**访问Gitlab页面**
如果你的服务器性能不佳会报502错误

![在这里插入图片描述](https://img-blog.csdnimg.cn/0b4b9faebc9840a2b26623e7f2a7eaab.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/5eeae206d1bb4f02af0a86fa5f7a63f1.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/ffd6378fcb824c499f1ca6a4ffe1f783.png)


## Gitlab 添加smtp邮件功能
**获取qq邮箱授权码**

![在这里插入图片描述](https://img-blog.csdnimg.cn/40d716ad444a43e5a10a20e4f06613a4.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/1b6fe8392b7a4028a869d5a20abe1536.png)

```bash
vim /etc/gitlab/gitlab.rb
```
```bash
gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = '1271030564@qq.com'
gitlab_rails['gitlab_email_display_name'] = 'gitlab'
gitlab_rails['gitlab_email_reply_to'] = '1271030564@qq.com'
gitlab_rails['gitlab_email_subject_suffix'] = '[gitlab]'
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.qq.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "1271030564@qq.com"
gitlab_rails['smtp_password'] = "ngdkzettpeuzhicg" #这是我的qq邮箱授权码
gitlab_rails['smtp_domain'] = "smtp.qq.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
```
**重新初始化**
```bash
gitlab-ctl stop
gitlab-ctl reconfigure
gitlab-ctl start
```
gitlab发送邮箱测试

```bash
gitlab-rails console 
Notify.test_email('1271030564@qq.com', 'Message Subject', 'Message Body').deliver_now
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/c6b9add4b6c24df1b0750a3fe6f8ca8e.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/0002bab850724e68aaf26d27b6f27017.png)

## 使用Gitlab

机器内存不足一般报错

![在这里插入图片描述](https://img-blog.csdnimg.cn/4a0fe56ba2604d6283a4ad963ae33006.png)

**创建项目组**
![在这里插入图片描述](https://img-blog.csdnimg.cn/358e69a6721c49af8fe626cb5e10199e.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/9bccf1d0be0245febd6459a72e97f293.png)

**创建用户**

![在这里插入图片描述](https://img-blog.csdnimg.cn/8f540e976eeb4f28a7dfbf9553001797.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/c01192d76b9d49c88a290fb5213061e7.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/5dcf7b6b64d84c4d9f4c0927e99e76f4.png)

点击邮箱链接更改密码之后收到更改成功的邮件

![在这里插入图片描述](https://img-blog.csdnimg.cn/1860eff0a2954d4a98c1619597712889.png)

之后用test登录

![在这里插入图片描述](https://img-blog.csdnimg.cn/47cc2750454642589430853df3bf0d2f.png)
**添加用户到组**

使用admin用户登录，添加test，test2用户到组gitlab，test为Owner，test2为Developer

![在这里插入图片描述](https://img-blog.csdnimg.cn/6c32ff5af51246d4816c7bdee4d7a477.png)

使用test登录创建项目testproject

![在这里插入图片描述](https://img-blog.csdnimg.cn/8886ac5951664990b0409fc067d966c2.png)

生成公钥

```bash
useradd test
su test
ssh-keygen
cd .ssh/
cat id_rsa.pub
```
复制公钥到web

![在这里插入图片描述](https://img-blog.csdnimg.cn/c6d0a5916db149f8b811508430534436.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/a0376cbf5ab64245b7c45318b662d008.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2898e48551da478295ae0b64d80b8707.png)
**添加项目**

分支master添加文件a.txt

![在这里插入图片描述](https://img-blog.csdnimg.cn/276f6691c67d418a84ae98146f3c6679.png)

创建新分支dev

![在这里插入图片描述](https://img-blog.csdnimg.cn/39aefb7357194154902e82e48e578272.png)

分支dev添加文件b.txt

![在这里插入图片描述](https://img-blog.csdnimg.cn/76836571b84d4ecc8e7c139d944cfaaf.png)

**合并分支**
![在这里插入图片描述](https://img-blog.csdnimg.cn/0ad2243021bb44a09c843c08726b76ce.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/c93a700b53414fee9edefff87deb938c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b7f4a38eaa1c456895ec454a0ebb2903.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/41e6a7abd97f4306bc3323d546962c11.png)

**Git操作**

```bash
git clone git@192.168.2.159:test/testproject.git
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/580e91db3eaa48d09b26c80d766c3e47.png)

修改a.txt内容为tao，提交上传

![在这里插入图片描述](https://img-blog.csdnimg.cn/a67a06843c304a3d8622bf1143f6e710.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/8de8048c4b8c4eadbcbf98d6c2bb3a03.png)

## 利用Gitlab管理k8s集群

权限设置



**获取k8s集群API地址**

```yaml
kubectl cluster-info 
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/94390ae728de46dca553281afd1b13ff.png)


**获取k8s集群默认CA证书**

```
kubectl get secrets
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ab2ccc3f4b1a4ef5959bca21d24473da.png)

default-token-ll4sp为上面获取到的secrets的名称，用以下命令查看证书

```
kubectl get secret default-token-ll4sp  -o jsonpath="{['data']['ca\.crt']}" | base64 --decode
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/6f60316c80cc4db0a46148e19a1aa621.png)

**设置rbac**

```bash
vim gitlab-admin-service-account.yaml
```

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: gitlab-admin
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: gitlab-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: gitlab-admin
  namespace: kube-system
```

```bash
kubectl apply -f gitlab-admin-service-account.yaml
```

**获取gitlab-admin的token**

```yaml
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep gitlab-admin | awk '{print $1}')
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/3bced809873644a28f2e41f5caced275.png)
