---
title: windows下搭建hexo博客
date: 2022-03-18 22:53:55
tags: hexo
toc: true
categories: hexo使用
---

[TOC]

<!--more-->

## 1.安装nodejs

略

## 2.切换淘宝源(有梯子则不切换)： 

npm install -g cnpm --registry =https://registry.npm.taobao.org

##  3.安装hexo:  

cnpm install -g hexo-cli

##  4.使用hexo搭建。

 - 新建blog文件夹

 - 初始化sudo hexo init或 hexo init   

   - windows管理员权限出现问题。hexo : 无法加载文件 C:\Users\xxx\AppData\Roaming\npm\hexo.ps1，因为在此系统上禁止运行脚本。

   - 解决方案
     在默认情况下，我们是无法执行[powershell](https://so.csdn.net/so/search?q=powershell&spm=1001.2101.3001.7020)脚本的， 需要更改执行策略。win10下更改执行策略：
     1.打开设置

     2.搜索power

     选择选项：允许本地powershell在不签名的情况下运行
     3.勾选，点击应用就可以了

## 5.启动

hexo s

##  6.github新建仓库。 

仓库名必须是 用户名.github.io 

- 例如junjieliao593新建的仓库名为：junjieliao593.github.io.git

##  7.本地下载git插件

- cnpm install --save hexo-deployer-git

##  8.设置blog目录下__cofig.yml配置。

- Deloyment配置git仓库地址及分支 
  - deploy:
        type: git
        repo: https://github.com/junjieliao593/junjieliao593.github.io
        branch: master

##  9.部署发布

hexo d

##  10.访问

junjieliao593.github.io

##  11.主题

https://github.com/litten/hexo-theme-yilia
- git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
- 修改blog目录下_config.yml配置：   theme:  yilia
- 清理hexo clean
- 重构hexo g
- 预览hexo s
- 推到远程仓库hexo d

- 其他主题https://github.com/theme-next/hexo-theme-next

##  12.编辑 

文章md文件存放在 blog\source\_posts目录下，用其他编辑器操作即可。

## 13. 后续可参考文档

官方文档： https://hexo.io/zh-cn/docs/
完成操作文档(包含多终端工作指南)： https://blog.csdn.net/sinat_37781304/article/details/82729029


## 补充
推荐一个很好看的主题，也是当前本人博客所用的主题，有比较详细的中文文档，https://mengd.js.org/ 

