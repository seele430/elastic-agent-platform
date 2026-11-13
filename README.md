# elastic-agent-platform

从个人项目 **Personal Agent**（单机自用的本地 AI 工作台）中抽离留档的**弹性计算控制面**代码。

> 状态：留档（archive）。按原样抽取，保留原始目录结构与 import 路径（`src.platform.*`），
> 不保证可独立运行——它的执行底座（沙箱服务 / 执行队列 / 节点注册表 / SQLite schema）
> 仍留在原项目中。本仓库的价值在于完整保留了一套**经过生产级测试的设计与实现**。

## 这是什么

一个面向「本地节点 + 沙箱化执行」场景的弹性计算控制面，包含：

| 模块 | 职责 |
|---|---|
| `src/platform/elastic_controller.py` | 弹性控制器：周期采样 → 目标容量 → cordon/drain 节点 → 事件落库 |
| `src/platform/autoscale.py` | 扩缩容策略：基于队列深度与时延的目标容量计算（`desired_capacity`） |
| `src/platform/whatif.py` | What-if 推演：只读模拟策略效果，不改真实状态 |
| `src/platform/trends.py` | 趋势采样：容量/队列时序落库，支撑报表与回放 |
| `src/platform/scale_events.py` | 扩缩事件存储：来源隔离、时间偏移回放 |
| `src/platform/alerts.py` | 告警评估：队列积压 / 节点失联等规则 |

## 设计要点

- **控制面与数据面分离**：控制器只产出「意图」（目标容量、cordon 指令），执行由 runner 完成。
- **一切留痕**：每次扩缩决策、每次模拟、每条告警都落 SQLite，可回放、可审计。
- **模拟不改真值**：what-if 路径只读真实状态做推演，快照接口支持单区块失败不拖垮整页。
- **默认关闭**：控制器由环境变量显式开启（`PERSONAL_AGENT_ELASTIC`），未开启时退化为纯计算口径。

## 目录说明

```
src/platform/    # 控制面六个模块（原项目路径原样保留）
tests/           # 单元测试 5 个 + 集成测试 4 个（含控制台快照/模拟/回放）
scripts/         # demo_elastic_loop.py / autoscale_compare.py
```

## 来源

抽取自 [Personal Agent](https://github.com/seele430/Personal-Agent)（2026-11-13），
原项目回归测试 2 300+ 条全绿。抽取原因：原项目定位收敛为「单人单机」，
弹性伸缩超出其需求边界，独立留档以便日后多节点场景复用。
