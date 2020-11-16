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
- 改变一个属性的类型（resultValue原本只给定了Object）
``` 
Editor.Ipc.sendToPanel("scene", "scene:new-property", {
  id: this.target.uuid.value, // curComponent.uuid,
  path: "resultValue",
  type: "EditorStyleVO",
});
``` 
- 改变一个属性为一个数组 看起来要先new-property改变类型 在set成一个数组 注意isSubProp（错误 这个版本属性会消失 原因Array在定义的时候需要给定类型 cocos的接口没找到可以改变数组内类型的）
``` 
Editor.Ipc.sendToPanel("scene", "scene:set-property", {
  id: this.target.uuid.value, // curComponent.uuid,
  path: "result",
  type: "EditorStyleVO",
  value: [],
  isSubProp: true,
});
``` 

## 如何使用非继承自cc.component的class的Vue组件

### 原因

- cocos creator的inspector装饰器在2.2.2的使用中只能支持对继承自cc.component的组件使用。所以对自己定义的class无法在属性检查器中实现合适的面板渲染

### 大致实现思路

- 重点是要找到并改造cocos内部构建每个属性的节点数据函数【buildNode】
1. 编写自己的class和对应的vue组件
2. hook【buildNode】函数注入自己的回调 
3. 在自己的回调中遍历查找cocos构建好的属性节点 找到对应的class修改他的compType为自己的Vue组件类型

### 代码细节

- 首先在系统初始化的时候hook cocos内部的函数，在addEditorCustomVueCom函数里注册自己的class和vue组件的映射关系到一个dictionary中
```
function addEditorCustomVueCom(){
  // 伪代码
  customCompNames[className] = vueComName;
  // 例如
  customCompNames['EditorTemplateAttr'] = 'editor-template-attr';
}
```
```
function init() {
    addEditorCustomVueCom();

    let inspectorUtil = Editor.require('packages://inspector/utils/utils');
    let oldFun = inspectorUtil.buildNode;
    inspectorUtil.buildNode = function (e, r, a) {
        oldFun(e, r, a);
        handleCompType(r);
    };
}
```
- 在回调中修改【compType】。【customCompNames】就是自己生成的记录映射关系的dictionary。注意2点
  1. hook有风险
  2. 注意```value.compType !== 'cc-array-prop'```用来防止篡改数组，因为对象数组的类型cocos也解析成了数组里面对象的类型
```
function handleCompValue(value) {
    if (value.attrs && value.attrs.type in customCompNames && value.compType !== 'cc-array-prop') {
        // 如果是数组 不用替换类型
        value.compType = customCompNames[value.attrs.type];
    }

    if (value.type == 'Array') {
        for (let index = 0; index < value.value.length; index++) {
            const subValue = value.value[index];
            handleCompValue(subValue);
        }
    } else if (value.type == 'Object') {
        for (const subKey in value.value) {
            const subValue = value.value[subKey];
            handleCompValue(subValue);
        }
    }
}

function handleCompType(compData) {
    for (const compKey in compData) {
        if ('__comps__' !== compKey) {
            continue;
        }

        let comps = compData[compKey];
        for (let index = 0; index < comps.length; index++) {
            const comp = comps[index];
            for (const valueKey in comp.value) {
                const value = comp.value[valueKey];
                handleCompValue(value);
            }
        }
    }
}
```

