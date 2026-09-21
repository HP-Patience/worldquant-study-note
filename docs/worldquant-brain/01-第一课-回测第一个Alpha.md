# WorldQuant BRAIN 学习笔记

> 目标：从零开始学习 Alpha Research，先跑通一次可解释的回测，再逐步学习改进、提交和积累 Challenge 积分。
>
> 本笔记依据 WorldQuant BRAIN 官方平台页面整理，内容以当前账户可见页面为准。回测结果不代表真实交易收益，也不构成投资建议。

## 学习方式

每次只推进一个小目标：

1. 先提出一个可以验证的市场假设。
2. 把假设写成 BRAIN 表达式。
3. 固定回测设置，只改变一个变量。
4. 查看结果，记录失败原因和下一步实验。
5. 暂不盲目批量提交，不用随机调参代替研究。

核心流程：

```text
研究假设 → Alpha 表达式 → 股票打分 → 模拟持仓 → 历史 PnL → 检查提交
```

## 第一课：回测第一个 Alpha

### 1. 先理解 Alpha 是什么

Alpha 可以先理解为：

> 每天根据已有数据，为一批股票打分的一条规则。

BRAIN 根据分数模拟多头和空头仓位，再用历史数据计算盈亏。它不是输入一只股票后直接预测明天涨跌，也不会替代真实下单。

### 2. 第一个研究假设

假设：

> 过去几天跌得较多的股票，接下来可能出现短期反弹。

这是一个短期反转假设，不是必然规律。

官方教学表达式：

```text
-ts_delta(close, 5)
```

拆解：

| 部分 | 含义 |
|---|---|
| `close` | 收盘价字段 |
| `ts_delta(close, 5)` | 当前值减去 5 个交易日前的值 |
| `-` | 取相反数，让近期下跌的股票得到更高分 |

例如：

| 股票 | 5 个交易日前 | 当前 | 变化 | 取负后的分数 |
|---|---:|---:|---:|---:|
| A | 100 | 90 | -10 | +10 |
| B | 100 | 110 | +10 | -10 |
| C | 100 | 100 | 0 | 0 |

表达式输出的是原始分数，不是最终持仓比例。BRAIN 还会进行中性化、缩放、权重分配等处理。

### 3. 在 BRAIN 上操作

打开：<https://platform.worldquantbrain.com/simulate>

点击右上角 `Settings`，先使用下面的设置：

| 设置 | 值 |
|---|---|
| Language | Fast Expression |
| Instrument Type | Equity |
| Region | USA |
| Universe | TOP3000 |
| Delay | 1 |
| Neutralization | Market |
| Decay | 0 |
| Truncation | 0.08 |
| Pasteurization | On |
| Unit Handling | Verify |
| NaN Handling | Off |

点击 `Apply`，回到 `CODE` 输入：

```text
-ts_delta(close, 5)
```

然后点击 `Simulate`。

本课暂时不要点击 `Submit Alpha`，先看懂结果。

### 4. 两个容易混淆的参数

#### 表达式中的 `5`

表示向前看 5 个交易日，计算价格变化。

#### 回测设置中的 `Delay = 1`

表示建立仓位时使用前一个交易日及以前可获得的信息。它不是“持有 1 天”，表达式中的 `5` 也不是“持有 5 天”。

### 5. 结果先看什么

回测完成后，先观察：

1. 是否成功完成模拟。
2. 累计 PnL 曲线是否大致持续，还是只靠少数时间段上涨。
3. Sharpe、Fitness、Turnover、Drawdown 数值。
4. 是否有权重、相关性、Fitness 或子股票池测试失败。

官方把这个表达式作为需要改进的教学示例，因此结果不好不代表操作错误。

## 第二步：只做两个对照实验

完成第一条回测后，只改变一个变量，分别测试：

```text
-ts_delta(close, 10)
```

```text
-ts_delta(close, 20)
```

其他回测设置保持不变。记录：

