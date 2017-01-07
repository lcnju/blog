---
title: 配置hexo自动部署
date: 2017-01-07 09:40:05
tags:
  - hexo
  - hudson
---

# 将博客源码上传到github

新建`.gitignore`文件,添加如下内容,然后将源码上传到github即可
```gitignore
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```

# hudson添加hexo博客项目

在服务器上拉取github中的博客内容
```bash
# cd ~
# git clone https://github.com/lcnju/blog.git
```

为blog文件夹添加tomcat7用户的读写权限
```
# setfacl -d -R -m user:tomcat7:rwx ~/blog
```

hudson新增webhook账号,配置只读权限
![](/images/2017_1_7_add_hudson_user.png)

hudson新增空项目,配置webhook,此处输入一个较长且随机的字符串作为token提供给github.
![](/images/2017_1_7_hudson_webhook.png)

hodson新增build步骤*Execute shell*
```bash
cd /home/ubuntu/blog/
git pull
hexo clean
hexo g
```

ngxin中配置hexo博客的代理.
```nginx
server {
    listen 80;

    location / {
        root /home/ubuntu/blog/public;
        index index.html;
    }
}
```

此时点击立刻构建后,hexo即可发布到公网中.

# github配置webhook
依次点击Settings -> Webhooks -> Add webhook,在Payload URL中填入.替换大写的常量,可以复制到浏览器中点击看是否触发自动构建任务.
```http
http://ACCOUNT:PASSWORD@HUDSON_URL/job/blog/build?token=TOKEN
```

# 结语
至此hexo的持续集成已经配置完毕,只需在本地将项目提交到github后,会触发hudson的webhook,然后会发布新的博客内容到公网中.
