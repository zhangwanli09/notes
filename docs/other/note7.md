# MVC、MVP、MVVM

### MVC

MVC 是是应用最广泛的软件架构之一：`Model`（模型）、`View`（视图）、`Controller`（控制器）。

1. View 传送指令到 Controller
2. Controller 完成业务逻辑后，要求 Model 改变状态
3. Model 将新的数据发送到 View，用户得到反馈

Controller 是 Model 和 View 的协调者。View 和 Model 不直接联系。通信都是`单向的`。

### MVVM

MVVM 是把 MVC 中的 Controller 改变成了`ViewModel`。

View 的变化会自动更新到 ViewModel，ViewModel 的变化也会自动同步到 View 上显示，通过数据驱动视图层。（双向绑定）。

优势：

1. 低耦合。
2. 开发人员可更专注与业务逻辑和数据。
