---
layout: posts
title: 【Apache Ant】Unsupported major.minor version 52.0
date: 2018-11-20 14:35:23
tags:
- Apache Ant
categories: 问题解决
---

# 一、问题描述：

&ensp;&ensp;&ensp;&ensp;在windows、jdk版本为1.7的环境下安装了Apache Ant最新版 apache-ant-1.10.3，配置完环境变量执行ant命令时报错Unsupported major.minor version 52.0

![png1]([Apache-Ant]Unsupported-major-minor-version-52-0/png1.png)

# 二、解决方案：

&ensp;&ensp;&ensp;&ensp; 经查，该问题是由于class的版本与jdk的版本不对应导致，版本对应关系如下：

    JDK1.8 = 52
    JDK1.7 = 51
    JDK1.6 = 50
    JDK1.5 = 49
    JDK1.4 = 48
    JDK1.3 = 47
    JDK1.2 = 46
    JDK1.1 = 45
&ensp;&ensp;&ensp;&ensp;所以解决办法是将JDK版本换为1.8或者下载低版本的Apache Ant ，这里我重新下载了 apache-ant-1.9.11 配置完环境变量运行成功结果如下：

![png2]([Apache-Ant]Unsupported-major-minor-version-52-0/png2.png)

&ensp;&ensp;&ensp;&ensp;注：版本不匹配的问题不仅出现在Ant中，其它软件也可能会出现问题。找到对应的版本即可。