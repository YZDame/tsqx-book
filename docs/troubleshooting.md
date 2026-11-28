# TSQX 编译与故障排查手册

## 1. 环境自检

```bash
python3 -m pip install tsqx
tsqx --help
asy --version
pdftocairo -v   # VS Code 实时 SVG 预览与 PNG 导出需要
pdftoppm -v     # CLI 视觉核验 --check-png 需要
```

如果 `asy --version` 低于 `2.69`，请先升级 TeX Live 或 Asymptote；如果遇到 `tsqx: command not found`，确认 `pip` 与运行时的 `python3` 是否属于同一环境，或改用 `python3 -m tsqx`。

## 2. 分阶段定位常见错误

先拆分两步运行，确认错误发生在 `tsqx` 翻译层还是 `asy` 渲染层：

```bash
tsqx -p -s 7.5cm < figure.tsqx > figure.asy
ASYMPTOTE_DIR=templates/tsqx asy -autoimport ldgeo -u classic -f pdf -o figure figure.asy
```

| 现象 | 常见根因 | 解决方法 |
| --- | --- | --- |
| `draw(A--B, dashed, blue)` 报错 | 绘制样式漏写 `/` 分隔符 | 改为 `A--B / dashed blue` |
| `gray(+0.5+)` 或 `rgb(+"..."+)` 语法错误 | 在 `/` 后写了带括号的 Asymptote 函数 | 改用 `ldgeo` 无括号语义变量（`aux`、`key`）或 `!draw(...)` |
| `pair P = 2*A(-B)` 报错 | 减号两边留了空格 | 写成 `2*A-B` 或 `minus (2*A) B` |
| `foot(...)*-(-)*foot(...)` 报错 | 两个括号表达式直接用 `--` 连接 | 先将两端点赋给变量 `D`、`E`，再写 `D--E` |
| `no matching variable 'anglemark'` | 使用了仅存于 `olympiad.asy` 的 Wiki 函数 | 启用 `ldgeo.asy`（`scripts/build_tsqx.py` 或 `tsqx.theme`） |
| `no matching variable 'given'` | 使用了语义令牌但未注入 `ldgeo.asy` | 编译时加 `-autoimport ldgeo` 与 `ASYMPTOTE_DIR`，或开启 VS Code 的 `tsqx.theme` |
| 交点选到了已知顶点或另一支 | 误用 `OP` 当作“另一交点” | 改用 `ldgeo` 的 `otherPt c A B` 或 `other c l X` |
| `assert FAILED: check_perp failed` | 几何构造不满足题设垂直/共圆/共线 | 恭喜：断言提前拦下了错图，请回查构造顺序与交点选择 |
| 画面被拉得极扁、主三角形缩成一团 | 某条直线延长过长或交点跑飞到远处 | 调整初始顶点角度（避开近平行），或限制辅助线裁剪范围 |
| 图中字母相对正文过小或过大 | `size(...)` 与 `\includegraphics[width=...]` 不一致 | 改用 `--scale compact|standard|wide` 并在 `.tex` 中按 1:1 尺寸嵌入 |
| Linux / CI 下编译手册报字体缺失 | 缺少 macOS 字体 `Songti SC` / `Heiti SC` | 手册已内置跨平台回退，或设置 `FONT_MAP='Songti SC:Noto Serif CJK SC,Heiti SC:Noto Sans CJK SC'` |

## 3. “编译通过”不等于“图画对了”

在把插图放进讲义或试卷之前，坚持三道防线：

1. **工具链通过**：`tsqx` 与 `asy` 均返回 0 且生成非空 PDF。
2. **数值断言通过**：对关键垂直、共线、共圆、等距关系添加 `!check_*` 断言。
3. **视觉核验通过**：通过 VS Code 实时预览或 `--check-png` 渲染 PNG 目检标签是否压线、圆弧是否被裁切、黑白打印主题下线条层次是否清晰。
