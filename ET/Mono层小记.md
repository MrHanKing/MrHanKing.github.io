# 大纲

- Mono 层主要存放了 ILRuntime 相关辅助类、扩展数据处理类、网络层 KCP 和 TCP、资源 AssetsBundle 相关类、数据库相关、基础 MonoBehaviour 扩展、生命周期控制层

## 生命周期控制层 CodeLoader

- 热更脚本 dll 加载和其内生命周期委托声明在此处
- 启动后调用 ModelView 层的 Entry.cs 类的 Start 函数 即整个热更层的入口。binding 生命周期、注册事件 和启动热更层场景

## 网络层 KCP 和 TCP

## ILRuntime 相关辅助类

## 扩展数据处理类

## 资源 AssetsBundle 相关类

## 数据库相关

## 基础 MonoBehaviour 扩展
