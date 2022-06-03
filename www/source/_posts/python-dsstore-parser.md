---
title: DS_Store文件解析工具
date: 2022-06-03 02:57:20
update: 2022-06-03 02:57:20
tags: [工具, 信息安全]
---



在Mac系统的文件夹下常常会出现一个名为.DS_Store的隐藏文件。

本文介绍了.DS_Store的作用以及该文件泄露可能导致的潜在风险，最后介绍如何使用Python-dsstore库解析.DS_Store文件。

<!-- more -->

## 什么是DS_Store

桌面服务存储（Desktop Services Store，DS_Store），是一种由macOS系统自动创建的隐藏文件，存在于每一个用「访达」打开过的文件夹下面。

虽然不能在「访达」中直接看到它，但是通过「终端」App，可以输入`ls -la`命令列出。同时，通过`file`命令，可以显示出其文件类型，即“Desktop Services Store”。

## DS_Store存储什么信息

DS_Store 文件的主要作用，是存储当前文件夹在桌面显示相关方面的一些自定义属性，包括文件图标的位置、文件夹上次打开时窗口的大小、展现形式和位置等。 这有助于保留为特定文件夹配置的设置，例如，将桌面文件夹设置为查看按名称排序的图标，同时将下载文件夹配置为将文件显示为列表并按日期排序，最近修改的先显示。

## 由DS_Store引起的安全问题

DS_Store中可以解析出文件夹下的目录信息。由于它本身是一个隐藏文件，所以常常会被忽视，如在开发过程中有时会忘记将.DS_Strore加入.gitignore文件中，导致.DS_Strore文件被git跟踪。若将DS_Store带到生产环境中，可能导致安全隐患。具体来说，对于一个网站，只要攻击者获得了DS_Store文件，他就可能通过解析工具解析出与网站相关的目录与文件名。攻击者可通过解析这一文件，可能会发现数据库备份文件、配置文件以及一些缓存文件，甚至是密钥等。

此外，对于希望在网络中隐藏真实身份的用户来说，在某些情况下文件名包可能会我们的姓名、ID等个人信息。DS_Store文件的泄漏可能导致别有用心的人知晓我们的真实身份。

## 用Python-dsstore库解析DS_Store

{% ghcard gehaxelt/Python-dsstore %}

该库的作者在文章 [“Parsing the .DS_Store file format”](https://0day.work/parsing-the-ds_store-file-format/) 中更加详细地介绍了DS_Store文件的相关信息与该库实现的技术细节。

该库的使用十分简单。

首先，把代码`clone`到本地。

```bash
git clone https://github.com/gehaxelt/Python-dsstore.git
```

在Python环境下运行`main.py`即可，参数是DS_Store文件的路径。

```python
python main.py samples/.DS_Store.ctf
```



