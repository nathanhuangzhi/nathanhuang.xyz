---
title: 通过 RStudio 搭建个人主页
author: Nathan Huang
date: '2021-09-17'
slug: personal-website
description: 一直想搭建一个自己的个人主页，David Perell 说过创作者需要 Own your distribution，不然只是在给各个平台的导流。之前折腾了好几次主页，但都不是很满意。在海外交流不管是公司还是个人，有自己的主页是非常有必要的。
categories: 
  - R
tags:
  - Blogdown
draft: no
---

# 搭建流程

一直想搭建一个自己的个人主页，David Perell 说过创作者需要 Own your distribution，不然只是在给各个平台的导流。在海外交流不管是公司还是个人，有自己的主页是非常有必要的。

之前折腾了好几次主页，但都不是很满意。这次试着记录下自己的步骤，以免日后忘记。搭建框架上常见的有 Jekyll，Hexo，Hugo 可以选，不折腾的话也可以选 WordPress，Gridea 等等。我比较熟悉R语言，可以直接调用`Blogdown`，就选了 Hugo。搭建的主要过程参考了 [统计之都](https://cosx.org/2018/01/build-blog-with-blogdown-hugo-netlify-github/ "统计之都")，步骤稍作修改。

初始设置先在 RStudio 中的进行如下设置：

`Tools -> Global Options -> Sweave -> Weave Rnw files using:knitr`

`Tools -> Global Options -> Sweave -> Typeset LaTex into PDF using:XeLaTeX`

RStudio 第一次运行 Blogdown 需要安装：

```
install.packages("blogdown")
blogdown::install_hugo()
```

## 在 Github 创建一个 Repo

Repo 名字任意，选择 `Add a README file`， 同时选择 `Add .gitignore with R`

回到 RStudio，选择新建一个 `File -> New Project -> Version Control -> Git`，输入 Repo 的网址，创建项目。

在 RStudio的 `Files` 中找到 `.gitignore` 文件，覆盖原文件，复制粘贴下面内容：

```
.Rproj.user
.Rhistory
.RData
.Ruserdata
public
static/figures
blogdown
```

## 创建本地网站

打开 RStudio：`File -> New Project -> New Directory -> Website using blogdown`，主题写 **athul/archie**,

![blogtest.PNG](https://dgbp4uvz49ycd.cloudfront.net/blogtest.PNG)


完成后在 `Console` 运行：

```
blogdown::serve_site()
```
中间有报错内容：

> ERROR 2021/09/17 11:33:37 instagram shortcode: Missing config value for services.instagram.accessToken. This can be set in config.toml, but it is recommended to configure this via the HUGO_SERVICES_INSTAGRAM_ACCESSTOKEN OS environment variable. If you are using a Client Access Token, remember that you must combine it with your App ID using a pipe symbol (<APPID>|<CLIENTTOKEN>) otherwise the request will fail.


在 `Files` 中找到 `config.toml`，加入一行 `ignoreErrors = ["error-missing-instagram-accesstoken"]` 保存，重新运行 `blogdown::serve_site()`。在 `config.toml` 中把 baseURL 改为自己的主页(https://nathanhuang.xyz/)。

在 `Tools` 菜单找到 `Addins`，选择 `New Post` 可以创作新文章。

> Filename 处会自动帮你填写为 Title 处的内容，Filename 和 Slug 还是建议使用字母，尤其是 Filename，如果博文里面不需要用到 R 语言的代码计算结果生成图表的话，Format 处就选择 Markdown 格式，这可以省去一些系统生成的步骤，ok，点击 Done，就会在 /content/post文件夹下面生成一个新文件了，content  文件夹下面的文件就是博客的文章了。---------[统计之都](https://cosx.org/2018/01/build-blog-with-blogdown-hugo-netlify-github/ "统计之都")



完成后通过以下步骤，将本地项目同步到 Github。
```
cd <本地项目目录>
git init
git add .
git commit -m "first comment"
git remote add origin https://github.com/<github帐号>/<仓库名称>
git remote -v
git pull origin master --allow-unrelated-histories
git push -u origin master
```

以后修改文件后，将本地项目同步到 Github 步骤如下：
```
git add .
git commit -m "first comment"
git remote add origin https://github.com/<github帐号>/<仓库名称>
git remote -v
git pull origin master --allow-unrelated-histories
git push -u origin master
```




## 通过 Amazon AWS S3 部署图床

平时在网站上放图片需要一个图床，我选了 AWS S3，教程来自(https://troyyang.com/2018/02/16/hosting-images-with-aws-s3/)。想省事的可以直接选微博图床或者 Bilibili 图床。

通过(https://console.aws.amazon.com/console/home)创建 `bucket`，名称任意（这里选择 nathanhuang），在 `Permission` 取消 `Block all public access`，上传图片在 `Action` 选择 `make public`。此时生成的节点是 http://nathanhuang.s3-website-ap-southeast-1.amazonaws.com/，国内访问不便。下面通过 CloudFront 自定义域名。


## CloudFront Distribution

在静态托管 `Static website hosting` 处选择托管，键入索引文件 index.html，错误文档 error.html。Origin Domain Name 中选择刚才所建的 S3 Bucket 域名。

部署完成需要等待一段时间，完成之后得到新的访问地址 dgbp4uvz49ycd.cloudfront.net，选择一张图片，加上 https://，再次访问 https://dgbp4uvz49ycd.cloudfront.net/blogtest.PNG，即可完成。



## 设置Netlify

注册登录 Netlify 后选择 `New site from Git`，再点击 Github，Brand 选择 Main，Build Command 填写 hugo，Publish directory 填写 public 点击 `Depoly Site`。

在 Namesilo 购买个性化域名 nathanhuang.xyz，回到 Netlify 选择 `Domain Management->Domain->Add Domain`，输入域名 nathanhuang.xyz。在 DNS Panel 处复制 Netlify DNS:
```
dns1.p07.nsone.net
dns2.p07.nsone.net
dns3.p07.nsone.net
dns4.p07.nsone.net
```

在 Namesilo 处选择 Unlock domain，并找到在 NameServers 并修改成刚才复制的 Netlify DNS。

回到 Netlify，选择 enable HTTPS。

部署好后的页面如果无法显示主题，可以把  `config.toml` 中的 baseURL 改回原先的 Netlify 域名，成功之后再改回自己的域名。


## Google 收录

进入 [Google Search Console](https://search.google.com/search-console/)，

![untitled.PNG](https://dgbp4uvz49ycd.cloudfront.net/Capture123123123123.PNG)


输入 Domain (nathanhuang.xyz) 点击 CONTINUE，复制 TXT 部分内容，回到 Netlify，点击 `Domain Settings -> Options -> Go to DNS Panel -> Add New Record`，Record Type 选择 TXT，Value 部分粘贴刚才复制的内容，保存设置，回到 Google Search Console 点击 Verify，成功后主页就可以被 Google 搜索到。  

到此网站已经搭建完毕，后续其实还有可以改进的地方，比如给网站加上 Google Analytics (国内的朋友可以搜一搜替代方案)，还可以给网站检测一下 SEO，针对性的修改网站。这里就不具体展开，我把一些相关链接放在附录里，可以作为参考。 



## 附录

自定义主题教程：(https://hugo-apero-docs.netlify.app/learn/)

在网站中加入Google Analytics: (http://cloudywithachanceofdevops.com/posts/2018/05/17/setting-up-google-analytics-on-hugo/)

SEO检测：(https://www.seobility.net/en/seocheck/)

7000 字告诉初学者 2022 Google SEO 怎么玩: (https://sspai.com/post/68905)

网站测速：(https://www.boce.com/)