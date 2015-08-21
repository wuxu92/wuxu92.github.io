---
layout: post
title: 使用sed处理文本
description: linux的文本处理工具除了awk之外，还有一个利器，那就是sed。sed用于文本的替换，也是以行为单位，使用正则表达式进行匹配。
category: blog
tags: linux sed 
published: true
---

{{page.description}}

参考： [http://coolshell.cn/articles/9104.html](http://coolshell.cn/articles/9104.html "http://coolshell.cn/articles/9104.html")

## 基础 ##
sed的命令模式是这样的：

```bash
sed -i 'sed-command' file-to-process
```
其中，-i参数是很常用的，如果不带-i参数处理的结果会在终端打印出来，带上-i参数后会将处理结果替换输入文件的内容。

中间引号内的是sed的命令内容，常用单引号，但是要注意在单引号里面，反斜杠(\)转义将不能起作用。命令常用模式为 `s/old-pattern/new-str/g`
最后的是要处理的文本的文件名。

由于sed主要依赖正则匹配实现功能，所以先熟悉一下基础的正则规则：

- ^ 表示一行的开头。如：`/^#/` 以#开头的匹配。
- $ 表示一行的结尾。如：`/}$/` 以}结尾的匹配。
- `\<` 表示词首。 如 \<abc 表示以 abc 为首的詞。
- `\>` 表示词尾。 如`abc\>` 表示以 abc 結尾的詞。
- . 表示任何单个字符。
- `*` 表示某个字符出现了0次或多次。
- `[ ] `字符集合。 如：`[abc]`表示匹配a或b或c，还有`[a-zA-Z]`表示匹配所有的26个字符。如果其中有^表示反，如`[^a]`表示非a的字符

练习：下面的命令可以去掉html文件中的标签，只留下文本：

```bash
sed -i "s/<[^>]*>//g" index.html
```
其中，命令开头的s代表替换，`[^>]*`表示一个以上的非>字符，替换为空。

上面的命令如果是:

```
sed -i 's/<.*>//g' index.html
```
看起来也好像能工作，但是， <.*>会匹配最长的尖括号内容，即从文本的第一个< 到最后一个>，这样达不到我们的效果。

## 进阶 ##

### 指定替换 ###
**替换指定行的内容**
有时候我们仅需要对某些行进行替换，可以在命令中指定行：

```bash
# 只对第3行进行替换
sed -i '3s/me/you/g' file-name

#只对第3-6行替换
sed -i '3,6s/me/you/g' file-name
```

**指定每一行替换的个数**

```bash
# 替换每一行的第一个匹配
sed -i 's/me/you/1' file-name

#替换每一行的第2个匹配
sed -i 's/me/you/2' file-name

#替换第5个以后的所有匹配
sed -i 's/me/you/3g' file-name
```

### 多个匹配 ###
如果要对一行进行两个匹配，可以在命令字符串中使用;分割多个匹配项：

```
sed '1,3s/me/you/g; 3,$s/you/me/g' file-name

# 下面的命令等效
sed -e '1,3s/me/you/g' -e '3,$s/you/me/g' file-name
```
这样将1-3行的me替换为you，将3到最后一行的you替换为me。

### 圆括号匹配 ###
类似于正则表达式中的分组，在s中使用的括号，可以在替换串中使用\1,\2指代


## 待续 ##