---
title: php代码审核之路
date: 2017-03-20 15:00:00
tags:
 - 新浪游戏
 - php

categories:
 - php


---


上周审核了几个同事的代码，发现代码规范用处太小。使用工具进行代码的审核势在必行。

<!-- MORE -->
# 寻觅

## Rietveld
谷歌出品工具，很犀利，免费开源。PYTHON + Django， 使用Google App Engine。

## Gerrit
免费开源，网页管理界面，GIT管理。

## Crucible
Atlassian公司出品，收费。暂时不考虑。

# 初见
LINUX 虚拟机
CentOS7

## Rietveld

### 搭建
1. 首先就得科学上网，见另一篇 《[Linux科学上网](/Linux科学上网/)》
2. CentOS 自带的python版本低，需要升级，见另一篇《[升级python](/升级python/)》
