# Bug Investigation Template

## 默认模式

GPT-5.6 Luna Max Fast。

## 调查流程

1. 收集错误现象
2. 定位影响模块
3. 分析日志、调用链、状态变化
4. 生成假设
5. 验证根因
6. 修复并测试

## 升级 Astra Diagnostician 条件

- Crash无法稳定复现
- 多线程问题
- 死锁
- QObject生命周期异常
- UB风险
- Luna Max 两轮分析失败

## Astra 输出要求

必须提供：

- Root Cause
- Evidence
- Fix Strategy
- Validation Plan
