---
name: codex-workflow-astra
description: Quality-first software engineering workflow using GPT-5.6 Luna Max as the default engineering worker and GPT-6 Astra for architecture, long-running projects, critical review, and hard diagnosis.
---

# Codex Workflow Astra

## 核心目标

本工作流不追求最低单次 token 消耗，而追求：

> 在可控成本下，以最低返工率完成高质量工程任务。

默认策略已经从「低成本优先」调整为「质量优先」。

## 默认模型策略

### Root Agent 默认

```
Model: GPT-5.6 Luna
Reasoning: max fast
```

适用于大多数真实软件工程任务：

- C++
- Qt
- 多文件修改
- 重构
- 调试
- 架构内实现

原因：

对于复杂工程，较高初始推理投入通常可以减少：

- 错误方案
- 遗漏依赖
- 重复修改
- 测试失败后的返工

## Agent 定位

### Luna Max Engineer

默认工程执行者。

负责：

- 分析代码
- 修改实现
- 编译测试
- 调试修复
- 完成功能

### Luna Medium

仅用于低风险任务：

- 明确的小修改
- 简单代码生成
- 局部调整
- 机械性实现

### Astra Architect

用于：

- 架构设计
- 模块边界
- API设计
- 大型迁移方案

默认：

```
GPT-6 Astra
Reasoning: low/medium
```

Astra输出设计决策后，应返回 Luna 执行。

### Astra Diagnostician

用于：

- 死锁
- race condition
- QObject 生命周期问题
- UB
- 崩溃根因分析
- 高风险系统问题

默认：

```
GPT-6 Astra
Reasoning: high
```

## 三种运行模式

# Mode 1: Standard Engineering

默认模式。

流程：

```
Luna Max Fast
    ↓
实现
    ↓
验证
    ↓
完成
```

适合：

- Qt UI开发
- 普通功能
- 重构
- Bug修复

---

# Mode 2: Architecture Mode

当任务存在架构选择时启用。

流程：

```
Luna Max
    ↓
Astra Architect
    ↓
设计决策
    ↓
Luna Max实现
```

不要让Astra承担普通编码工作。

---

# Mode 3: Long Running Engineering Mode

适用于：

- 多天任务
- 大型重构
- 多阶段迁移
- 需要长期保持设计一致性的项目

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

Astra负责：

- 长期方向
- 架构一致性
- 决策记录

Luna负责：

- 实际工程实现

## 升级规则

升级 Luna 到 Astra：

1. 存在重要架构决策；
2. 涉及并发、生命周期、安全、数据完整性；
3. Luna Max 已进行两轮证据驱动尝试仍无法解决；
4. 需要高置信度独立审查。

不要因为：

- 文件多
- diff大
- 项目大

直接升级 Astra。

## Reasoning 选择

```
机械任务              Luna Medium
普通开发              Luna Max Fast
复杂实现              Luna Max Fast
架构决策              Astra Low/Medium
疑难诊断              Astra High
极端问题              Astra xHigh/Max
```

Max 不作为普通默认升级路径。

## Astra使用原则

Astra解决高价值问题：

- 架构
- 判断
- 根因
- Review

完成后：

```
Astra Decision
        ↓
Handoff
        ↓
Luna Implementation
```

不要长期让Astra执行普通代码修改。

## Handoff格式

```text
ROUTE HANDOFF
Model: GPT-5.6 Luna | GPT-6 Astra
Reasoning: medium | high
Purpose: implementation | planning | diagnosis | review
Task:
Context:
Constraints:
Success criteria:
Evidence:
Return:
```

## 完成标准

任务完成必须满足：

- 功能或分析目标完成；
- 编译/测试结果明确；
- 临时分支、worktree、实验修改已处理；
- 最终说明修改内容和剩余风险。
