---
layout: post
title: 量化交易
permalink: quant
---
这里主要研究数字货币的量化交易。

做量化分析、量化投资，任何算法、模型、短期收益的高低都无所谓，关键是稳定。只要稳定了，使用复利公式，哪怕是1%的增长，其增长速度都是几何裂变模式的。

# Technical Analysis Indicators —— 技术分析指标
量化的原理就是根据历史和现在的数据，根据相应公式算出技术指标，来决定当前如何交易。

## SMA —— Simple Moving Average

## EMA —— Exponential Moving Average
加了权重的 SMA，日期越近，权重越大。

# 交易策略
根据数据来决定如何交易的算法。

# Backtesting —— 回归测试
用历史数据测试当前的策略的收益。

# 软件

## [Gekko](https://github.com/askmike/gekko)
基于 Node 的量化交易平台。

## [rqalpha](http://rqalpha.io/)

## [vn.py](http://www.vnpy.org/) - 基于python的开源交易平台开发框架

## [pytrader](https://github.com/owocki/pytrader)
用了机器学习的交易机器人。

# 套利
套利算是策率简单但有效的量化交易。

## [Blackbird](https://github.com/butor/blackbird)
C++ 写的套利软件，策略是不用搬移货币，因为货币价格总是趋于一致的，所以低价的买高价的卖。

## [bitcoin-arbitrage](https://github.com/maxme/bitcoin-arbitrage)

## [rbtc_arbitrage](https://github.com/hstove/rbtc_arbitrage)

# Python
Python 是量化的标准语言了，有众多功能强大的第三方库，也被华尔街官方指定。

```python
# py量化：三大件:pd,ts,zp,py量化分析的三个核心模块库，
import pandas as pd
# TuShare是一个免费、开源的python财经数据接口包
import tushare as ts
import zipline as zp
import talib as ta
import pyalgotrade as pat


# py数据分析：三大件,py数据分析，科学计算三个核心模块库，
import numpy as np
import scipy as sp
import matplotlib as mpl #mpl
# 画图库
import matplotlib.pyplot as plt
```

## [ANACONDA](https://www.continuum.io/downloads)
ANACONDA 是一个集成的数据分析平台。它已经包含了各种包。

使用时有两个接口：

- conda
- anaconda-navigator  conda 的图形界面



# 参考
- http://www.huangzhong.ca/zh/bitcoin-arbitrage-trading-robots-open-source/
- https://www.zhihu.com/question/52589498
