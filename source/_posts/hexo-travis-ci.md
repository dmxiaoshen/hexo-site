---
title: Hexo博客托管Github Pages并结合Travis CI自动部署
date: 2016/11/19
toc: true
---

用hexo博客写作已经是很方便了，部署也只要deploy到github上即可，但是还是受限于需要node.js环境，而且还不能通过浏览器写作。
那么问题来了，有没有这种操作，我只要一提交文章代码，就自动构建出静态网站并部署上去呢？答案当然是有的，那就是接下来要介绍的Travis CI。
<!--more-->
## 登陆Travis CI官网

假设已经有一个hexo博客并托管在GitHub Pages，此时应该有一个Repository叫`yourname.github.io`。

登陆[Travis CI](https://travis-ci.org/)官网，并且用github账号登陆，此时会自动关联所有github上的仓库，
页面左边My Repositories右边有个'+'号,点击选择`yourname.github.io`项目，这里说明一点，该项目应该有2个分支，master
分支用来部署博客页面，然后source（也可以叫别的名字）是博客源码（包括文章，主题等)。

![source分支](https://ws1.sinaimg.cn/large/683a46dcgy1flp25fcb07j20s30cjwf9.jpg)

注意: package.json文件很重要，里面包含了项目依赖
添加完项目后，选择如下配置:

![配置项目](https://ws1.sinaimg.cn/large/683a46dcgy1flp2abwc46j20pw0i7t9h.jpg)

## 登陆github网站

主要是为了设置**personal access token**,画面如下:

![access token](https://ws1.sinaimg.cn/large/683a46dcgy1flp2f2ydgkj20rp0xgwh4.jpg)

设置完成会获得一串token，复制下来等会要配置到Travis CI网站上。

## 配置Access Token

回到Travis CI网站，把刚才的access token值配置上去，就在刚才项目配置页面,新增一个环境变量叫GH_TOKEN,

![配置token](https://ws1.sinaimg.cn/large/683a46dcgy1flp2hxb59sj20qg05z74c.jpg)

## 编写.travis.yml文件

Travis CI自动构建想要成功，必须正确配置.travis.yml文件，该文件放置位置为source分支下根目录。

整理如下:

```yml
language: node_js  #设置语言

node_js: stable  #设置相应的版本

cache:
    apt: true
    directories:
        - node_modules # 缓存不经常更改的内容

before_install:
    - export TZ='Asia/Shanghai' # 更改时区

install:
  - npm install  #安装hexo及插件

script:
  - hexo clean  #清除
  - hexo g  #生成

after_script:
  - git clone https://${GH_REF} .deploy_git  # GH_REF是最下面配置的仓库地址
  - cd .deploy_git
  - git checkout master
  - cd ../
  - mv .deploy_git/.git/ ./public/   # 这一步之前的操作是为了保留master分支的提交记录，不然每次git init的话只有1条commit
  - cd ./public
  - git config user.name "name"  #修改name
  - git config user.email "email"  #修改email
  - git add .
  - git commit -m "Travis CI Auto Builder at `date +"%Y-%m-%d %H:%M"`"  # 提交记录包含时间 跟上面更改时区配合
  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master  #GH_TOKEN是在Travis中配置token的名称

branches:
  only:
    - source  #只监测source分支，source是我的分支的名称，可根据自己情况设置
env:
 global:
   - GH_REF: github.com/yourname/yourname.github.io.git  #设置GH_REF，注意更改yourname
```
