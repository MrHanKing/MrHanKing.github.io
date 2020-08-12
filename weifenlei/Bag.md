# Bag 的一种实现

## 功能

- 展示物品数据内容。
- 提供切页、ScrollView、动态加载、性能优化支持。

## 实现思路

- BagCtrl.cs

  1. BagWin 的主要控制器，提供切页和刷新 ScrollView 的功能
  2. 接口:
     - InitViewContent: 初始化内容
     - OnPageButtonClick: 切页逻辑
     - OnBtnCloseClick: 关闭界面逻辑
     - RefreshScrollView: 刷新 ScrollView

- ItemPanelCtrl.cs
  1. 当个物品的显示具体内容控制器。
  2. 只提供 refresh 接口。
- BagCtrlDymaic.cs

  1. BagCtrl 的加强版本，支持无上限的 item 显示。
  2. 使用限定计算出来的 item 数量，根据拖动情况不断刷新位置和内容不再增加实际 ob 数量。

- 数据来源，不用实现自己的 Manager，从其他数据模块获得 item 数据。

## 具体实现

- 注: 查看 U3d_Game_3D_RPG 仓库
