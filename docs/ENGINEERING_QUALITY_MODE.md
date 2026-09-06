# Engineering Quality Mode

## v0.4 调整目标

根据实际开发反馈，默认 GPT-5.6 Luna Medium 在大型 C++/Qt 工程中容易产生较高返工率，因此默认执行策略调整为质量优先。

## 新默认策略

Root Agent:

- GPT-5.6 Luna
- reasoning: max fast

适用于：

- C++/Qt 工程开发
- 多文件修改
- 重构
- 调试
- 需要理解现有架构的任务

## Reasoning 路由

### Luna Medium

仅用于：

- 简单修改
- 明确实现
- 低风险任务

### Luna Max Fast（默认）

用于：

- 普通功能开发
- UI 重构
- 多文件修改
- 调试
- 代码理解

### Astra Low/Medium

用于：

- 架构决策
- 模块边界设计
- 大型迁移方案

### Astra High

用于：

- 并发问题
- 生命周期问题
- UB
- 死锁
- 安全关键问题

## 成本原则

目标不是最低单次 token，而是最低完成任务总成本。

如果 Medium 导致多轮返工，Max Fast 可能更经济。

## 长周期任务

大型持续任务使用：

Astra Project Lead
+
Luna Max Worker

Astra负责长期方向和决策，Luna负责工程实现。
