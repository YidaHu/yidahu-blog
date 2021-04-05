## Git服务器搭建并自动部署Hexo到云主机与Github

## 前言

本文旨在搭建一个自己的博客在腾讯云主机上，然后把本地的Hexo生成的静态页面发布到服务器，动态的自动部署。

自动化部署可以省去很多麻烦的操作，更为重要的是按照流程操作，可以减少各种错误。以往都是Hexo生成静态页面然后通过Ftp方式上传，麻烦而且不高效。

目前我的做法是通过在云主机搭建Git服务器，本地将markdown文件渲染成静态文件，然后通过Git推送到服务器的repository，而服务器在使用Git Hook来同步到Nginx的网站目录。

<!--MORE-->

---

- 前提
- 服务器配置
- 本地客户机配置
- 搭建Git服务器
- Nginx配置
- 本地配置
- 自动化部署
- 自动上传到Github
- 常见问题



## 前提

- 云主机
- 域名

## 服务器配置

- Git
- Node.js
- Nginx

---



## 本地客户机配置

本地客户机用于写文章等操作

- Hexo
- Git
- Node.js

---



## 搭建Git服务器

### 1. 创建git用户

~~~
adduser git
passwd git
chmod 740 /etc/sudoers
vim /etc/sudoers
~~~

找到以下内容

~~~
## Allow root to run any commands anywhere
root    ALL=(ALL)     ALL
~~~

添加一行

~~~
git ALL=(ALL) ALL
~~~

保存退出后改回权限

~~~
chmod 400 /etc/sudoers
~~~

切换至 git 用户，创建`~/.ssh`文件夹和`~/.ssh/authorized_keys`文件，并赋予相应的权限

~~~
su git
mkdir ~/.ssh
vim ~/.ssh/authorized_keys
~~~

然后在本地主机中查找`id_rsa.pub`，我的是Windows，在`C:\Users\huyd\.ssh`目录下，如果没有`id_rsa`(私钥)与`id_rsa.pu`b（公钥），使用如下代码生成。请注意，为了方便后面SSH免密登录顺利，在生成公钥过程中，一路回车，暂不设置key密码。

~~~
//如果没有.ssh文件请自行创建。生成后为C:\Users\用户名\.ssh\id_rsa.pub（注：此处为本地机器）
ssh-keygen -t rsa -C "xxx@gmail.com"	//这里邮箱自己写
~~~

然后将本地机器生成的id_rsa.pub里的内容复制到Git服务器中`authorized_keys`（注：一行为一个用户的公钥），目录位置为`~/.ssh/authorized_keys`

这里由于Git服务器待会也要使用Git的功能，所以也需要在服务器中创建id_rsa.pub，操作与上面一致

~~~
//如果没有.ssh文件请自行创建。生成后为~/.ssh/id_rsa.pub（此处为云主机服务器）
ssh-keygen -t rsa -C "xxx@gmail.com"	//这里邮箱自己写
~~~

同样，然后将服务器中生成的id_rsa.pub里的内容复制到Git服务器中`authorized_keys`（注：一行为一个用户的公钥），目录位置为`~/.ssh/authorized_keys`

更改Git服务器中`authorized_keys`权限

~~~
chmod 600 ~/.ssh/authorzied_keys
chmod 700 ~/.ssh
~~~

然后就可以执行ssh 命令测试是否可以免密登录

~~~
ssh -v git@SERVER	//这里SERVER填上服务器IP地址
~~~

到这里，Git服务器基本就搭建完成了。

---



## Nginx配置

此处默认服务器已安装好Nginx，具体百度Nginx安装

~~~
server
{
    listen 80;
    #listen [::]:80;
    server_name www.huyidada.com huyidada.com;
    index index.html index.htm index.php default.html default.htm default.php;
    #这里要改成网站的根目录
    root  /path/www;  

    include other.conf;
    #error_page   404   /404.html;
    location ~ .*\.(ico|gif|jpg|jpeg|png|bmp|swf)$
    {
        access_log   off;
        expires      1d;
    }

    location ~ .*\.(js|css|txt|xml)?$
    {
        access_log   off;
        expires      12h;
    }

    location / {
        try_files $uri $uri/ =404;
    }

    access_log  /home/wwwlogs/blog.log  access;
}
~~~

