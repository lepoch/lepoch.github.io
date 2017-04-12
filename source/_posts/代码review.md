---
title: 代码review
date: 2017-04-11T15:00:00.000Z
tags:
  - 工作
  - 日常
  - 新浪游戏
categories:
  - 工作
  - 日常
---

games 5.2

<!-- MORE -->

 # 庆禄

## 代码格式化问题

## 疑似BUG

### model/news_model.php

- dayRecommendList

  - 缓存key，缺少max_id

- searchGameListByKeyword

  - `count = 1000` ?
  - `if ($returnData['result'] == 'fail')`

### api/news.php

- imagineGame

  - `if (!isset($keyword))`
