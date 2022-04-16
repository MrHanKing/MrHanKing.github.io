# 事件系统 EventSystem

- Unity 的 EventSystem 主要从三类脚本来看:
  1. EventSystem.cs: 处理输入、射线检测、发送事件的中心系统
  2. XXXInputModule.cs: 输入模块具体的配置和输入处理 以及事件行为
  3. XXXRaycaster.cs: 射线检测的具体执行类

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
// 注意Display是显示器的类 displays[0]特殊 始终返回桌面分辨率

// 根据Blocking配置 计算UI被2D或3D物体遮挡时的 遮挡物距离 用于后面遮挡时候的逻辑

// 进行eventPosition的Raycast测试。注意这里的pos坐标是ScreenPoint坐标 按需转换。然后得到该点的UI元素List
// 1. 先检查cull 和 深度
// 2. 检查UI的rectTransform是否包含ScreenPoint
// 3. 若有摄像机 则检查Z深度
// 4. 进行graphic的Raycast测试

// 对取得的UI元素List进行 正反面的判断和屏幕距离的比较后 构建最后要返回的检测通过的UI元素数据 List<RaycastResult>
}
```

- 获得 Raycaster 结果后 事件的具体调用与否在 InputModule 这个行为类里处理。
- GraphicRaycaster 注意属性:
  1. Ignore Reversed Graphics:勾上(翻转到背面不能点击) 取消勾选(不管怎么翻转都能点击)
  2. Blocked Objects: 有物体遮挡(3D or 2D 等)在 UI 前面，并且点击了遮挡部分的时候，是否应该忽略这次点击
  3. Blocking Mask: Blocked Objects 匹配的层

# input 输入

- BaseInputModule 可以取到 input，会在自身所在节点上挂载 BaseInput 脚本。该脚本从 Engine 的 Input 中获取 Input 输入。
- BaseInputModule 有 inputOverride 可以用来自定义 Input。

# 关于 InputSystem 的

- 老的方式用 Input 获取输入, InputModule 进行处理。
- 新的 InputSystem 用 InputActionAsset 定义的所有外部设备的输入行为。UI 的 InputModule 使用 InputActionAsset 的 Action 替换 UI 的操作。所有的输入获取封装在了 InputSystem 内部 可以不用再自己获取了。
- InputSystem 跟旧方式相比, 多了 InputActionAsset 设置和
- 关于新的 InputSystem 如何实现 UI 操作杆功能, 这里有多种做法:
  1. 方法一:
  - UI 操作杆实现基础的逻辑后 去直接调用角色身上的数据层 改变移动数据等。
  - 然后角色在 Update 的时候根据数据做出变化。同时角色身上可以挂载 PlayerInput 之类的 InputSystem 系统提供的脚本 同时响应硬件设备的输入
  - 这个方法可以让 UI 操作杆和硬件输入同时起效，但可能不利于我们统一控制 以及需要耦合操作杆和角色等。
  2. 方法二:
  - 使用 InputActionAsset 的 Generate C#功能生成资源匹配的脚本(GameInput)。然后制作自己的 InputReader 脚本(可以是 ScriptableObject), 去 new GameInput() 并把自己设置为回调 同时实现对应的 callback 函数。以此来实现硬件设备的输入监听
  - 同时 UI 操作杆的逻辑可以直接调用对应 InputReader 的 callback 函数，只需要伪造一份入参就可以了 而这份入参可以直接使用来自硬件设备的输入监听。
  - 好处是输入设置和 callback 函数可以统一, 并且解耦。无需给角色和 UI 分别设置不同的脚本 使用同一份就可以了。
