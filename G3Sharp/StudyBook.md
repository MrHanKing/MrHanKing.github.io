# 符号距离场(Signed Distance Fields)

## 理解

* 采用数学函数（方程）描述空间内的点到3D几何体表面的距离。标准约定为几何体内部为负，外部为正。
* 符号距离场只定义了方程，并没有可视化即描述具体的模型（Mesh）。描述具体的模型需要通过MarchingCubes算法所得，大致做法是根据距离函数在距离场空间内进行采样，获得距离场网格体（例如MeshSignedDistanceGrid），然后进行插值（例如DenseGridTrilinearImplicit）获得不同精度的距离场（空间标量场）。
* 获得距离场空间标量场后就可以通过给定定值距离获得模型形状。

# 隐式表面建模(Implicit Surface Modeling) 

* 隐式表面建模可以通过创建单独的单个网格的符号距离场，然后可以很方便的通过数学运算把它们组合起来从而建立具体模型。
* F_XX代表距离场函数

## 简单运算

* 合并A模型和B模型: 
  * F_Union = Mathf.Min(F_A, F_B);
  * 解释: F_A或F_B 只要一个是负数就说明距离在结果模型内部，都是正才在模型外部，为0时则在A的表面或B的表面上。
* A模型和B模型的重叠部分
  * F_Intersection = Mathf.Max(F_A, F_B);
  * 解释: F_A或F_B 只要任意个为正就说面在结果模型外部。只有都为负是才在模型内部。
* 从A模型裁剪掉B模型重叠的部分，即从A模型中减掉B模型
  * F_Difference = Mathf.Max(F_A, - F_B);
  * 解释: - F_B 反转了F_B模型的内外部，然后根据重叠部分的定义求了F_B外部和F_A内部的重叠部分。就是从A模型中减去B模型的结果 

## 曲面偏移

* F_XX(X_Position) = 0 可以获得模型表面的空间描述。那么给定定值偏移量offset，就可以获得对应模型外扩或者内收束的模型。
* 可以使用的G3Sharp类例如ImplicitOffset3d等。

## 平滑模型 

* 让简单运算之后的模型在进行offset变化时 边缘交接处的变化变的平滑。
* 平滑模型并非精确计算，整个空间都会发生平滑变化而不仅仅是交接处。
* F_Smoothunion = 2 / 3 * (F_A + F_B - Mathf.Sqrt(F_A * F_A + F_B * F_B - F_A * F_B ))

## 混合运算

* 解决2个模型合并时边缘的平滑过度问题
* W_Blend 混合系数
* F_Blend = F_Smoothunion - W_Blend / (1 + F_A ^ 2 / W_A ^ 2 + F_B ^ 2 / W_B ^ 2);

# 卷绕数(Winding Numbers)

# 数据结构相关
