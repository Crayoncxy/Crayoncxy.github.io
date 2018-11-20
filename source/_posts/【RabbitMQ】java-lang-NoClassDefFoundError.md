---
layout: posts
title: >-
 【RabbitMQ】java.lang.NoClassDefFoundError: org/springframework/util/backoff/BackOff
date: 2018-11-20 15:10:33
tags:
- RabbitMQ
categories: 问题解决
---

# 一、问题描述：

&ensp;&ensp;&ensp;&ensp;Spring整合RabbitMQ时，配置了消费者监听之后启动报错如下：

![png1]([RabbitMQ]java-lang-NoClassDefFoundError/png1.png)

# 二、解决方案：

&ensp;&ensp;&ensp;&ensp;目前项目中Spring版本为 3.2.8.RELEASE，spring-rabbit版本为1.7.5.RELEASE，经分析是因为版本不一致的原因导致。有两种方案要么对Spring进行升级，要么对spring-rabbit进行降级。因项目整体依赖Spring，故最后选择降级spring-rabbit到版本1.3.5.RELEASE，随后问题解决。