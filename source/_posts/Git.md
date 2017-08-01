---
title: Git
date: 2017-05-22T16:55:00.000Z
tags:
  - Git
  - 笔记
---

Git

<!-- MORE -->
# 彻底删除某文件及历史
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch "doc/base.sql" ' --prune-empty --tag-name-filter cat -- --all
git push origin --force --all
git push origin --force --tags
git for-each-ref --format='delete %(refname)' refs/original | git update-ref --stdin
git reflog expire --expire=now --all
git gc --prune=now
git count-objects -v
