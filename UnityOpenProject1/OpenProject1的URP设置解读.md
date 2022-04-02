# read

- 关于 Unity 官方开源项目的笔记

# URP 设置流程及细节

## URP Asset

1. URP Asset 设置了 Depth Texture 和 Opaque Texture

## ToonRendererData

1. ToonRendererData 是使用的前向渲染管线资源
2. 首先来看 Filtering 下 OpaqueLayerMask 和 TransparentLayerMask 分别指明了不透明物体和透明物体需要渲染哪些 layer 物体
3. 接下来看 RenderFeature 首先根据 Event 的类型理清楚各个 RenderFeature 处理的时机和顺序

### 【OccludingObjects】这个 Feature

1. 在正常渲染完不透明物后先执行。根据 Filters 中对符合【Opaque】队列和【OccludingObjects】layer 的物体的图片像素进行一次处理。
2. Overrides 中 Depth 没有勾选 所以走默认深度测试
3. Stencil 勾选了 进行模版测试 以下具体值行为
   - Value: 比较值 这个值用来和模版缓冲中存储的值进行比较 此处使用了 1
   - Compare Function: 比较 Value 值和缓冲中存储值的规则，通过则渲染像素。此处配置了 Always 即【Opaque】的【OccludingObjects】对象的像素始终渲染
   - Pass: 通过 Compare Function 的像素点对模版缓冲值如何处理。此处配置 Replace 意思即【Opaque】的【OccludingObjects】对象所在的像素点的模版缓冲值替换为 Value 值即 1
   - Fail: 没通过 Compare Function 的像素点对模版缓冲值如何处理。此处配置为 Keep 即保持之前的模版缓冲值。此处可以理解为不做处理
   - Z Fail: 通过模版测试但没有通过深度测试的像素点如何处理。此处为 Keep，此处可以理解为不做处理 因为 Depth 测试没开。
4. 这里可以理解为根据 Opaque Texture 在上面的【OccludingObjects】对象的像素部分的对应位置的模版缓冲上写入了全为 1 的值标记了这部分区域 为后续处理做准备

### 【CharacterObstructed】Feature

1. 和上一个同为 AfterRenderingOpaque 时机触发，在【OccludingObjects】之后执行。
2. 为【Opaque】队列和【Characters】即角色的不透明部分再进行一次渲染
3. Material: 渲染使用的材质
   - Pass Index: 处理使用的材质上 shader 的第几个 Pass
4. Depth： 勾选开启深度测试设置覆盖
   - Write Depth：深度测试后是否写入像素的深度。此处不勾选 即只使用深度缓冲中的值进行判断 判断后不进行写入。
   - Depth Test: 深度测试的操作。通过操作为 true 才进行渲染。此处配置为 Greater Equal 即像素深度 >= 缓冲值才进行渲染。像素深度近小远大，此处即角色像素在【OccludingObjects】对象像素后面的物体判断为 true。
5. Stencil：
   - Value： 1
   - Compare Function：Equal 模版缓冲中值和 Value 相等的像素通过模版测试，渲染像素
   - Pass： Keep 通过模版测试的像素不改变模版缓冲值
   - Fail： Keep 不通过模版测试的像素不改变模版缓冲值
   - Z Fail： Keep 通过模版测试但不通过深度测试的像素不改变模版缓冲值
6. 此处结果，模版缓冲的值不改变。在 Opaque Texture 上进行一次渲染，渲染【OccludingObjects】对象像素上符合【Characters】角色在【OccludingObjects】对象之后的像素。即实现了角色被遮挡物遮挡的部分的颜色。

### 【HideWaterMask】 Feature

1. 注意时机 跳过【ObstructedTransparentFiller】Feature 先说【HideWaterMask】，因为前者在渲染透明物体之后才进行。
2. 为【Opaque】队列的【HideWaterMask】对象进行一次渲染
3. Stencil：
   - Value： 5
   - Compare Function：Never 所有符合的像素都没通过模版测试 即不渲染
   - Pass： Keep 通过模版测试的像素不改变模版缓冲值
   - Fail： Replace 不通过模版测试的像素点对应位置的模版缓冲值写入为 5
   - Z Fail： Keep 通过模版测试但不通过深度测试的像素不改变模版缓冲值
4. 此处构造了一层对水体的遮罩【HideWaterMask】,首先符合的这层物体自身不渲染，但进行模版测试，在【HideWaterMask】所在的位置写入模版缓冲值 5 标记隐藏水的区域。

### 【CharacterUnobstructedTransparents】 Feature

1. 在渲染完不透明队列的最后面，正常对角色的半透明部分进行一次渲染。
2. 注意最开始的渲染 Filtering 中没有任何对【Transparent】【Characters】部分的渲染勾选。

### 【Water】 Feature

1. 在渲染半透明物体之前进行【Water】渲染。
2. Depth:
   - 只做测试 不写入深度。小于等于深度值的水体进行渲染
3. Stencil:
   - 模版测试 跟之前【HideWaterMask】记录的 5 的值进行比较，不等于 5 的像素点才进行渲染。同时不改变模版缓冲的值

### 【ObstructedTransparentFiller】 Feature

1. 在半透明物体渲染完成之后 对【Opaque】【OccludedTransparents】物体进行一次渲染。类似角色被遮挡的像素渲染。
