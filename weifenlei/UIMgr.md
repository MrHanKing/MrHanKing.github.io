# UIManager 一种设计思路

## 功能

- 管理各种 UI 模块之间的切换和过渡，确保单一时刻点只存在一个 UI 模块。
- 管理所有弹窗功能，解耦 UI 模块和弹窗的关系。

## 实现思路

- UIManager.cs UI 的控制中心
  1. 提供切换 UI 模块、弹出窗口功能接口。
  2. 内部维护 UI 模块列表和弹窗列表，用于对象池功能以及维护 UI 模块和弹窗间的关系。
  3. 注册所有需要使用的 UI Enum 便于 Test，也可以使用其他配置方法视项目需求而定如表格。
  4. 提供主动释放 win 或 popup 接口或者内置非激活 UI 的释放决策逻辑，控制资源的使用量。
  5. Canvas 的管理。
- UIBase.cs
  1. 抽象类
  2. 只提供抽象接口 GetName 接口，做调用检查和 EasyGo。可以理解为 UI 的唯一标识
- UIWindow.cs (extends UIBase)
  1. 所有 UI 模块脚本基于该类实现 抽象类提供样式和通用方法实现
  2. 通用方法：
     - EasyPresent: 方便的弹窗接口 调用 UIManager 内部的弹窗接口
  3. 抽象接口:
     - Enter(UIWindow last): 入口方法 进入 Window 会执行的内容
     - Exit(UIWindow next): 出口方法 退出 Window 会执行的内容
- UIPopUp.cs (extends UIBase)
  1. 所有弹窗脚本基于该类实现 抽象类提供样式和通用方法实现
  2. 通用方法：
     - Show: 弹窗入口函数 只被 UIManager 调用。
     - Dismiss: 弹窗出口函数 会返回结果 只被自己内部调用。
  3. 抽象接口:
     - Present(object param): 弹窗逻辑所在函数，会传入指定参数，子类只需要实现该方法。
- UITransition.cs
  1. 提供 window -> window 之间的过渡效果类型选择
  2. 提供 popup 的出现效果类型选择
  3. 暂时指定提供给 UIManager 的

## 具体实现

- 注: 查看 RPG_Code 仓库
