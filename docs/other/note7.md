# MVC，MVP，MVVM

### MVC

MVC 是是应用最广泛的软件架构之一：`Model`（模型）、`View`（视图）、`Controller`（控制器）。

1. View 传送指令到 Controller。
2. Controller 完成业务逻辑后，要求 Model 改变状态。
3. Model 将新的数据发送到 View，用户得到反馈。

Controller 是 Model 和 View 的协调者。View 和 Model 不直接联系。通信都是`单向的`。

### MVP

MVP 模式将 Controller 改名为`Presenter`，同时改变了通信方向。

1. 各部分之间的通信，都是双向的。
2. View 与 Model 不发生联系，都通过 Presenter 传递。
3. View 不部署任何业务逻辑，称为`被动视图`（Passive View），即没有任何主动性，而 Presenter 部署所有逻辑。

### MVVM

MVVM 可以看做是一种特殊的 MVP 模式，或者说是对 MVP 模式的一种改良。将 Presenter 改名为`ViewModel`，基本上与 MVP 模式完全一致。

区别是，它采用`双向绑定`（data-binding）：View 的变动，自动反映在 ViewModel，反之亦然。通过数据驱动视图层。

优势：

1. 低耦合。
2. 开发人员可更专注与业务逻辑和数据。

> [参考](https://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html)
