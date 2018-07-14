# 搭建一个 Git 服务器端仓库并且实现自动部署
## 创建一个 Git 服务器端仓库
### 1. 首先，你得要有一个服务器
### 2. 安装 `git`
```shell
sudo apt-get install git
```
### 3. 创建一个 `git` 用户，用来运行 `git` 服务
```shell
sudo adduser git
```
### 4. 初始化 GIT 仓库，假定是在 `/gitrepo/` 目录下
```shell
cd /gitrepo
sudo git init --bare sample.git
```
Git 会创建一个裸仓（--bare），裸仓只有配置信息，没有具体代码，为的是不让用户在服务器上直接改代码。
然后，把 owner 改为 `git`
```shell
sudo chown -R git:git sample.git
```
### 5. 添加公钥
将用户公钥导入到 `/home/git/.ssh/authorized_keys` 文件中，一行一个即可。
### 6. 用户克隆远程仓库
```shell
git clone git@server:/gitrepo/sample.gi
```
如果 authorized_keys 文件中没有添加该用户的 id_ras.pub 公钥，则会显示输入密码，即通过密码的验证方式克隆仓库。
## 使用 GitHook 实现自动部署
### 1. 将服务器上的 Git 仓克隆到 web 服务器的目录下，假定 web 服务器目录是 `/data/www/`
```shell
cd /data/www/
git clone /gitrepo/sample.git
```
### 2. 配置 Git Hook
进入到 `/gitrepo/sample.git/hooks/` 目录下，创建一个 `post-receive` 文件
post-receive:
```shell
#!/bin/sh

#判断是否是远程仓

IS_BARE=$(git rev -parse --is-bare-repository)
if [ -z "$IS_BARE" ]; then
echo >&2 "fatal: post-receive: IS_NOT_BARE"
exit 1
fi

unset GIT_DIR

DeployPath="/data/www/sample/"

echo "==============================================="
cd $DeployPath
echo "deploying the test web"

git fetch --all
git reset --hard origin/master
git pull

time=`date`
echo "web server pull at webserver at time: $time."
echo "================================================"
```
然后给 post-receive 文件添加可执行权限
```shell
sudo chmod +x post-receive
```
**注意：`/data/www/sample/` 下的文件必须具有可写权限**

参考资料：
[搭建Git服务器](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137583770360579bc4b458f044ce7afed3df579123eca000)
[服务器自动部署项目之GitHooks神器](https://blog.csdn.net/wsyw126/article/details/52167147)
