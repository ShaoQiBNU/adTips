# 广告系统基础及综述

## 基本概念和名词

| 名词 | 含义 | 计算方式 |
| :----:| :----: | :----: |
| send | 下发数 |  |
| show | 曝光数 |  |
| click | 点击数 |  |
| convert/action | 转化数 |  |
| ctr | 点击率 | $$\frac{click}{show}$$ |
| cvr | 转化率 | $$\frac{convert}{click}$$ |
| cost | 广告主消耗 or 广告主付出的成本，也就是广告平台赚的钱 |  |
| rank_bid | 出价 | 主要有 $ cpc\_bid, cpm\_bid, cpa\_bid $ |
| $cpc\_bid$ | 广告主为每次点击付出的成本 |  $$\frac{cost}{click}$$ |
| $cpm\_bid$ | 广告主为每**1000**次曝光付出的成本 |  $$\frac{cost}{show * 1000}$$ |
| $cpa\_bid$ | 广告主为每次转化付出的成本 |  $$\frac{cost}{convert/action}$$ |
| $eCPM$ | 预期1000次曝光的cost |  $$eCPM = rank_bid * ctr * cvr * 1000$$ |
| auction_price | 计费 | 主要有 GFP(按照排序第一的出价计费)、GSP(按照排序第二的出价计费) |

由表格可以得出：

$$ cpm\_bid = cpa\_bid * ctr * cvr * 1000 $$

$$ cpm\_bid = cpc\_bid * ctr * 1000 $$

## 广告类型

### 品牌广告
> 品牌广告追求品牌的曝光和触达，出价方式分为两种：
- CPT(cost per time)：包段时间内整个位置，溢价比较高
- GD(guaranteed delivery)：购买特定数量，较高的溢价

### 效果广告
> 效果广告也叫流量竞价广告，以 $eCPM$ 作为排序依据，主要有以下出价类型 $CPC, CPM, CPA, oCPC, oCPM$

## 广告竞价和计费

由于不同出价类型的广告出价方式不同，因此竞价时会将各类型广告统一计算 $eCPM$，然后按照 $eCPM$ 从高到低进行排序，然后按照排序结果进行计费。各出价类型的广告竞价和计费方式如下：

| 出价类型 | 全名 | 说明 | $eCPM$ 的计算方式 | 平台计费方式 | 广告类型 | ratio |
| :----:| :----: | :----: | :----: | :----: | :----: | :----: |
| CPT | Cost per time | 包段时间内整个位置 | 不用算 | 合同 | 品牌 | 1 |
| GD | guaranteed delivery | 购买特定数量 | rank_bid | 合同 | 品牌 | 1 |
| CPC | Cost per click | 按照广告点击出价，按照广告点击计费 | CPC_bid * ctr * 1000 | auction_price / ctr / 1000 = CPC_bid| 效果 | 1000 * ctr |
| CPM | Cost per mile impression | 按照广告展示1000次出价，按照广告展示计费 | CPM_bid | auction_price / 1000 = CPM_bid | 效果 | 1000 |
| CPA | Cost per action | 按照广告转化出价，按照广告转化计费 | CPA_bid * ctr * cvr * 1000 | auction_price / ctr / cvr / 1000 = CPA_bid | 效果 | 1000 * ctr * cvr |
| oCPC | optimized CPC | 按照广告转化出价，按照广告点击计费 | CPC_bid * ctr * 1000 = CPA_bid * ctr * cvr * 1000 | auction_price / ctr / 1000 = CPA_bid * cvr / 1000 = CPC_bid | 效果 | 1000 * ctr |
| oCPM | optimized CPM | 按照广告转化出价，按照广告展示计费 | CPM_bid = CPA_bid * ctr * cvr * 1000 | auction_price / 1000 = CPM_bid / 1000 = CPA_bid * ctr * cvr| 效果 | 1000 |

CPC和CPA需要广告主将click、convert/action数据上报给广告平台，广告平台才能计算ctr、cvr，作为广告主会有动力不上报真实的click和convert数据，因此出现了oCPC和oCPM。

## 广告boost策略

在计算完 $eCPM$ 之后，广告平台会对广告进行boost干预，以增加或减少广告的投放，具体方式是计算hidden_cost，加到 $eCPM$ 上，得到 $sorted_ecpm$ 。

$$ sorted_ecpm = eCPM + hidden_cost $$ 

hidden_cost的数值可以是正数，也可以是负数，代表对广告的扶持或打压。一般来说，这种调整包括冷启动boost、广告质量打压以及行业的boost等。通过调整hidden_cost，可以有效地控制广告的投放量和展示效果。

## 广告定价
auction_price也就是广告计费，定价主要有 GFP(按照排序第一的出价计费)、GSP(按照排序第二的出价计费)。

$$ auction_price = (sorted\_ecpm - hidden\_cost) / ratio $$

对于GFP，$ sorted\_ecpm $ 就是第一名的 $ sorted\_ecpm $
对于GSP，$ sorted\_ecpm $ 就是第二名的 $ sorted\_ecpm $

ratio是不同出价类型在计费时的auction_price除去的部分。
