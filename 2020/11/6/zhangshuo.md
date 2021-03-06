原文：https://medium.com/box-tech-blog/how-we-learned-to-stop-worrying-and-read-from-replicas-58cc43973638

### 问题
* 主和从一致性问题，导致写主读从策略有风险

### 解决
#### 常规方法
1. 直到从同步后，主才 ACK
2. 直到从同步后，才允许用户读从
   ##### BOX 实践
   1. 将读策略分为两种，一种只追求最终一致，一种强调写后读
   2. 根据 use-case 分为对延迟敏感和不敏感的。不敏感的应用常规方案；敏感的，轮询从，一旦有一个和主同步，查询该从，不然返回超时
   3. 策略存在风险。主从延迟大时，即主有大量写，从此时频繁读主，导致主压力更大
   4. 通过方法存在以下限制
        * 合理的复制延迟时间内会存在高延迟
        * 复制严重滞后时会持续失败

#### 替代方法
1. 即使从延迟，只要用户关心的数据到达从，则返回
2. 通过写时将复制位置一并发送到数据层