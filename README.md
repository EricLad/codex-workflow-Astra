# codex-workflow-Astra

面向 Codex 的 **GPT-6 Astra + GPT-5.6 Luna 成本感知编程工作流**。

目标不是“尽量少用 Astra”，而是把 Astra 的额度集中在高认知密度节点，把大量实现、搜索、编译、测试和常规修复交给 Luna，从而同时维持工程质量和 Plus 用户的可持续使用成本。

> **Astra Think → Luna Build → Astra Judge → Luna Finish**

## 核心原则

1. **默认 Luna，不默认 Astra。**
2. **默认中等思考，不默认最高思考。**
3. **模型选择与 reasoning effort 分开决策。**
4. **Astra 只用于高价值决策、疑难问题和关键审查。**
5. **同一失败假设最多重试两轮，随后升级而不是反复消耗。**
6. **Astra 完成判断后立即降级回 Luna 执行。**
7. **Review 优先读取变更包，而不是重新扫描整个仓库。**
8. **小改动只做与风险匹配的验证，避免无意义的全量测试。**

## 默认路由

| 任务 | 模型 | reasoning |
|---|---|---|
| 机械修改、搜索、重命名、格式整理 | GPT-5.6 Luna | `none` / `low` |
| 日常功能开发、常规修复、测试 | GPT-5.6 Luna | `medium` |
| 较复杂实现、普通疑难调试 | GPT-5.6 Luna | `high` |
| 普通架构判断、计划、关键 diff review | GPT-6 Astra | `low` |
| 跨模块设计、复杂架构分析 | GPT-6 Astra | `medium` |
| 并发、生命周期、UB、复杂 root cause、安全关键审查 | GPT-6 Astra | `high` |
| 极困难系统问题 | GPT-6 Astra | `xhigh` |
| 极少数最终关键判断 | GPT-6 Astra | `max`，仅显式升级 |

`max` 不是日常档位。工作流默认禁止自动进入 `max`。

## 工作流

```text
User task
   │
   ▼
Classify task + risk
   │
   ├─ Routine / bounded ───────────────► Luna
   │                                     │
   │                                     ├─ implement
   │                                     ├─ build/test
   │                                     └─ fix
   │
   └─ Architecture / high-risk / hard ─► Astra
                                         │
                                         ├─ decide / diagnose / review
                                         ▼
                                       Luna
                                         │
                                         └─ implement / finish
```

典型闭环：

```text
Astra plan
  ↓
Luna implementation
  ↓
Luna validation
  ↓
Astra review only when risk justifies it
  ↓
Luna fixes
  ↓
targeted final validation
```

## Astra 升级条件

出现以下任一情况时，从 Luna 升级：

- 需要做跨模块架构或接口边界决策；
- 涉及并发、线程退出、对象生命周期、内存安全或未定义行为；
- 涉及认证、权限、加密、协议兼容、数据迁移等高风险逻辑；
- root cause 不清晰，并且 Luna 对同一问题已经进行了两轮有依据的尝试仍未解决；
- 修改范围大且存在多个互相制约的设计选择；
- 关键变更需要高置信度独立审查。

以下情况**不构成**升级理由：

- 文件很多但修改机械；
- 编译报错本身；
- 普通 CRUD；
- 明确设计下的实现工作；
- 常规 CMake、UI、测试、文档修改；
- 单纯因为“任务看起来很大”。

## Astra Review Packet

调用 Astra 做审查时，优先提供最小充分上下文：

```text
Goal
Design decisions
Changed files / git diff
Relevant interfaces or call chain
Validation results
Known risks / unresolved questions
Review focus
```

除非发现证据需要扩大范围，否则不要让 Astra 从头重新读取整个仓库。

## 运行时模型切换

本项目定义的是**模型与思考等级路由策略**，不伪造 Codex 运行时不存在的切换能力。

- 如果当前 Codex 环境支持为 subagent / thread 指定模型与 reasoning，则直接按路由结果委派。
- 如果当前环境不能自动选择模型，则工作流必须输出明确的 handoff：`model + reasoning + task + context packet`。
- 不允许声称已经切换模型，除非运行时确实完成了切换或委派。

## 文件

- [`SKILL.md`](./SKILL.md) — Codex 可加载的核心工作流指令。
- [`WORKFLOW.md`](./WORKFLOW.md) — 完整执行状态机与开发流程。
- [`docs/ROUTING.md`](./docs/ROUTING.md) — 模型与 reasoning 双路由规则。
- [`docs/HANDOFF.md`](./docs/HANDOFF.md) — Astra/Luna 交接包格式。

## 使用方式

将本仓库作为一个独立 skill 目录提供给 Codex，并让 Codex 读取根目录的 `SKILL.md`。如果你的 Codex 环境支持用户级或项目级 Skills，按该环境当前的 Skills 安装方式导入本目录即可。

然后在任务中明确启用 `codex-workflow-astra`，由工作流负责判断：

1. 当前任务属于 Luna 还是 Astra；
2. 应使用什么 reasoning effort；
3. 是否需要拆分/并行；
4. 什么时候升级；
5. 什么时候立即降级；
6. 需要做到什么程度的测试和 review。

## 设计边界

本项目只负责**工程执行策略与模型路由**，不会覆盖目标仓库自身的：

- `AGENTS.md` 项目规则；
- 构建系统与测试命令；
- 代码风格；
- 提交规范；
- 安全或发布流程。

用户的明确指令以及目标项目自身的硬性要求优先于本工作流的默认建议。若发生冲突，工作流应指出冲突来源，并遵循更高优先级的明确要求。

## 状态

当前为 **v0.1 初始版本**。重点是建立稳定、可解释、成本敏感的 Astra/Luna 双模型开发闭环。