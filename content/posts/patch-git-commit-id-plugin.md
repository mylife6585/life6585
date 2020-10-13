---
title: "Patch Git Commit Id Plugin"
date: 2020-10-13T10:24:11+08:00
description: "批量安装maven git-commit-id插件"
tags: ["maven","shell"]
categories: ["devops"]
keywords: ["maven","shell"]
draft: false
isCJKLanguage: true
---

## 问题

想给所有的service增加maven git-commit-id-plugin,实现版本与代码关联

微服务代码使用了git的submodule批量管理

## 解决

pom.xml 需要增加以下依赖,保存为/tmp/add.xml

```
            <plugin>
                <groupId>pl.project13.maven</groupId>
                <artifactId>git-commit-id-plugin</artifactId>
                <version>4.0.0</version>
                <executions>
                    <execution>
                        <id>get-the-git-infos</id>
                        <phase>initialize</phase>
                        <goals>
                            <goal>revision</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <dotGitDirectory>${project.basedir}/.git</dotGitDirectory>
                    <verbose>false</verbose>
                    <dateFormat>yyyy-MM-dd HH:mm:ss</dateFormat>
                    <prefix>git</prefix>
                    <generateGitPropertiesFile>true</generateGitPropertiesFile>
                    <generateGitPropertiesFilename>${project.build.outputDirectory}/git.properties</generateGitPropertiesFilename>
                    <format>json</format>
                    <gitDescribe>
                        <skip>false</skip>
                        <always>false</always>
                        <dirty>-dirty</dirty>
                    </gitDescribe>
                </configuration>
            </plugin>           
```


批量修改

创建 ~/.netrc 避免手动输入用户名、密码
```
machine git.souban.io
login username
password password
```


```
git clone https://git.souban.io/Server/CREAMS/creams-microservices.git
cd creams-microservices
git checkout release
git submodule update --init --recursive
git submodule foreach 'git checkout release'
git submodule foreach 'git pull'

git status
git submodule status

find ./*-service -name "pom.xml" | xargs sed -i "/<plugins>/r /tmp/add.xml"

git submodule foreach 'git commit -am "feat(maven): add git-commit-id-plugin" || :'
git submodule foreach 'git push || :'
git commit -am 'update'
git push --recurse-submodules=check

```
