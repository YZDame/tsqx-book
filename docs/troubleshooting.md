# TSQX 编译与故障排查

## 环境与入口

**TSQX** 负责将 `.tsqx` 翻译为 `.asy`；**Asymptote** 将 `.asy` 编译为 PDF；**Poppler** 的 `pdftocairo` 只在某些 VS Code 预览/导出场景使用，并非 TSQX 到 PDF 的必备前提。

```sh
python3 -m pip install tsqx
tsqx --help
asy --version
pdftocairo -v # 需要 VS Code 的实时预览 / SVG、PNG 导出时再检查
```

`pip install tsqx` 安装的是上游 Python 包；本仓库随附的 `tsqx/` 是用于教学对照的源码副本，不保证与最新上游一致。

## 分阶段定位错误

先单独运行：

```sh
tsqx -p < example.tsqx > example.asy
asy -f pdf -o example example.asy
```

| 现象 | 定位方式 | 常见原因 |
| --- | --- | --- |
| 找不到 `tsqx` | 检查当前 Python 环境及 PATH | `pip` 装在其他解释器环境 |
| 找不到 `asy` | 使用 `asy --version` | 尚未安装 Asymptote 或 PATH 未配置 |
| 生成的 ASY 不可编译 | 直接打开 `.asy` 查看 Asymptote 报错行 | 类型不匹配、路径无交点、表达式不合法 |
| PDF 可输出但几何不对 | 对照题目约束与各构造定义 | 几何关系误解或选择了另一交点 |
| VS Code 预览不可用 | 运行 **TSQX: Diagnose Toolchain** | 缺少 `pdftocairo` 或路径错误 |
| LaTeX 中内嵌绘图失败 | 先验证单独的 `.asy` | 缺少前导定义、`-shell-escape` 或构建步骤 |

注意：Asymptote 行号指向**生成的 ASY**，不一定能无歧义映射到原始 TSQX 行。VS Code 扩展会保留生成文件供检查。

## `-p` 的正确含义

`-p` 会**添加**前导代码；如果独立输出 PDF，一般使用 `tsqx -p`。如果需要把 TSQX 片段嵌入已有 `asy` 环境，通常不使用 `-p`，但必须自行提供被引用的函数和包。不要把这两种场景混为一谈。

## 维护边界与回归

1. 以 [上游 vEnhance/tsqx](https://github.com/vEnhance/tsqx) 为语言实现来源。
2. 扩展入口为 [tsqx-compiler](https://github.com/YZDame/tsqx-compiler)；AI 生成示例参照 [tsqx-gen-skill](https://github.com/YZDame/tsqx-gen-skill)。
3. 改动函数签名或语法说明时，至少编译 `examples/figures/` 下相关 TSQX 示例，并人工检查约束和标签。
4. 编译通过只能证明工具链正常；**不能替代几何语义核验**。

返回：[语法参考](syntax-reference.md) · [仓库首页](../README.md)。
