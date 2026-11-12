# TSQX 语法与几何构造参考

> 本页是 **语法参考**，与入门 PDF 的逐步教程分工不同。示例以 [vEnhance/tsqx](https://github.com/vEnhance/tsqx) 的 Python 解析器及本仓库的 `tsqx/__main__.py` 为依据；上游更新后应重新核对。请先阅读 [入门教程](../examples/tsqx_example.pdf)。

## 一、输入与编译模型

TSQX 是面向欧氏几何图形的 Asymptote 预处理语言，不是完整的动态几何求解器。它翻译构造与绘制语句；**图能编译不等于题目条件被正确表达**。

```text
figure.tsqx  --tsqx -p-->  figure.asy  --asy -f pdf-->  figure.pdf
```

可独立运行的最小命令：

```sh
tsqx -p < figure.tsqx > figure.asy
asy -f pdf -o figure figure.asy
```

`-p` / `--pre` **会添加**包含 `size(...)`、Asymptote `geometry` 等定义的前导代码；`-s 10cm` 可调整使用该前导代码时的大小。`-t` 会隐藏输出末尾的原始 TSQX 注释。嵌入已有完整 Asymptote 环境的片段通常不加 `-p`，但须确保环境自行定义了所需函数。

## 二、点、标签与构造

```tsqx
~triangle A B C
D = midpoint B--C
H NW = orthocenter A B C
I 45 = incenter A B C
A--B--C--cycle
A--H / gray
```

- `~triangle A B C` 创建固定方向的三个初始点（110°、210°、330°），**不会独立设置图形尺寸**。
- `~regular A B C D` 生成单位圆上的等间隔顶点。
- `P = expression` 定义一点；`P NW = ...`、`P 45 = ...` 指定标注方向（方位或角度）。
- 名称可以包含撇号，例如 `P'`，编译器将其转换为可用的 Asymptote 标识符。
- 点后的选项影响绘制与标注：`=` 默认带点和标签；`:=` 两者都不显示；`.=` 仅点；`;=` 仅标签。在 `--soft-label` 模式下，默认标注行为会变化。
- `#` 引入注释；以 `!` 开头的整行会原样注入 Asymptote，适合 TSQX 难以表达的高级构造。

注意：`D = midpoint B--C` 中的 `B--C` 是一条几何路径，参数并非简单的英文句子。

## 三、常见几何函数

| 写法 | 含义 | 需要留意 |
| --- | --- | --- |
| `incenter A B C` | 内心 | 顶点不能退化共线 |
| `circumcenter A B C` | 外心 | 退化三角形不可用 |
| `orthocenter A B C` | 垂心 | 可能位于三角形外 |
| `centroid A B C` | 重心 | 顶点顺序通常不影响结果 |
| `foot P A B` | P 到直线 AB 的垂足 | 垂足可能在线段外 |
| `midpoint A--B` | AB 中点 | 使用路径表达式 |
| `extension A B C D` | 直线 AB 与 CD 的交点 | 平行情况需避免 |
| `incircle A B C` | 内切圆 | 返回的是可绘制对象 |
| `circumcircle A B C` | 外接圆 | 返回的是可绘制对象 |
| `CP O A` | 以 O 为圆心经过 A 的圆 | O 与 A 不应重合 |
| `CR O r` | 圆心 O、半径 r 的圆 | 半径需要为正 |
| `IP p q`、`OP p q` | 两条路径的交点 | 两点顺序与路径方向有关，需目视验证 |
| `Line A B` | 过 A、B 的完整直线 | A、B 不能重合 |

这里列举的是常用词法与函数；Asymptote 本身能表达的几何运算更广，不应把本表当作完整标准库。遇到版本差异，请以 [上游解析器](https://github.com/vEnhance/tsqx) 和生成的 `.asy` 为准。

## 四、路径与样式

```tsqx
A--B--C--cycle / 0.10 lightcyan / blue
A--D / gray
B--H / red dashed
circumcircle A B C / blue
```

- `A--B` 为直线段，`--cycle` 将路径闭合。
- 有两个分隔符的写法是 `对象 / 填充笔 / 描边笔`；仅一个分隔符时是描边笔。
- `gray`、`red`、`dashed`、`opacity(...)` 等来自 Asymptote 的样式与笔对象。
- 推荐从**构造点**、**验证几何关系**到**决定标签位置**、**最后设置画线样式**，不要用视觉近似代替数学约束。

## 五、演示：三角形的高与垂足

保存为 `altitude.tsqx`：

```tsqx
~triangle A B C
D 270 = foot A B C
A--B--C--cycle / 0.08 lightcyan / black
A--D / gray dashed
```

检查清单：

1. `D` 确实落在直线 `BC` 上。
2. `AD` 与 `BC` 垂直，而非仅看起来垂直。
3. `D` 的标签在输出图中可辨认，线段没有被遮挡。
4. 如需插入 LaTeX，优先使用独立 PDF：`\\includegraphics{altitude.pdf}`。

**编译成功只验证语法及工具链，不替代数学正确性检查。**

继续阅读：[故障排查](troubleshooting.md) · [VS Code 扩展](https://github.com/YZDame/tsqx-compiler) · [AI 生成 Skill](https://github.com/YZDame/tsqx-gen-skill)。