----



## 本地配置

这里为本地机器，我的系统为Windows10

Hexo环境搭建，具体搭建不是本文内容，如果没有可以参考我的[《Hexo配置使用与主题配置》](http://123.207.183.26/2016/01/03/hello-world/)

~~~
npm install -g hexo-cli   //安装hexo-cli
cd E:
mkdir hexo
cd hexo
hexo init                 //初始化hexo
~~~



下面进行部署自动化

安装插件

- hexo-deployer-git 	// Git自动部署
- hexo-server                  //本地简单服务器（默认已安装）

~~~
cd hexo
npm install hexo-deployer-git --save
npm install hero-server
~~~

生成文章

~~~
hexo new "hello world"
vim sources/_posts/hello-world.md
~~~

编辑完后，使用`hexo g`将`.md`文件渲染成静态文件，然后启动`hexo s`

~~~
hexo g
hexo server
~~~

简单博客搭建好了

---



## 自动化部署

使用 Git Hook 自动部署 Hexo 到云主机

### 服务器上建立git裸库

创建一个裸仓库，裸仓库就是只保存`git`信息的`Repository`, 首先切换到`git`用户确保`git`用户拥有仓库所有权 一定要加 `--bare`，这样才是一个裸库。

~~~
su git
cd ~
git init --bare blog.git
~~~

### 使用 git-hooks 同步网站根目录

在这里我们使用的是`post-receive`这个钩子，当git有收发的时候就会调用这个钩子。 在 `~/blog.git` 裸库的 `hooks`文件夹中， 新建`post-receive`文件。

~~~
vim ~/blog.git/hooks/post-receive
~~~

在`post-receive`文件中添加如下

~~~
#!/bin/bash
GIT_REPO=/home/git/blog.git	//仓库地址
TMP_GIT_CLONE=/tmp/hexo
PUBLIC_WWW=/path/www	//这里Nginx网站根目录
rm -rf ${TMP_GIT_CLONE}
git clone $GIT_REPO $TMP_GIT_CLONE
rm -rf ${PUBLIC_WWW}/*
cp -rf ${TMP_GIT_CLONE}/* ${PUBLIC_WWW}
~~~

保存后，要赋予这个文件可执行权限

~~~
chmod +x post-receive
~~~

### 本地配置_config.yml,完成自动化部署

然后打开 `_config.yml`, 找到 `deploy`

~~~
deploy:
    type: git
    repo: git@SERVER:/home/git/blog.git    //<repository url>
    branch: master            //这里填写分支   [branch]
    message: 提交的信息         //自定义提交信息 (默认为 Site updated: {{ now('YYYY-MM-DD HH:mm:ss') }})
~~~

保存后，尝试将我们刚才写的"hello world"部署到服务器

~~~
hexo clean  //清除以前的静态html
hexo generate --deploy  //生成html并发布
~~~

访问服务器地址，就可以看到我们写的文章"Hello hexo",以后写文章只需要

~~~
hexo new "Blog article name"
···写文章
hexo clean && hexo generate --deploy
~~~

这里写文章后部署，开始可能有点慢，访问网站还没有效果，这可能和后台上传有关，稍等一会儿访问就好。

---



## 自动上传到Github

本地博客上传到github，做备份

其实就是简单的几条命令，如下

~~~
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/huyida/huyida.me.git	//这里写上自己的github仓库地址
git remote -v
git pull --rebase origin master
git push origin master
~~~

---

## 常见问题

### SSH信任关系建立后仍需要输入密码

**原因分析，以及处理步骤：**

1  查看 log/secure，分析问题在何处；检查/var/log/messages

2  查看 /root/.ssh/authorized_keys文件的属性，以及.ssh文件属性   是不是权限过大。.ssh目录的权限必须是700，同时本机的私钥的权限必须设置成600：

3  修改/etc/ssh/sshd_config文件,  把密码认证关闭, 将认证改为 passwordAuthentication no   重启下sshd。 service sshd restart;（设置后貌似只能ssh git登录）

4  执行setenforce 0,暂时关闭selinux