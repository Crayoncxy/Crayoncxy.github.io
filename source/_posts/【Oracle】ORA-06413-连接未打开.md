---
layout: posts
title: 【Oracle】ORA-06413 连接未打开
date: 2018-11-20 14:20:43
tags:
- Oracle 
- Toad
categories: 问题解决
---

# 问题描述：

&ensp;&ensp;&ensp;&ensp;使用toad连接oracle报错“ORA-06413 连接未打开”

# 解决方案：

&ensp;&ensp;&ensp;&ensp;toad安装路径下存在诸如‘（’等特殊字符，比如我的路径是Program Files（x86），修改安装路径为D：\tools等不带特殊字符的路径即可