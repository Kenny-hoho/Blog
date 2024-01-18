---
title: TortoiseGit配置
date: 2024-1-18
index_img: "/img/bg/TortoiseGit.jpg"
tags: [Git]
categories: 
   -[笔记]
---
使用TortoiseGit时遇到无法使用ssh链接进行git操作的问题，原因是TortoiseGit配置有问题，这里记录解决方法。
<!-- more -->

# git bash端口22报错解决方法

原因应该是端口22被禁用了，更换端口443，到 **/.ssh** 添加一个config文件（可以从别的地方复制一个config），在其中填写：
```C++
Host github.com
Hostname ssh.github.com
Port 443
User git
``` 
即可使用ssh进行正常的git操作；

# TortoiseGit无法使用ssh进行git操作

原因是TortoiseGit的配置不对，首先到 **设置->网络->SSH** 中设置ssh；
![](/article_img/2024-01-18-14-02-32.png)

之后需要为TortoiseGit配置与git相同的密钥，否则需要为TortoiseGit再单独生成一个密钥，再去github配置密钥；

回到常规设置，点击**重新运行首次启动向导**：
![](/article_img/2024-01-18-14-04-38.png)
选择密钥类型为OpenSSH（Git使用的密钥类型，与TortoiseGit默认的密钥类型不同）
![](/article_img/2024-01-18-14-04-59.png)
运行**PuTTYgen**，将git使用的密钥（一般在C:\Users\.ssh）导入并保存：
![](/article_img/2024-01-18-14-08-11.png)

完成！可以正常使用TortoiseGit！
