# Qt UI Refactor Template

## 适用场景

- QWidget/QML 界面重构
- MainWindow 拆分
- UI 与业务解耦
- Signal/Slot 调整

## 默认模式

GPT-5.6 Luna Max Fast。

## 执行阶段

### Phase 1: 分析

检查：

- UI结构
- QObject生命周期
- Signal/Slot关系
- 业务依赖
- 线程边界

### Phase 2: 设计判断

如果发现：

- MainWindow职责过多
- 模块边界不清
- 需要长期架构调整

升级 Astra Architect。

### Phase 3: 实现

Luna Max 执行：

- 类拆分
- UI修改
- CMake调整
- 编译验证

### Phase 4: Review

检查：

- UI线程安全
- QObject ownership
- 信号连接正确性
- 回归风险
