# elastic-agent-platform

从 [Unimere](https://github.com/seele430/unimere) 抽离的弹性计算控制面代码，抽取时间为 2026-11-13。

## 仓库状态

这是留档仓库，不能独立运行。目录结构和 `src.platform.*` import 路径按原项目保留；沙箱服务、执行队列、节点注册表和 SQLite schema 仍在 Unimere 中。

抽离的原因是 Unimere 收敛为单人单机项目，弹性伸缩超出了当前使用需求。这里保留实现和测试，供以后有多节点需求时参考。

## 模块

| 文件 | 职责 |
|---|---|
| `src/platform/elastic_controller.py` | 周期采样、计算目标容量、执行 cordon/drain、记录事件 |
| `src/platform/autoscale.py` | 根据队列深度与时延计算目标容量 |
| `src/platform/whatif.py` | 只读模拟策略效果，不修改真实状态 |
| `src/platform/trends.py` | 保存容量和队列时序样本，用于报表与回放 |
| `src/platform/scale_events.py` | 保存扩缩事件，隔离来源并支持时间偏移回放 |
| `src/platform/alerts.py` | 评估队列积压、节点失联等告警规则 |

## 设计约定

- 控制面与数据面分开：控制器产生目标容量和节点指令，实际执行由 runner 完成。
- 扩缩决策、模拟和告警写入 SQLite，便于回放和核对。
- What-if 路径只读取真实状态；快照中的单个区块失败不应影响整页。
- 控制器默认关闭，只有设置 `PERSONAL_AGENT_ELASTIC` 后才启用；未启用时只提供计算结果。

## 目录

```text
src/platform/    # 六个控制面模块，保留原项目路径
tests/           # 单元测试和集成测试，依赖原项目运行时与 schema
scripts/         # demo_elastic_loop.py、autoscale_compare.py 等脚本
```

测试和脚本同样按留档保留，不能据此认为当前仓库可以脱离 Unimere 单独安装或运行。
