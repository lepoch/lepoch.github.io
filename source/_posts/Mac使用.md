---
title: Mac使用
date: 2017-05-11T21:14:00.000Z
tags:
  - Mac
categories:
  - Mac
---

Mac使用


<!-- MORE -->

# ssh登陆  
不用每次输入完整命令、密码
## 使用expect  
使用iterm2终端，ssh + expect + 别名

** 此种方式登陆会出现lrzsz没效果的情况！！ **
1. yum install expect -y

- ~/expect/login.sh 
```
#! /usr/bin/expect
set timeout 30
eval spawn [lrange $argv 1 end]
expect {
        "(yes/no)?"
        {send "yes\n";exp_continue}
        "password:"
        {send "[lindex $argv 0]\n"}
}
interact
```

- ~/.bash_profile  
expect ~/expect/login.sh 密码 ssh 完整命令
```
alias lssh-vultr='expect ~/expect/login.sh 密码 ssh root@45.77.12.xxx'
```

- source ~/.bash_profile

使用 lssh-vultr 直接登陆

## 使用 sshpass

## 使用.ssh (推荐)


# 终端文字颜色
1. ~/.bash_profile
```
alias ls='ls -G'
alias ll='ls -l'
alias grep='grep --color'
```
- source ~/.bash_profile
