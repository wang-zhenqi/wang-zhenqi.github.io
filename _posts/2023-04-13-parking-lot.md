---
layout: post
title: parking lot insurance
---

# 一、背景

（这一部分验证我对业务场景的理解是否正确。）

车主在停车场停车时，不免会遇到超出计费区间的情况：例如超过免费停车时长、超出出场时间、以及时间累计到下一计费区间等等。作为车主来说，仅仅因为几分钟之差就要多交停车费是非常令人郁闷的。基于这种情景，保险公司可以开发一款新的产品，姑且称作停车超时险，抓住车主不愿多交钱的心理，解决因停车超时而额外交钱的痛点。

# 二、产品定位

（这一部分来自于我的疑问——是针对什么人群而设计的保险？昨天在和佳兴的通话中这点比较模糊。这一部分的分析主要是帮助我理清思路，纯个人看法。）

## 1. 消费者

在背景分析中已经提到，车主不希望为几分钟的超时付出额外的费用，而超时的情况又时常会发生，因此车主更希望有保险公司来分担风险。投保人可能会有两种情况。

### （1）停车场

停车场作为投保人，可以为前来停车的车主购买停车超时险，从而提升车主的停车体验，吸引更多车主。

由保险公司支付超时费用，保险公司可能会遇到的风险就是停车场会从两边获利——一方面收取车主的超时停车费，另一方面用车主超时的证据向保险公司索赔（也可能是其他更隐蔽的做法）——从而引起不必要的理赔。如果需要对这类行为进行监管，那么保险公司势必要增加额外的投入。

由停车场作为投保人时，保单数量受到停车场进场量的限制，停车场的投保意愿受到利润的限制。

### （2）车主

车主也同样适合作为投保人和受益人。车主的投保意愿可能受限于他的行为模式以及保险定价。

### （3）导航软件

导航软件的用户粘性主要来自于精准、高效的导航服务，以及交通相关的附加服务，例如酒店预定、目的地周边消费优惠等。可以看到导航软件会给用户推荐目的地附近的停车场，以帮助用户节约停车时间。那么他们也很有可能愿意通过为用户购买停车超时险来提升产品体验和用户粘性。

## 2. 价格定位

该产品的价格主要受到停车超时场景的出现概率、时间和地域的差异，以及停车场定价水平等因素的影响。基本特征是小额、便捷、普适。由于价格受到市场供需关系以及消费水平的影响，因此最好能够针对不同时间、区域、客户群体提供不同的策略。

所需分析的数据见下文。

# 三、数据分析

## 1. 停车超时场景的出现概率（频率）

### （1）统计各停车时段的缴费数占比（已做）

目的是为了找出停车时长的分布规律。

### （2）重点关注短超时的概率

短超时是指超出停车计费区间不久的订单，这一部分订单可能更需要这个保险。“短”的定义可以在统计的过程中讨论，根据分析结果做调整。也可将不同的超时时长做加权平均，算出超时情况的期望值。

### （3）同一辆车多次超时的概率（已做）

多次超时的车主更有可能成为潜在客户。

## 2. 时间、地域的影响

### （1）在年、季度、月、周、天的粒度下，各个时间段所出现的停车超时频率（比例），以及超时情景下支付的费用的期望

目的是找出投保需求最频繁的时间，根据供需关系厘定价格。同时，高需求时段也会影响停车场或者导航软件为顾客购买保险的意愿。

### （2）不同省、市、等行政区域内，不同类型的停车场内，出现停车超时的频率

这个数据会影响在不同区域内，人们对超时险的需求程度。也可帮助保险公司厘定与该区域相匹配的定价。

### （3）不同时空下，因超时而收取的停车费，占总停车费的比例

这个比例会让停车场考虑是否要为顾客购买超时险。

### （4）不同地域内，人均消费水平、汽车保有量、停车频率等数据之间的关系

这个数据可以用于挖掘潜在客户，例如有多少人原本不愿开车出门，但可能会因为有了超时险而选择开车。这里的数据集不一定是这些，需要具体调研才能找出从哪里获取到这样的信息。

## 3. 停车场收费情况

这里描述的情景需要更加细化到具体的某类停车场，例如同一城市的，同一商区的，或者同一收费标准的等等。

### （1）停车场因超时而获得的收益占总收益的比例

### （2）停车场临时车与长期车的比例，不同时间粒度下，双方收费占比是多少。

### （3）停车场内出场时间的长短

这个时间会影响车主短超时的出现概率。

# 四、总结

对于停车超时险的定价分析，主要要考虑超时所需缴纳的费用和保费之间的关系。这个关系会受到停车场的定价以及客户数量等因素的影响。因此在厘定保费定价时，应该充分考虑超时场景的出现频率、造成额外支出的金额、以及潜在客户的数量等方面。

目前我能想到的就只有这些建议，是否能落地需要通过实际数据来进行分析。