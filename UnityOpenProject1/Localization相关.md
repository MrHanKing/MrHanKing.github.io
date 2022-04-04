# Localization

- 看起来是个本地化包，但功能大到超出了预估。所以来好好学习整理下。(PS: 早点发现他能省掉之前的 App 很多多语言工作)
- 首先看支持的功能：
  1. 多语言字符串支持
  2. 多语言资产支持，纹理、模型、音频等(支持表格配置)
  3. 多语言数据(ScriptableObject 数据)和 Csv、GoogleSheets、XLIFF 的互相导出导入以及编辑。这点非常赞，可以省掉很多事情。(对普通列暂未支持 需要自己扩展)

# 扩展正常表格 统一数据配置

## 1

- Localization 系统使用 StringTableCollection 为数据单位来存储整个编辑器和运行时的总数据。 运行时使用对应的 Table 数据。
- 所以第一步先扩展 NormalTableCollection 和 NormalTable
