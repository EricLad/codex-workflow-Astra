# Final Routing Model

## 目标

codex-workflow-Astra 从最低 token 消耗策略调整为最低任务总成本策略。

核心原则：

> 选择能够高质量一次完成任务的最低充分推理等级。

## 默认开发模式

### Root Agent

```
Model: GPT-5.6 Luna
Reasoning: max fast
```

适用于大部分真实工程任务：

- C++
- Qt
- 多文件修改
- 重构
- 调试
- 测试修复

原因：

Luna Medium 在复杂工程中可能因为分析不足导致返工；Max Fast 优先提高首次方案质量。

## 三种主要模式

## 1. Standard Engineering Mode

```
Luna Max Fast
```

默认模式。

流程：

Luna 分析 → 实现 → 编译 → 测试 → 修复

## 2. Architecture Mode

触发：

- 模块边界设计
- API设计
- 大型重构方案
- 多种实现路线选择

流程：

Luna Max Fast

→ Astra Architect

→ Luna Max Fast 实施

## 3. Long Running Engineering Mode

触发：

- 多小时/多天任务
- 大量历史决策
- 大型架构迁移
- 多阶段重构

架构：

```
Astra Project Lead
        |
        +-- Luna Explorer
        |
        +-- Luna Max Implementer
        |
        +-- Astra Reviewer
```

Astra负责长期方向和决策一致性。

Luna负责工程执行。

## Astra升级条件

仅在以下情况使用：

- 架构决策
- 两轮 Luna Max 仍无法解决的问题
- 并发/生命周期/UB
- 安全关键逻辑
- 高风险迁移
- 长期项目管理

## Reasoning策略

```
简单修改:
Luna Medium

普通开发:
Luna Max Fast

复杂实现:
Luna Max Fast

架构:
Astra Low/Medium

疑难诊断:
Astra High

极端问题:
Astra xhigh/max
```

## 重要原则

不要为了节省单次 token 使用低推理模型导致多轮返工。

优化目标：

```
总消耗 = 初始分析成本 + 修改成本 + 返工成本
```

而不是：

```
总消耗 = 第一次请求 token
```
