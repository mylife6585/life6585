---
title: "Hugo Github Blog Deploy"
description: "使用hugo、github pages搭建个人blog"
tags: ["hugo","github","travis"]
categories: ["application"]
keywords: ["hugo","github","travis"]    
date: 2020-06-16T10:56:19+08:00
draft: false
isCJKLanguage: true
---


## 方案

github搭建个人博客

* 常用的生成器有hugo、hexo、jekyll,google对比了一下，hugo简单又快，就采用了hugo

* github pages有几种类型: user pages、project pages等

   user pages: repository name必须是{username}.github.com, 只存放生成的静态网站，需要有另外一个repository存放代码，不能指定pages的分支或者路径

   project pages: repository name不是{username}.github.com就可以，代码和生成的静态网站可以存放在一个repository,生成的静态网站可以是一个目录或者一个分支(常用gh-page)

* deploy

  脚本: huogo有样例，通过手动触发部署

  ci: travis ci等，push 代码的时候，自动部署

* result

  先用project pages的分支方式部署成功，觉得有点复杂，换成了user pages,最终方案hugo+github user pages+travis ci


## hugo

### mac系统安装
```bash
brew install hugo
hugo version
```

### create site

```bash
hugo new site  life6585
cd life6585
git init
# 可以从https://themes.gohugo.io/选择主题
git submodule add --depth 1 https://github.com/reuixiy/hugo-theme-meme.git themes/meme
rm config.toml && cp themes/meme/config-examples/en/config.toml config.toml
hugo new "posts/hello-world.md"
hugo new "about/_index.md"
hugo server -D
```

通过http://localhost:1313预览


## github

github 创建 repository: life6585,用做代码仓库 ，事先加好github ssh-key

push to github

```bash
git add .
git commit -m "first commit"
git remote add origin git@github.com:mylife6585/life6585.git
git config  user.name "life6585"
git config  user.email "life85@126.com"
git push --set-upstream origin master
```

github 创建 repo: mylife6585.github.com,用做网站仓库

```bash
echo "public" >> .gitignore
git submodule add  git@github.com:mylife6585/mylife6585.github.com.git public
```


## deploy

### 1. script

可以通过脚本手动推送网站到github

增加deploy.sh

```bash
#!/bin/sh

# If a command fails then the deploy stops
set -e

printf "\033[0;32mDeploying updates to GitHub...\033[0m\n"

# Build the project.
hugo # if using a theme, replace with `hugo -t <YOURTHEME>`

# Go To Public folder
cd public

# Add changes to git.
git add .

# Commit changes.
msg="rebuilding site $(date)"
if [ -n "$*" ]; then
	msg="$*"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master

```

### 2. travis ci

也可以通过travis ci自动部署

github增加personal access tokens , 通过github账号登陆[travis ci](https://travis-ci.org) , 管理repository，并添加$GITHUB_TOKEN 环境变量

代码仓库life6585增加.traivs.yml

```yaml
language: go

git:
  depth: 1

install:
  - wget https://github.com/gohugoio/hugo/releases/download/v0.72.0/hugo_0.72.0_Linux-64bit.deb
  - sudo dpkg -i hugo*.deb

script:
  - hugo

deploy:
  provider: pages
  skip_cleanup: true
  local_dir: public
  github_token: $GITHUB_TOKEN
  on:
    branch: master
  repo: mylife6585/mylife6585.github.com
  target_branch: master
  email: deploy@travis-ci.org
  name: deployment-bot
```

README.md 增加traivs ci build status[![Build Status](https://travis-ci.org/mylife6585/life6585.svg?branch=master)](https://travis-ci.org/mylife6585/life6585)


## reference

https://zyfdegh.github.io/post/201705-how-i-setup-hugo/

https://themes.gohugo.io/hugo-theme-meme/

https://gohugo.io/hosting-and-deployment/hosting-on-github/

https://mogeko.me/2018/018/

https://gohugo.io/hosting-and-deployment/hosting-on-github/

https://zhuanlan.zhihu.com/p/37752930

https://dev.to/zaracooper/create-your-developer-portfolio-using-hugo-and-github-pages-35en

https://medium.com/swlh/hosting-a-hugo-blog-on-github-pages-with-travis-ci-e74a1d686f10

https://mogeko.me/2018/028/

https://zyfdegh.github.io/post/201705-how-i-setup-hugo/

http://www.ruanyifeng.com/blog/2017/12/travis_ci_tutorial.html


## problem

1. github project pages和user page的区别

    user pages的repository name 特殊，为{username}.github.com,并且github pages不能设置路径或者branch

2. hugo文章不显示

    去掉文章中的draft


