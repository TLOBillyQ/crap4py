# crap4py

> 「4py 系列」:[crap4py](http://lzxsvn:3000/qinyuanj/crap4py) · [dry4py](http://lzxsvn:3000/qinyuanj/dry4py) · [mutate4py](http://lzxsvn:3000/qinyuanj/mutate4py)
> 对应「4lua 系列」:[crap4lua](http://lzxsvn:3000/eggy/crap4lua)

`crap4py` 是 CRAP (Change Risk Anti-Patterns) hotspot analyzer for Python。它是「4lua 系列」的 Python 同构实现:
上游 unclebob 的 crap4clj/go/java 规格以 Python 忠实实现(「零本地独有发明」教义;
Python 无上游规格的部分以本文档为规格,对齐不变量逐条列出,偏离注明理由)。

库自包含:`src/crap4py/` 下不硬编码任何宿主专属路径;共享基础件
(common / covdata / astnorm)vendor 在 `_internal/`,不是公共 API。
源自 [ugc-image-tool](http://lzxsvn:3000/qinyuanj/ugc-image-tool) 的 `tools/packages/crap`。

## 安装与入口

```sh
pip install git+http://lzxsvn:3000/qinyuanj/crap4py.git
python -m crap4py --help        # 或直接 crap4py --help
```

在宿主项目根目录运行;宿主解释器优先取 `.venv/bin/python`,否则当前解释器。
覆盖率只来自宿主的 coverage.py 产物(`coverage json`),宿主需装 `coverage`。

## crap(对齐不变量)

- CRAP 公式 `CC² × (1 - cov)³ + CC`;覆盖率缺失时得分为空,显示为 `N/A` ——
  绝不当作 0(JSON 中为 `null`,排序沉底)。
- 风险带 `1-5 low / 5-30 moderate / 30+ high`。
- 五阶段骨架:查找源文件 → 解析函数边界 → 计算圈复杂度 → 覆盖率归因到函数 →
  排序输出。
- 覆盖率仅来自解析生态标准工具的固定产物 — **coverage.py 的 `coverage json`**
  是唯一覆盖率路径(luacov 的 Python 对应)。
- 圈复杂度基于真实 Python AST 计算:基线 1,决策点 +1(if/elif、for/async for、
  while、except handler、with/async with、assert、and/or、三元、推导式、match/case)。
  嵌套函数/类方法体里的决策点算在自己的作用域,不重复计入外层。
- 覆盖率归因分母 = coverage.py 认定的语句行(`executed ∪ missing`),不是 scope 的
  行号跨度:多行语句的续行、docstring 与 `# pragma: no cover` 行都不计入,
  否则分母虚高、CRAP 会被系统性高估。

```sh
python tools/cli.py crap [--top N] [--json] [--gate] [--gate-threshold N] [--source PATH]
```

**有意偏离**:lua 版拆 `collect / report / summary / dry-run` 四命令,此处收敛为
单趟(collect 后直接 report);门禁阈值可配置(默认 5.0,同 lua)。
`--gate`:任一非空 CRAP 超过阈值即退出码 2。

## 布局

```text
src/crap4py/
  cli.py          # 选项走查与退出码
  engine.py       # 核心算法
  _internal/      # vendor 的共享基础件:common / covdata / astnorm
tests/            # pytest;test_internal_* 覆盖 _internal
```

## 测试

```sh
python3 -m venv .venv && .venv/bin/pip install -e '.[test]'
.venv/bin/python -m pytest
```
