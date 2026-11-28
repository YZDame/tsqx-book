# TSQX 语法与几何函数完全参考

> 本文以 [vEnhance/tsqx](https://github.com/vEnhance/tsqx) 解析器源码（`tsqx/__main__.py`）以及本仓库实测环境（PyPI `tsqx` + Asymptote 2.87）为事实依据，与 [TSQX 中文完全手册](../examples/tsqx_example.pdf) 同步维护。

## 1. 输入与编译链路

```text
figure.tsqx  --[tsqx -p -s 7.5cm]-->  figure.asy  --[asy -autoimport ldgeo -u classic -f pdf]-->  figure.pdf
```

常用命令行参数：

| 参数 | 作用 | 典型场景 |
| --- | --- | --- |
| `-p` / `--pre` | 添加独立运行所需的 Asymptote 前导代码（含 `import geometry;`、`foot`、`CP`、`IP`、`OP`、`Line` 与 `size(...)`） | 独立编译 `.tsqx -> .asy -> .pdf` 时**必加** |
| `-s <尺寸>` / `--size <尺寸>` | 设置前导代码中的 `size(...)`，默认 `8cm` | 按 1:1 物理宽度控制插图字号，例如 `-s 7.5cm` |
| `-b` / `--soft-label` | 默认 `=` 仅标注字母、不画圆点；`;=` 改为带圆点标注 | 模仿传统几何教科书无黑点风格 |
| `-t` / `--terse` | 省略输出 `.asy` 末尾的来源注释块 | 批量构建或精简输出 |

---

## 2. 四类语句语法

### 2.1 点声明与修饰符

```text
点名 [标签方向] [点/标修饰]= 表达式
```

点修饰符对照（常规模式，未加 `-b`）：

| 写法 | 画圆点 | 画标签 | 常见用途 |
| --- | :---: | :---: | --- |
| `=` 或 `dl=` | 是 | 是 | 默认顶点与主要交点 |
| `:=` | 否 | 否 | 仅参与计算的辅助点 |
| `.=` 或 `d=` | 是 | 否 | 标记圆心、辅助点位而不贴字 |
| `;=` 或 `l=` | 否 | 是 | 仅贴字母、不压黑点（如垂足、中点） |

标签方向支持 4 种写法：

| 写法 | 生成的 Asymptote 方向 | 说明 |
| --- | --- | --- |
| 省略 | `dir(P)` | 沿原点到 `P` 的射线外推；对单位圆上的点较准确，内部点建议显式指定 |
| 方位词：`N`、`NE`、`SW` 等 | `plain.NE` | 8 方位简写，须匹配 `N?S?E?W?` |
| 整数度数：`45`、`270` | `dir(45)` | 逆时针角度（度），最常用 |
| 线性偏移：`2S3E`、`1N2W` | `2*plain.S+3*plain.E` | 远离密集线段 |
| 极径偏移：`1.5R30` | `1.5*dir(30)` | 沿 `30°` 拉远 1.5 倍距离 |

特殊点名自动转义：

- 撇号：`P'` 转为变量 `P_prime`，标签渲染为 `$P'$`。
- 星号：`P&` 转为变量 `P_asterisk`，标签渲染为 `$P^{\ast}$`。
- 下标：`A_1`、`M_a` 保留原名，标签渲染为 `$A_1$`、`$M_a$`。

### 2.2 绘制语句

```text
几何路径或曲线 [/ 填充笔 / 描边笔]
几何路径或曲线 [/ 描边笔]
```

- **两个 `/`** 编译为 `filldraw(exp, fill, outline);`；第二个 `/` 后留空时 `outline` 自动取 `defaultpen`。
- **一个 `/`** 编译为 `draw(exp, outline);`。
- **省略 `/`** 编译为 `draw(exp);`，仅可绘制默认黑线，**不能直接跟颜色或线型**。
- 填充笔中的纯小数（如 `0.12`）自动翻译为 `opacity(0.12)`，相邻令牌以 `+` 连接。

### 2.3 快捷宏

| 宏 | 展开结果 | 说明 |
| --- | --- | --- |
| `~triangle A B C` | `A = dir 110`、`B = dir 210`、`C = dir 330` | 仅定义三点，**不单独设置图幅尺寸** |
| `~regular A B C D E` | 从 `90°` 起按 `360°/n` 等分单位圆 | 任意正 `n` 边形 |

### 2.4 注释与原生 Asymptote 逃生舱

- `#` 起至行尾为注释，转译为 `// ...`。
- 行首 `!` 将整行原样输出至 `.asy`，用于声明 `!path`、自定义函数、数值求解或 `!check_*` 几何断言。

---

## 3. 内置函数与官方 Wiki 差异实测表

上游 `vEnhance/tsqx` Wiki 沿用了原作者本机的 `olympiad.asy` + `cse5.asy`，而标准安装下 `tsqx -p` 使用的是 `import geometry;` 外加 8 个兼容函数。以下按实测结果明确区分：

### 3.1 `tsqx -p` 默认直接可用

| 类别 | 函数 | 说明 |
| --- | --- | --- |
| 点构造 | `dir a` | 单位圆上角度 `a`（度）的点 |
| | `midpoint A--B` | 线段/路径中点 |
| | `foot P A B` | 点 `P` 到直线 `AB` 的垂足 |
| | `extension A B C D` | 直线 `AB` 与直线 `CD` 的交点 |
| | `circumcenter A B C` | 外心 |
| | `orthocenter A B C` | 垂心 |
| | `incenter A B C` | 内心 |
| | `centroid A B C` | 重心 |
| | `excenter A B C` | 旁心（`import geometry` 自带，Wiki 未列） |
| | `IP p q` / `OP p q` | 两路径的第 0 / 第 1 个交点（**顺序不保证对应“另一交点”**） |
| 变换 | `(rotate a O)(P)` | 绕 `O` 逆时针旋转 `a` 度 |
| | `(reflect A B)(P)` | 关于直线 `AB` 反射 |
| | `(scale k O)(P)` | 以 `O` 为中心缩放 `k` 倍 |
| | `(shift v)(P)` | 平移向量 `v` |
| 路径 | `circumcircle A B C` / `incircle A B C` | 外接圆 / 内切圆 |
| | `CP O A` / `CR O r` / `unitcircle` | 过 `A` 的圆 / 半径 `r` 的圆 / 单位圆 |
| | `Line A B` | 过 `A, B` 两端各延展 0.6 倍的直线段 |
| 算术 | `plus` / `minus` / `mult` / `divide` | 前缀算术运算符，规避中缀空格歧义 |

### 3.2 官方 Wiki 列出但标准 `tsqx -p` 缺失的函数（由 `ldgeo.asy` 补齐）

| Wiki 提及的写法 | 标准 `tsqx -p` 表现 | `ldgeo.asy` 推荐替代 |
| --- | --- | --- |
| `anglemark A B C` | 报错：`no matching variable` | `anglemark A B C / mark` |
| `rightanglemark A B C` | 报错：`no matching variable` | `rightanglemark A B C / mark` |
| `pathticks (A--B) 2` | 报错：`no matching variable` | `pathtick A B 2 / mark` |
| `bisectorpoint A B C` | 报错：`no matching variable` | `bisectorPt A B C` / `extBisectorPt A B C` |
| `tangent P c 1` | 报错：`no matching function` | `tangentPtR P O r 1` 或 `tangentPt P O A 1` |
| `IP p q n`（3 参数） | 报错：签名不匹配 | `other c l X` 或 `otherPt c A B` |
| `intersect p q` | 返回 `real[]` 而非 `pair` | 用 `extension`（直线）或 `otherPt`（圆线） |
| `arc A B a1 a2` | 签名不匹配（`arc(center, r, a1, a2)` 需传半径） | `!draw(arc(O, abs(A-O), a1, a2), circ);` |

---

## 4. 四大解析器词法陷阱

1. **绘制样式漏写 `/`**
   - 错误：`B--F dashed blue`（生成非法的 `draw(B--F, dashed, blue);`）
   - 正确：`B--F / dashed blue`
2. **`/` 后写入带括号的 Asymptote 笔刷函数**
   - 错误：`A--B / gray(0.5)`、`A--B / linewidth(1.5) red`、`A--B / rgb("1B4F9C")`
   - 原因：`tokenize()` 会把 `(`、`)` 拆成独立令牌，再用 `+` 拼接成非法代码 `gray(+0.5+)`。
   - 正确：改用无括号语义变量名（`aux`、`key`、`circ` 等，见 [样式主题指南](theme-guide.md)），或以 `!draw(...)` 原生写入。
3. **中缀算术运算符带空格**
   - 错误：`P = 2*A - B`（生成 `2*A(-B)`）
   - 正确：`P = 2*A-B` 或 `P = minus (2*A) B`
4. **括号表达式直接以 `--` 相连**
   - 错误：`(foot A B C)--(foot B C A)`（解析器生成非法的 `foot(...)*-(-)*foot(...)`）
   - 正确：先赋给命名点 `D = foot A B C`、`E = foot B C A`，再写 `D--E`。