```text
实验编号：
研究假设：
表达式：
完整回测设置：
本次只改变了什么：
Sharpe：
Fitness：
Turnover：
Drawdown：
未通过的测试：
结论：
下一步要验证的问题：
```

不要继续枚举大量窗口寻找历史最好看的数字。本练习的目标是理解变量如何改变结果，而不是过拟合历史。

## 提交前先知道的主要门槛

官方提交说明中，普通 Delay-1 Alpha 的主要检查包括：

| 检查 | 文档中的主要要求 |
|---|---|
| Sharpe | Delay-1 通常需大于 1.25 |
| Fitness | 通常需大于 1 |
| Turnover | 大于 1%，小于 70% |
| 权重 | 单只股票最大权重低于 10%，并检查整体分布 |
| 子股票池 | 在更小、更具流动性的股票池中仍需有足够表现 |
| 自相关性 | 通常要求与已提交 Alpha 的 PnL 相关性低于 0.7，或有足够明显的 Sharpe 改进 |

这些是提交测试，不是盈利保证，也不是成为顾问的保证。最终以当前 Alpha 的 `Check Submission` 结果为准。

如果高相关，优先研究新的机制或新的数据，而不是给旧表达式添加无意义的小项。官方文档也建议，低相关的新想法通常比小幅改进高相关 Alpha 更有价值。

## 顾问项目路线

官方当前页面给出的路线是：

```text
学习与模拟 → 提交合格 Alpha → Challenge 积分 → 满足地区和其他条件 → 背景调查与咨询协议 → 研究顾问
```

当前资料中的关键点：

- 顾问项目要求至少 10,000 分。
- Challenge 每天积分有上限，当前文档写明最高 2,000 分。
- 达到分数不保证获得顾问机会或报酬。
- 还需要通过背景调查、签署咨询协议及其他适用要求。
- 只能使用自己的账户和原创 Alpha，不要共享账号、复制他人研究或人为加入噪声。

## 建议的第一周安排

| 天数 | 任务 |
|---|---|
| 第 1 天 | 阅读 Starter Pack，运行 `-ts_delta(close, 5)` |
| 第 2 天 | 理解 Delay、Decay、Truncation、Neutralization |
| 第 3 天 | 运行 5 天和 10 天窗口对照实验 |
| 第 4 天 | 学习 Sharpe、Fitness、Turnover、Drawdown |
| 第 5 天 | 学习提交测试，查看为什么某个 Alpha 不能提交 |
| 第 6 天 | 阅读新手 Alpha 示例，选一个不同机制的想法 |
| 第 7 天 | 复盘实验记录，不急着批量提交 |

每天约 1—2 小时即可。第一周的验收标准不是赚多少钱，而是能够解释：

- 这条表达式在押注什么；
- 每个回测设置改变了什么；
- 结果为什么好或坏；
- 下一步实验只验证哪个问题。

## 官方资料

- [BRAIN 模拟页面](https://platform.worldquantbrain.com/simulate)
- [Starter Pack](https://platform.worldquantbrain.com/learn/documentation/discover-brain/read-first-starter-pack)
- [10 Steps to Start on BRAIN](https://platform.worldquantbrain.com/learn/documentation/discover-brain/10-steps-start-brain-platform)
- [BRAIN 如何运作](https://platform.worldquantbrain.com/learn/documentation/create-alphas/how-brain-platform-works)
- [如何选择回测设置](https://platform.worldquantbrain.com/learn/documentation/create-alphas/simulation-settings)
- [回测你的第一个 Alpha](https://platform.worldquantbrain.com/learn/documentation/create-alphas/running-your-first-alpha)
- [提交 Alpha 前需通过这些测试](https://platform.worldquantbrain.com/learn/documentation/interpret-results/alpha-submission)
- [WorldQuant Challenge](https://platform.worldquantbrain.com/learn/documentation/discover-brain/challenge-help)
- [Events](https://platform.worldquantbrain.com/events/)
