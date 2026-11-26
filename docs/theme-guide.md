# TSQX 语义化样式与主题切换指南

本指南说明如何在 `.tsqx` 源码中用**语义角色**代替零散的颜色名，实现类似 CSS 主题表的“改一个参数、整篇讲义全部插图统一换肤”。

## 1. 为什么采用“语义令牌 + 编译期注入”

1. **视觉层级稳定**：竞赛平面几何图只需要区分 5 层信息（基底填色、已知骨架、主/次圆系、辅助虚线、核心结论线），而不是十几种随机高饱和色。
2. **绕开 TSQX 词法限制**：原版 `tsqx` 不允许在 `/` 后书写 `rgb("...")`、`gray(0.5)`、`linewidth(1.2)` 等带括号函数；而预先定义好的无括号笔变量名（如 `given`、`aux`、`key`、`fill1`）完全兼容原版 `tsqx`。
3. **零样板源码**：通过 Asymptote 的 `-autoimport ldgeo -u <theme>` 在编译期注入样式，`.tsqx` 源文件无需写 `!import`，仍然保持最短可读形态；对外分享 `.asy` 时使用 `--standalone` 即可一键将主题与辅助函数内联进单个文件。

## 2. 语义笔令牌表（`templates/tsqx/ldgeo.asy`）

| 令牌 | 语义角色 | 典型几何对象 |
| --- | --- | --- |
| `given` | 已知骨架 | 主三角形、题设多边形边 |
| `aux` | 主辅助线（虚线） | 高线、中线、角平分线、延长线 |
| `aux2` | 次辅助线（点线） | 等距连线、垂足连线、参考径 |
| `key` | 第一核心结论线（加粗强调） | 待证共线、欧拉线、根轴、极线 |
| `accent` | 第二强调线 | 切线、垂足三角形轮廓、对照路径 |
| `circ` | 主圆 | 外接圆、题设主圆 |
| `circ2` | 次圆 | 内切圆、九点圆、辅助圆 |
| `mark` | 几何标记 | 直角标 `rightanglemark`、角弧 `anglemark`、等长刻度 `pathtick` |
| `fill1` | 主基底淡填色 | 原三角形、主圆淡底 |
| `fill2` | 次区域淡填色 | 内切圆、垂足三角形、第二圆淡底 |
| `fillkey` | 重点区域淡填色 | 核心目标区域 |

## 3. 五套预设主题

| 主题名 | 设计场景 | 视觉特征 |
| --- | --- | --- |
| `classic`（默认） | 日常竞赛讲义、电子 PDF | 深炭黑骨架 + 学院蓝主圆 + 松柏绿次圆 + 绯红结论线 |
| `print` | 黑白激光打印、校内阶段考试（`quizzes/`） | 纯黑/灰阶 + 线宽与虚实区分 + 实色淡灰填充（规避打印机 alpha 透明黑框） |
| `flexoki` | 护眼暖纸感阅读 | 低饱和墨纸色（Flexoki 600 阶色板） |
| `colormath` | ColorMath / 北师大风格长书稿 | 直接对齐 `templates/colormath/ColorMath.cls`（`BNUBlue`、`Orange`、`SYUSGreen`、`THUPurple`） |
| `slides` | 16:9 投影课件、希沃白板 | 对齐 `templates/exercise-slides/exercise-slides.sty`，字号 11pt、线宽提升约 40% |

## 4. 三种切换方式

### 方式 A：批量构建脚本（整篇讲义一键换肤）

```bash
# 讲义用 ColorMath 主题
python3 scripts/build_tsqx.py lectures/2026夏/202608/某个专题/ --theme colormath --scale standard

# 试卷用黑白印刷主题
python3 scripts/build_tsqx.py quizzes/competition/某场竞赛/figures/ --theme print --scale compact

# 导出独立可分享的 .asy（内联 ldgeo.asy 与所选主题）
python3 scripts/build_tsqx.py figure.tsqx --theme classic --standalone --check-png
```

### 方式 B：项目级声明（零参数自动匹配）

在讲义项目的 `project.yaml` 中加入一行：

```yaml
tsqx_theme: colormath
```

或在图形目录放置 `.tsqxrc.json`：

```json
{
  "theme": "print"
}
```

此后直接运行 `python3 scripts/build_tsqx.py <项目目录>` 或在 VS Code 中编译，都会自动应用对应主题。

### 方式 C：原生命令行调用

```bash
tsqx -p -s 7.5cm < fig.tsqx > fig.asy
ASYMPTOTE_DIR=/path/to/templates/tsqx asy -autoimport ldgeo -u print -f pdf -o fig fig.asy
```

## 5. 物理尺度与字号 1:1 规范

Asymptote 默认标签字号会随 `\includegraphics[width=...]` 的缩放比例发生二次缩放。推荐按三档固定物理宽度出图，并在 LaTeX 中按 1:1 宽度嵌入（不写随意百分比缩放）：

| 档位（`--scale`） | `tsqx -s` 参数 | LaTeX 推荐写法 | 适用场景 |
| --- | --- | --- | --- |
| `compact` | `5.5cm` | `\includegraphics{fig.pdf}`（或 `width=5.5cm`） | 双栏图、并排对照图、小题配图 |
| `standard` | `7.5cm` | `\includegraphics{fig.pdf}`（或 `width=7.5cm`） | 正文单栏标准例题配图 |
| `wide` | `11.5cm` | `\includegraphics{fig.pdf}`（或 `width=11.5cm`） | 复杂多圆/根轴/分步大图 |
