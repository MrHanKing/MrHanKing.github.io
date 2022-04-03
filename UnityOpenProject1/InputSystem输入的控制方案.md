# 项目例子

1. 该项目启用了新的 Input System 来作为输入控制。可以很好的统一控制输入。
2. 首先有一个 Input Action Asset 资源【GameInput】作为配置文件，配置了所有输入。分成 4 个 Map 分别控制玩家输入、UI 菜单、对话输入、cheats 作弊控制台。
3. 建立了一个【inputReader】的 ScriptableObject 作为输入观察者和输入消息 channel 的功能给全局使用。
4. EventSystem 上的 InputSystemUIInputModule 输入控制被设置为【GameInput】的配置，替换了原来的监听类型来协作 UI 层的输入。

# Think

1. 配置由【GameInput】统一管理并且多平台设置。
2. 整个游戏的输入由【inputReader】统一控制和分发，有利于简化逻辑和解耦。并且设置了 4 个 Map 分管不同层 可以很方便的处理 UI 层和游戏层的输入，例如剧情对话的时候可以直接关闭角色控制输入的监听。
3. 甚至可以扩展不同 Camera 对输入的反应实现家庭玩家类的多人游戏。
