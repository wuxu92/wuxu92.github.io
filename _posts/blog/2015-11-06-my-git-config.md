---
layout: post
title: 我的git配置
description: gitconfig的配置非常重要，有时候换到一台新机器由于没有自己配置搞得很不顺手，干脆把gitconfig摘出来，方便以后配置。
category: post
tags: git
published: true
lastUpdate: 2015-11-06
---
![](/images/post/git.jpg)

{{ page.description }}
主要是命令别名的配置。

```bash
[user]
        name = wuxu92
        email = wuxu92@163.com
[alias]
        lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%Creset [%an]' --abbrev-commit --date=relative
        ls = ls --show-control-chars --color=auto
        go = "! bash -c \"git pull && git add .; if [ \\\"\\\" == \\\"\\\" ]; then git commit -a; else git commit -am \\\\¡\\\\¡; fi; git push origin master:yy
our-id;\""
        ci= commit
        co = checkout
        br = branch
        st = status
        list = log --pretty=format:\"%h %ad | %s%d [%an]\" --graph --date=short
        undo = reset --hard
        llog = log --date=local
        changes = diff --name-status -r
        diffst = diff --stat -r
        conf    = diff --name-only --diff-filter=U
[http]
# proxy = 127.0.0.1:1080

[https]
# proxy = 127.0.0.1:1080

[push]
        default = simple
```
