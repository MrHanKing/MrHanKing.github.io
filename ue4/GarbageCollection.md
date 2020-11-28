# UE4的GC

* UE4中的内存管理方法分为3种: 垃圾回收、智能指针、标准C++内存管理

## 垃圾回收

- 虚幻实现垃圾回收: 不再被引用或已被显式标记为销毁的【UObject】将被定期清除。
- 如何判断【UObject】不再被引用: 引擎构建了一个引用图表，在图表根部指定了一组【根集】【UObject】，从【根集】开始创建了整个引用树。任何树中搜索不到的对象被假设为不再需要，被删除
- 保持引用的方式: 
    * 【UObject】上被标记了【UPROPERTY】的属性
    * 将指向Object的指针存储在【TArray】或其他引擎容器类中，【TArray、TMap、TSet】
    * Actor和ActorComponent例外，被引用。因为Actor通常被链接回【根集】，例如所属关卡level。而ActorComponent被Actor引用。所以销毁方式为显式标记，Actor调用Destroy，ActorComponent调用DestroyComponent。
- 垃圾回收设置ProjectSetting--Engine--GarbageCollection提供设置。

- 官方文档地址: https://docs.unrealengine.com/zh-CN/Programming/UnrealArchitecture/Objects/Optimizations/index.html

## 智能指针

## 标准C++内存管理
