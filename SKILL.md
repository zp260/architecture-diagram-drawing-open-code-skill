---
name: |-
  [EN] I CAN do architecture diagram drawing.
  [CN] 我可以绘制架构图.
description: "[EN] I CAN do architecture diagram drawing.Generate and create .svg .drawio format files.[CN] 我可以绘制架构图. 生成及创建 .svg .drawio 格式文件"
---
# 需要确认的系统环境
1. 需要当前环境安装配置drawio mcp。包含:mcp-app-server和mcp-tool-server。关于drawio mcp的github仓库介绍：[drawio-mcp-readme](https://github.com/jgraph/drawio-mcp/blob/main/README.md)。[draw-mcp-github-clone-address](https://github.com/jgraph/drawio-mcp.git).
2. 确定mcp-app-server访问地址可用。
3. [opencode配置mcp方法](https://opencode.ai/docs/zh-cn/mcp-servers/)。
4. 确定mcp-app-server与mcp-tool-server已启用，因为此技能要用到这两个mcp。

# 需要的技能
| 技能                          | 与 SVG 的关联                      |
| --------------------------- | ------------------------------ |
| `frontend-ui-ux`            | 前端开发中经常处理 SVG，可以用代码操作 SVG      |
| `canvas-design`             | 创建视觉设计（输出 .png/.pdf），不直接处理 SVG |
| `drawio-app_create_diagram` | 创建 draw.io 图表，支持 XML 格式输出      |

# 配色规范参考
> 没有体现的层级，自己决定确定符合配色规范的色值。

| 层级    | 颜色  | 色值                | 说明        |
| ----- | --- | ----------------- | --------- |
| 用户层   | 蓝色系 | #E3F2FD / #1565C0 | 代表用户交互与展示 |
| 前端应用层 | 绿色系 | #E8F5E9 / #2E7D32 | 代表界面与业务逻辑 |
| 后端服务层 | 橙色系 | #FFF3E0 / #EF6C00 | 核心业务处理    |
| 数据层   | 紫色系 | #F3E5F5 / #7B1FA2 | 数据持久化存储   |
| 设备层   | 灰色系 | #ECEFF1 / #455A64 | 基础设施支撑    |


# 架构图设计规范
- **层级分明**：使用 Material Design 色彩系统，层次感强.
- **专业规范**：遵循 C4 Model 架构可视化原则.
- **可扩展性**：drawio 源文件便于进一步美化调整.
- 遵循软件架构规范，使用专业的层级配色方案。
- 不同的层级需要用不同的颜色进行区分，以增强架构图显示效果，做到美观大气。

# 你能做的

- 用代码读取、解析、修改 SVG 文件（XML 操作）
- 在前端代码中集成 SVG
- 通过 draw.io mcp创建图表
- 创建及读取.drawio文件内容转换为svg图片，将svg图片存储到项目的svg目录下。

## **你不能做的：**

- 专业图形设计软件那样的可视化 SVG 编辑
- 复杂的矢量图形转换/优化
- 系统没有 draw.io 工具可用

# 文件的存储
- svg文件存储到项目的svg目录下。
- drawio文件存储到项目的drawio目录下。
- 当前没有相关目录，则创建。
# drawio-to-svg脚本
1. # drawio-to-svg脚本可以根据.drawio文件转换为.svg文件。
2. For advanced features, JavaScript libraries, and detailed examples, see REFERENCE.md. 

# 架构图生成的流程
1. 使用drawio mcp 根据需求和规范生成.drawio格式的架构图文件。
2. 根据"drawio-to-svg脚本"可以将 .drawio 文件转换为 SVG.
3. 确定脚本可以正常转换，查看转换结果。
4. 输出转换结果报告。
5. 其他可选场景，将svg架构图文件插入到docx,ppt,excel等文件中。