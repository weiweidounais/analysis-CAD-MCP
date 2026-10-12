# analysis-CAD-MCP

面向 AutoCAD 的图纸分析与碰撞点位标注 Skill。通过 `best-cad-mcp` 检查构件、管线、零件之间的干涉与净距问题，将已提供或经验证的点位转成编号标记、引线说明和可追溯的问题报告。

仓库名称为 **analysis-CAD-MCP**；Skill 标识为 **`analysis-cad-mcp`**，显式调用方式是 `$analysis-cad-mcp`。

## Preview

### 碰撞点位标注与文字说明

![构件与管线碰撞点位标注示意](assets/collision-annotation-preview.png)

**CL-001** 展示管线与构件的干涉区域圈注：编号与涉及对象、位置、证据、问题说明和建议相互关联。**CL-002** 展示净距问题的描述方式：同时列出实测值与项目要求，避免只标一个没有依据的“碰撞点”。

> 图片是本项目制作的合成示意，图示、坐标和数值均为展示用途，不是 AutoCAD 实测结果。20 mm 仅为假设的示例项目要求，不是通用净距标准。

### 工作流与逐点问题报告

![碰撞标注工作流和问题报告示意](assets/workflow-report-preview.png)

工作流依次组织输入与预检、几何核验、编号标注、复查交付。报告将**确认问题、候选问题和未知数据分别记录**，以同一编号关联图上标记、对象句柄、文字说明、结构化数据和局部视图。示意中的“已复查”等状态只展示报告格式，并非真实执行记录。

[查看完整 Preview 说明](PREVIEW.md)。

## 主要功能

- **干涉与净距审查**：在指定对象或区域内利用实际可用的 MCP 几何工具核验，区分实体干涉、投影交叉与候选区域。
- **已知点位标注**：可直接接收坐标、句柄、用户清单或截图标记，不将每个任务扩大为全图扫描。
- **逐点编号与引线**：使用 `CL-001` 等稳定编号，关联点位、构件/管线、图层与说明；三维点保留真实 Z 坐标。
- **文字描述与报告**：记录位置、实际/要求净距、测量依据、问题、可能影响、建议与待核实项，组织 Markdown 和 JSON 输出。
- **继承装配图流程**：保留原有图纸分析、尺寸与约束验证、装配/剖视/爆炸图、BOM、图片转 CAD、CADPlan 和导出工作流。

## 安装到项目

在目标项目根目录执行（目标文件夹应不存在）：

```bash
mkdir -p .agents/skills
git clone --depth 1 https://github.com/weiweidounais/analysis-CAD-MCP.git .agents/skills/analysis-cad-mcp
```

如果仓库是私有的，先完成 GitHub 身份验证。Skill 可以在下一轮消息中发现；未显示时重新打开 Codex。

安装 Skill 不会自动安装或连接 MCP 服务。实际 CAD 操作需要另行配置 [best-cad-mcp](https://github.com/LokmenoWer/best-cad-mcp)；其运行环境为 **Windows + 已安装并授权的 AutoCAD + Python 3.11 或更新版本**。服务器和 AutoCAD 应使用同一 Windows 账户。

## 触发方式

显式调用：

```text
$analysis-cad-mcp
把这些管线碰撞点编号，用引线逐点说明，并生成问题清单。
```

也可以用自然语言触发，例如：

- “检查这些构件之间的干涉与净距不足，标注已确认问题。”
- “把这份零件干涉清单的坐标标到装配图，并逐点描述。”
- “只输出碰撞点位报告，不修改 DWG。”

尺寸文字和引线之间的排版重叠不属于本 Skill 新增的物理碰撞审查范围。

## 输出与运行边界

碰撞流程可以组织以下交付物，是否生成取决于任务要求、已授权操作与实际工具能力：

| 交付物 | 内容 |
|---|---|
| `collision-report.md` | 检查范围、判据、逐点说明、证据和未完成项 |
| `collision-points.json` | 点位、对象、坐标、测量、描述及检测/标注状态 |
| DWG 审查标记与 MLEADER | 编号、点/区域标记、简短说明及新增实体句柄 |
| PNG / PDF 局部视图 | 定位证据与标注可读性复查 |

这是 **Skill 指令与参考资料包**，没有新增一个独立的碰撞求解器或 MCP 服务。包围盒重叠、截图交叉或 `validate_geometry` 的成功状态不能单独证明真实三维干涉；不能获得可靠几何或测量时应报告未知，而非编造结果。

用户请求创建审查标记并不包含移动原构件、消除碰撞或覆盖保存 DWG 的授权。图上标注完成与问题已经解决是两种不同状态。

当前发布内容通过了 Skill 格式与文件引用检查；**尚未完成连接 AutoCAD 的端到端功能验证**。Preview 不用于制造或施工。

## 文件与修改入口

```text
analysis-CAD-MCP/
├── SKILL.md
├── agents/openai.yaml
├── references/
│   ├── collision-annotation.md
│   ├── assembly-drawing-requirements.md
│   └── assembly/
│       ├── index.md
│       └── standards/generic-mechanical.md
├── assets/
│   ├── collision-annotation-preview.png
│   └── workflow-report-preview.png
├── PREVIEW.md
├── README.md
└── LICENSE
```

- [SKILL.md](SKILL.md)：触发描述、工作流路由和主要约束。
- [碰撞点位规则](references/collision-annotation.md)：点位字段、编号、图层、标注方法、说明格式、复查和报告要求。
- [UI 配置](agents/openai.yaml)：显示名称、默认调用提示和 MCP 依赖。

具体项目的对象范围、净距阈值与标准在执行时读取，不硬编码为通用判据。

## 来源与许可

本 Skill 派生自 [LokmenoWer/best-cad-mcp](https://github.com/LokmenoWer/best-cad-mcp) 的 `draw-assembly-diagrams`，基于上游提交 `9f6e3a91c54026e9a253ffdffb869ae63f6af2a8`，保留原装配图参考模块及 MIT 许可。当前版本增加了物理碰撞点位与净距审查的标注/描述流程、中文 Preview 和示意图片。

许可见 [LICENSE](LICENSE)，原作者版权声明保留。Preview 图片为本项目原创合成示意，随本项目以 MIT 许可发布；图片不暗示已经完成真实 CAD 检测。
