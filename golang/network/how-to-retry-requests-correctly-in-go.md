# 在 Go 中如何正确重试请求
我们平时在开发中肯定避不开的一个问题是如何在不可靠的网络服务中实现可靠的网络通信，其中 http 请求重试是经常用的技术。但是 Go 标准库 net/http 实际上是没有重试这个功能的，
所以下文主要讲解如何在 Go 中实现请求重试。

## 概述

## 参考文献
- https://mp.weixin.qq.com/s/GKggVplX_ZzoXJDuWf5ctA