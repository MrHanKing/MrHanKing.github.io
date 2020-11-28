# UE4的GC

* UE4中的内存管理方法分为3种: 垃圾回收、智能指针、标准C++内存管理

## 垃圾回收

- 虚幻实现垃圾回收: 不再被引用或已被显式标记为销毁的【UObject】将被定期清除。
- 如何判断【UObject】不再被引用: 引擎构建了一个引用图表，在图表根部指定了一组【根集】【UObject】，从【根集】开始创建了整个引用树。任何树中搜索不到的对象被假设为不再需要，被删除
- 保持引用的方式: 
    * 【UObject】上被标记了【UPROPERTY】的属性
    * 将指向Object的指针存储在【TArray】或其他引擎容器类中，【TArray、TMap、TSet】
    * Actor和ActorComponent例外，被引用。因为Actor通常被链接回【根集】，例如所属关卡level。而ActorComponent被Actor引用。所以销毁方式为显式标记，Actor调用Destroy，ActorComponent调用DestroyComponent。
    * 【UStructs】不被垃圾回收。可通过智能指针管理
    * 非派生自【UObject】的class保持变量引用不被垃圾回收。派生自【FGCObject】并覆盖AddReferencedObjects方法把指针加入到收集器内。
- 垃圾回收设置ProjectSetting--Engine--GarbageCollection提供设置。

- 官方文档地址: https://docs.unrealengine.com/zh-CN/Programming/UnrealArchitecture/Objects/Optimizations/index.html

## 智能指针

- 注意:
   * 引用计数
   * 不要在构造函数中调用AsShared或Shared，共享引用此时并未初始化。会导致崩溃或throw
- 智能指针类型
   * 共享指针 TSharedPtr: 拥有引用对象，在无共享指针或共享引用时处理删除对象。指针可空，可为其引用对象生成共享引用。
   * 共享引用 TSharedRef: 拥有引用对象，引用对象非空，可转换为引用非空共享指针。
   * 弱指针  TWeakPtr: 不拥有引用对象，可中断引用循环。可生成引用对象的共享指针。
   * 唯一指针 TUniquePtr: 显式拥有其引用对象。自身被销毁时自动删除其引用的对象。不应与共享指针和共享引用混用。


- 官方文档地址: https://docs.unrealengine.com/zh-CN/Programming/UnrealArchitecture/SmartPointerLibrary/index.html

## 标准C++内存管理
