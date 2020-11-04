# Creator 2.2.2版本

## 动态抓取并创建属性内容
- 详见下面代码：
- 设置单个属性的值 path可以是xxx.xxx.xxx 看起来都是改变单值 不支持直接设置对象
``` 
Editor.Ipc.sendToPanel("scene", "scene:set-property", {
  id: this.target.uuid.value, // curComponent.uuid,
  path: "stringValue",
  type: "String",
  value: "hahaha",
  isSubProp: false,
});
``` 
- 改变一个属性的类型
``` 
Editor.Ipc.sendToPanel("scene", "scene:new-property", {
  id: this.target.uuid.value, // curComponent.uuid,
  path: "resultValue",
  type: "EditorStyleVO",
});
``` 
- 改变一个属性为一个数组 看起来要先new-property改变类型 在set成一个数组 注意isSubProp
``` 
Editor.Ipc.sendToPanel("scene", "scene:set-property", {
  id: this.target.uuid.value, // curComponent.uuid,
  path: "result",
  type: "EditorStyleVO",
  value: [],
  isSubProp: true,
});
``` 
