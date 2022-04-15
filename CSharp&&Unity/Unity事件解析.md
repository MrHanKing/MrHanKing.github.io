# 事件系统 EventSystem

- Unity 的 EventSystem 主要从三类脚本来看:
  1. EventSystem.cs: 处理输入、射线检测、发送事件的中心系统
  2. XXXInputModule.cs: 输入模块具体的配置和输入处理
  3. XXXRaycaster.cs: 射线检测的具体行为类

## EventSystem.cs 源码解读

- 首先来看核心存储的数据

```c#
// 存储了自身挂载节点上的所有输入组件列表 所以可以根据需求切换不同的输入组件
private List<BaseInputModule> m_SystemInputModules = new List<BaseInputModule>();
// 当前在处理的输入组件
private BaseInputModule m_CurrentInputModule;
// EventSystem 激活的时候会把自己注册进来。 从这里可以知道理论上支持多EventSystem
private static List<EventSystem> m_EventSystems = new List<EventSystem>();

// 当前使用EventSystem 取List里index为0的值。set时会把对应m_EventSystems里面匹配的EventSystem提升到index为0的位置
public static EventSystem current{...}

// 需要注意UpdateModules()方法 这个方法会在InputModule的OnEnable时机触发 会刷新m_SystemInputModules只记录所有激活的输入模块
```

- 接着来看 Update 具体的处理

```c#
// 不复制具体代码出来了

// 对所有激活的XXXInputModule进行一次UpdateModule()调用: 如StandaloneInputModule具体实现是对上一帧未处理的事件进行处理 并记录当前帧鼠标位置。
TickModules();

// 切换m_CurrentInputModule 到第一个激活着的InputModule 如没有就启用List里最前面的
// 注意这里 如果当前帧切换了InputModule 是不处理事件的

// 调用InputModule的 Process()函数进行处理

// 综上可以看出只有一个InputModule会被执行 若需要支持多InputModule同时执行需要自己扩展改写Update
```

# StandaloneInputModule.cs

- Unity 有很多种 InputModule 这边拿标准的输入模块举例
- 接着解析 Process() ProcessTouchEvents() 和 GetTouchPointerEventData()

```c#
public override void Process()
{
    // 检查焦点状态
    if (!eventSystem.isFocused && ShouldIgnoreEventsOnNoFocus())
        return;

    // 通知选中UI对象的IUpdateSelectedHandler接口
    bool usedEvent = SendUpdateEventToSelectedObject();

    // 先触发touch 若失败了再出发鼠标
    if (!ProcessTouchEvents() && input.mousePresent)
        // 鼠标检测在开始的GetMousePointerEventData()函数内便进行了射线检测和数据存储
        ProcessMouseEvent();

    if (eventSystem.sendNavigationEvents)
    {
        if (!usedEvent)
            // 移动间隔回调 IMoveHandler
            usedEvent |= SendMoveEventToSelectedObject();

        if (!usedEvent)
            // 选中和取消 ISubmitHandler 和 ICancelHandler
            SendSubmitEventToSelectedObject();
    }
}

// 以touch举例
private bool ProcessTouchEvents()
{
    for (int i = 0; i < input.touchCount; ++i)
    {
        // 注意Touch有FingerID
        Touch touch = input.GetTouch(i);

        if (touch.type == TouchType.Indirect)
            continue;

        bool released;
        bool pressed;
        // 获取touch点的数据 Raycast检测发生在这里
        var pointer = GetTouchPointerEventData(touch, out pressed, out released);

        // 执行 IPointerDownHandler、IPointerClickHandler、IDragHandler、IEndDragHandler、IPointerExitHandler等接口
        // 点击 拖拽相关
        // 注意这里的回调接口都是冒泡向上逐级查找对象的
        ProcessTouchPress(pointer, pressed, released);

        if (!released)
        {
            ProcessMove(pointer);
            ProcessDrag(pointer);
        }
        else
            RemovePointerData(pointer);
    }
    return input.touchCount > 0;
}

protected PointerEventData GetTouchPointerEventData(Touch input, out bool pressed, out bool released)
{
    /// ... 代码
    if (input.phase == TouchPhase.Canceled)
    {
        pointerData.pointerCurrentRaycast = new RaycastResult();
    }
    else
    {
        // 在这里根据输入点的位置进行射线检测 并排序
        eventSystem.RaycastAll(pointerData, m_RaycastResultCache);

        var raycast = FindFirstRaycast(m_RaycastResultCache);
        pointerData.pointerCurrentRaycast = raycast;
        m_RaycastResultCache.Clear();
    }
    return pointerData;
}
```

- 从上整个流程就通了 EventSystem.Update() -> InputModule.Process() -> Touch/Mouse 等行为准备数据 -> Raycast 获得具体对象 -> handler 回调
- 这里还有 2 点还没解析到, 后面继续:
  1. Touch/Mouse 那里使用了 input 的数据是在哪里准备好的。
  2. Raycast 里 XXXRaycaster.cs 是怎么注册进来的。

# GraphicRaycaster.cs 以 UI 的射线检测为例

- 从所有 Raycaster 的基类 BaseRaycaster 中可以知道, 在 OnEnable 中会注册自己进 RaycasterManager。OnDisable 的时候从 Manager 中删除注册。
- EventSystem 直接调用 RaycasterManager。
- 具体 Raycaster 类实现 Raycast 函数来被调用

```c#
public override void Raycast(PointerEventData eventData, List<RaycastResult> resultAppendList)
{
// 注意GraphicRegistry这个类. 所有canvas上的Graph组件IsRaycastTarget是true 则会把自身根据canvas分类注册进GraphicRegistry这个单例里

// 根据Canvas设置获取对应的displayIndex 从Canvas上或Camera上取
// 注意Display.RelativeMouseAt(eventData.position); 它转换出来的z值和displayIndex匹配 不匹配就不响应Raycast了
}
```
