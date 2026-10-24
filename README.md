# FigForge · 科研图表与可发表级可视化工作台

> 六步流水线：**导入 → 类型推荐 → 期刊模板 → 配色标注 → 导出 → 代码复现**。一次选中期刊即合规，导出无需手动调整。

![形态](https://img.shields.io/badge/形态-单文件%20HTML-1A1D1F) ![依赖](https://img.shields.io/badge/外部依赖-0-1A1D1F) ![许可](https://img.shields.io/badge/license-MIT-green)

- **在线预览**：https://px-pangu.github.io/figforge/
- **仓库地址**：https://github.com/px-pangu/figforge
- **应用文件**：`index.html`（约 117 KB，单文件，零依赖）

---

## 1. 这是什么

面向科研作者的**图表工作台**。它要解决的不是「画不出来」，而是投稿时反复被退回的三类问题：

1. **规范不合** —— Nature / Elsevier / IEEE / ACS 四家对尺寸、字号、线宽、dpi 要求各不相同，每次手动调；
2. **可信度存疑** —— 删点、四舍五入、无参数的平滑拟合，reviewer 一问就穿帮；
3. **无法复现** —— 图能看，但说不清它是用什么参数、什么种子生成的。

FigForge 把四社现行条款写成四套模板，并把**「不许改数据」写成硬约束**贯穿全流程。

### 当前版本的性质

- 单文件纯前端，双击 `index.html` 即可运行，**不含任何外部网络请求**；
- 内置示例数据集，用于呈现六步流程与校验逻辑；
- 统计检验（Welch t 检验、BH / Bonferroni 校正）与色盲 / 灰阶模拟为**规则演示**，未接入统计库；
- 当前版本**不使用 localStorage**，刷新即回到初始状态。

---

## 2. 快速开始

```bash
# 方式一：在线打开
# https://px-pangu.github.io/figforge/

# 方式二：下载 index.html 后双击（断网可用）

# 方式三：克隆后本地起服务
git clone https://github.com/px-pangu/figforge.git
cd figforge
python -m http.server 8000
```

---

## 3. 目录结构

```text
figforge/
├── index.html                          # 应用主体（单文件）
├── docs/
│   └── FigForge_图表流程清单.md          # 原始清单：6 步的交互、算法规则、期刊条款、导出参数与校验项
├── .gitignore
├── LICENSE                             # MIT
└── README.md
```

---

## 4. 六步流水线详解

### 01 数据导入（只读锁定）

- **交互**：拖入 CSV / TSV；预览表逐列显示类型、缺失率、唯一值数，可手动改并锁定。
- **算法**：唯一率 > 95% → ID；非数值水平数 ≤ min(20, √n) → 分类；可解析 → 日期；缺失 > 30% 黄标但**不填补**。
- **期刊条款**：Nature 要求随文提交 source data；禁四舍五入、删点、插补与静默类型转换（SEPT9）。
- **导出**：附原数据副本 + SHA-256、UTF-8、时区与 `parse_log`（列类型、缺失清单）。
- **校验**：行数一致；哈希重算通过；缺失与可疑转换列 100% 列出；`alter_log` 为空。

### 02 类型推荐（3 候选图 + 反面提示）★

- **交互**：三卡并排同数渲染；推荐理由与**常驻不可关闭的反面提示**同屏显示；点选后摘要入图注。
- **算法**：按变量组合打分取前三；n > 2000 自动降密度渲染并公示分箱；n ≤ 10 强制全点叠加。
- **期刊条款**：Nature / Elsevier 劝退以「均值 ± 误差棒」代替分布；须显示全部点或确切 n。
- **迭代点**：本步由「推荐 3 种候选图」升级为「3 候选图 + 每张给出**该图会掩盖什么信息**的反面提示」。
- **导出**：`figure_spec.json` 记录 `chart_id`、`reason`、`caveat`、`bin_size`、`n`、`seed`。
- **校验**：每卡 caveat 非空；条形类默认全点叠加；n ≤ 10 降级为抖动散点；导出须知复述 caveat。

### 03 期刊模板（四社规范库）

- **交互**：四社卡 + 单双栏切换；选中后参数写死，只读显示条款出处。
- **算法**：`px = mm × dpi / 25.4`；字号线宽按印刷尺寸反算校验，超限告警；**模板不被滑杆覆盖**。

| 期刊 | 栏宽 | 字号 | 分辨率 |
|---|---|---|---|
| Nature | 89 / 183 mm | 5–7 pt | ≥ 300 dpi |
| Elsevier | 90 / 140 / 190 mm | 正文 ≈7 pt、上下标 ≥6 pt | 300 / 500 / 1000 dpi |
| IEEE | 3.5 / 7.16 in | 最终印刷 ≥8 pt | ≥ 300 dpi，须灰度可读 |
| ACS | 单栏 ≤240 pt、双栏 300–504 pt | 最小 ≥4.5 pt、线 ≥0.5 pt | 1200 / 600 / 300 dpi |

- **导出**：宽高 mm、dpi、RGB、TrueType 子集嵌入。

### 04 配色与显著性标注

- **交互**：Okabe–Ito / viridis / cividis / 灰度切换；**三型色盲与灰阶模拟**；选两组 + 检验 + 校正出星号。
- **算法**：定性 ≤ 8 色，禁红绿相邻与彩虹；连续色固定 vmin/vmax；Welch t 本地算 p，BH / Bonferroni 校正分级。
- **期刊条款**：ACS 要求文本对比度 ≥ 4.5:1、非文本 ≥ 3:1；Nature 禁红绿对比与彩虹尺，须有符号冗余。
- **导出**：色值十六进制、检验方法、统计量、df、精确 p、每组 n 入图注。
- **校验**：星号与 p 一致；灰度可辨；每组标 n；误差棒定义已写明。

### 05 导出（PDF / SVG / TIFF）

- **交互**：格式 + dpi 滑杆；**「投稿预检」全绿才放行**，未过项禁用导出并列出条目。
- **算法**：矢量路径优先；栅格按印刷尺寸 × dpi 反算像素；字体子集嵌入且**不转曲**。
- **期刊条款**：Nature 禁文字转曲；Elsevier TIFF 用 LZW 无损；分辨率按最终印刷尺寸核算，禁上采样。
- **导出**：文件名 `FigN_期刊_栏宽_dpi`；文件头内嵌 `source_url`、`sha256`、UTC 时间戳、renderer 版本。
- **校验**：链接与时间戳入文件头；dpi 元数据 = 像素 ÷ 尺寸；PDF 文字可复制。

### 06 代码复现（Python / R 双轨）

- **交互**：双栏代码同步显示，可复制下载，附依赖锁与 `sessionInfo` 快照。
- **算法**：由同一份 `figure_spec.json` 渲染，含种子、分箱参数、统计调用与环境版本，参数写死。
- **期刊条款**：Nature / Elsevier 要求图可由脚本重建；拟合平滑在图注与代码双处声明。
- **导出**：`figsize(mm→inch)`、`savefig(dpi, metadata)`、`np.random.seed` / `set.seed`。
- **校验**：重跑图元一致；重跑哈希一致；依赖可一键安装；拟合参数与图注文本一致。

---

## 5. 视觉与体验规范

| 项 | 取值 |
|---|---|
| 图表区底色 | 纯白 `#FFFFFF` |
| 网格线 | `#1A1D1F`，恒定 15% 不透明度 |
| 装饰 | **无渐变、无阴影、无装饰图形** |
| 性能 | 10 万行渲染 ≤ 3s |
| 合规 | 四套模板一次选中即合规，导出零手动调整 |

---

## 6. 硬约束（设计红线）

1. **禁止任何数据篡改** —— 不改值、不删点、不插补、不四舍五入，随机数一律带种子；
2. **默认 Y 轴不截断** —— 手动截断必画断轴符号并在图注写明截断区间；
3. **任何平滑 / 拟合 / 插补必在图注标明方法与参数**；
4. **导出文件必须内嵌原始数据链接与 ISO 8601 时间戳**；
5. **颜色不得单独承载信息** —— 须辅以符号 / 线型 / 灰度校验。

---

## 7. 规范出处

条款取自四社现行作者指南：Nature Research figure guidelines、Elsevier Artwork Instructions、IEEE Author Digital Toolbox、ACS Preparing Graphics。尺寸、字号、线宽、dpi 四项在模板锁定后不随联动滑杆改变。

---

## 8. 隐私与数据

- 数据全程在本机浏览器中处理，**不上传服务器**；
- 导出包内嵌源数据哈希，便于审稿人校验；
- 无账号、无追踪、无第三方请求。

---

## 9. 已知限制

- 原型未接入真实统计库，检验结果为规则演示；
- PDF / SVG / TIFF 的真实编码导出能力在原型中以参数与预检清单呈现；
- 未做多图批量处理。

---

## 10. 本系列的其他仓库

本仓库是「AI 提示词工作生成流 · 10 个应用开发实例」之一，同系列还有：

| # | 仓库 | 主题 |
|---|---|---|
| 01 | [scholarflow](https://github.com/px-pangu/scholarflow) | AI 论文共读与脉络梳理 |
| 02 | [mistake-genome](https://github.com/px-pangu/mistake-genome) | 自适应错题本与薄弱点溯源 |
| 03 | [modeling-war-room](https://github.com/px-pangu/modeling-war-room) | 数学建模竞赛 72 小时协作 |
| 04 | [four-year-roadmap](https://github.com/px-pangu/four-year-roadmap) | 大学生四年成长路径规划 |
| 05 | [concept-leap](https://github.com/px-pangu/concept-leap) | 费曼式深度理解训练 |
| 06 | [figforge](https://github.com/px-pangu/figforge) | 科研图表与可发表级可视化 |
| 07 | [classroom-echo](https://github.com/px-pangu/classroom-echo) | 大班课匿名提问与板书回顾 |
| 08 | [bp-polishing-pod](https://github.com/px-pangu/bp-polishing-pod) | 商业计划书诊断与路演打磨 |
| 09 | [mental-model-museum](https://github.com/px-pangu/mental-model-museum) | 跨学科心智模型迁移学习 |
| 10 | [decision-balance](https://github.com/px-pangu/decision-balance) | 保研 / 考研 / 留学 / 就业四赛道模拟 |
| ★ | [ai-prompt-workflow](https://github.com/px-pangu/ai-prompt-workflow) | 系列总览与 10 个提示词原文 |

---

## 许可证

[MIT](./LICENSE) © 2026 px-pangu
