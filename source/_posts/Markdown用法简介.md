---
title: Markdown用法简介
date: 2022-04-06 18:28:43
index_img: https://www.runoob.com/wp-content/uploads/2019/03/iconfinder_markdown_298823.png
tags: [Hexo, Fluid]
categories: 
   -[练习]
---

本文章用来练习hexo写博客，顺便练习Markdown语法。

<!-- more -->

taskkill /F /IM node.exe
hexo创建新博客文章：hexo new "第一篇文章"
# Markdown简介
Markdown是一种轻量级标记语言，创始人为约翰·格鲁伯。 它允许人们使用易读易写的纯文本格式编写文档，然后转换成有效的XHTML（或者HTML）文档。这种语言吸收了很多在电子邮件中已有的纯文本标记的特性。旨在令人们可以专注于编辑内容，而不会因为编辑格式而分心。

# 使用的编辑工具和插件
因为不想多下一个软件，所以用vscode来编辑Markdown。

使用的vscode扩展：

1. **Markdown ALL in One**（包含几乎所有Markdown相关功能）
2. **Markdown Preview Enhanced**（提供Markdown渲染后的预览，以及可以方便的导出pdf）
3. **Paste Image**（直接从剪切板粘贴图片）
4. **Code Spell Checker**（检查拼写错误）
   
# 基本的语法练习
## 二级标题
### 三级标题
正文

空一行换行表示另起一段，  
打两个空格再换行表示换行但不另起一段

## 强调
两个*加粗 **加粗**

一个*斜体 *斜体*

## 列表
1. 一
2. 二
   1. 二级列表

## 图片

![](/img/bg/asoul.jpg)  
*图片的说明文字（caption）原作者：未确认_Sora*

图片并排显示
![](/article_img/2023-01-30-20-10-34.png) | ![](/article_img/2023-01-30-20-12-26.png) | ![](/article_img/2023-01-30-20-14-01.png) | ![](/article_img/2023-01-30-20-18-46.png)
---|---|---|---

## 公式（支持latex公式书写）


$$
\lim_{x\to\ 0} \frac{sin(x)}{x}=1
$$


## 表格

| 向晚  | 贝拉  | 乃琳  |
| :---: | :---: | :---: |
|   1   |   2   |   3   |
|   4   |   5   |   6   |

**shift+alt+f** 可以格式化,让markdown编辑页面的表格同样美观

## 链接

这是一个[链接](https://www.bilibili.com/video/BV19Z4y1k7P7?spm_id_from=333.999.0.0)
## 代码
```python
def hello(){
   print("hello world!")
}
```
# vscode插件及设置

## 1. paste image

"pasteImage.BasePath": "${projectRoot}"

"pasteImage.path": "${projectRoot}/article_img"

"prefix": "/"

截图快捷键（windows自带）：**win+shift+s**

粘贴快捷键：**alt+ctrl+v**

## 2. Preview Markdown

# 参考资料
[Fluid主题文档](https://hexo.fluid-dev.com/docs/start/)

[Hexo官方文档](https://hexo.io/zh-cn/docs/)

b站视频：
1. [教你Markdown+VSCODE实现最完美流畅写作体验](https://www.bilibili.com/video/BV1si4y1472o?spm_id_from=333.337.search-card.all.click)
2. [【2021最新版】保姆级Hexo+github搭建个人博客](https://www.bilibili.com/video/BV1mU4y1j72n?spm_id_from=333.999.0.0)
3. [手把手教你从0开始搭建自己的个人博客 |无坑版视频教程| hexo](https://www.bilibili.com/video/BV1Yb411a7ty?spm_id_from=333.999.0.0)

文章：
1. Markdown: https://www.limfx.pro/ReadArticle/57/yi-zhong-xie-zuo-de-xin-fang-fa
2. latex书写数学公式：https://blog.csdn.net/weixin_42373330/article/details/89785443