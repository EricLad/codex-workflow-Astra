# Quality First Routing

## 核心变化

从 token 最小化调整为完成任务总成本最优。

错误策略：

Luna Medium 处理所有任务。

正确策略：

选择能够一次完成任务的最低充分推理等级。

## 路由

|任务|模型|Reasoning|
|-|-|-|
|机械修改|Luna|Low/Medium|
|普通开发|Luna|Max Fast|
|复杂实现|Luna|Max Fast|
|架构设计|Astra|Low/Medium|
|疑难诊断|Astra|High|
|长期大型项目|Astra Lead + Luna Max|

## 升级条件

以下情况升级 Astra：

- Luna Max 两轮证据驱动尝试仍无法解决
- 存在架构级决策
- 存在并发、生命周期、安全风险
- 需要长期保持多个历史设计决策

## 降级原则

Astra完成：

- 架构方案
- 根因分析
- Review

后立即返回 Luna 执行。