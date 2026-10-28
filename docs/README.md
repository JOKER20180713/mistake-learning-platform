# 文档索引

| 范围 | 入口 | 维护原则 |
| --- | --- | --- |
| 产品与用户 | [产品范围](product/brief.md) | 确认的方向与首期假设分开 |
| 授权 | [角色与权限](product/roles-and-access.md) | 所有资源访问共享同一授权判定 |
| 用户与数据用词 | [术语表](product/glossary.md) | 同一概念只有一个规范名称 |
| 状态机 | [状态与流程](product/states-and-flows.md) | 写明合法转换、前提与终态 |
| 工程要求 | [八类规范](standards/README.md) | 每条要求有检查动作 |
| 接口 | [API 契约](contracts/api.md) | HTTP、业务码、请求标识和幂等同源 |
| 数据 | [数据字典](contracts/data.md) | 含义、归属、版本和删除策略同源 |
| 决策 | [架构方向](decisions/0001-architecture-direction.md)、[待决项](decisions/open-questions.md) | 不把建议写成已完成验证 |
| 流程 | [变更](workflows/change.md)、[模型调整](workflows/ai-change.md)、[迁移](workflows/data-change.md)、[发布](workflows/release.md) | 按动作执行，不重复定义规则 |
| 验证 | [检查入口](verification/checks.md)、[本轮记录](verification/2026-10-28-standards-baseline.md) | 区分已执行、计划、未配置 |
| 例外 | [例外台账](decisions/exceptions.md) | 理由、范围、期限与补偿可追溯 |
| 模板 | [四文件变更模板](templates/change-template.md) | 复制结构后填写本项目事实 |
| 来源 | [参考与采用说明](references.md) | 只引用外部依据，不复制整篇课件 |

业务目标基线在 [openspec/specs](../openspec/specs/README.md)，本次文档变更在 [规范基线变更](../openspec/changes/2026-10-28-standards-baseline/proposal.md)。

文档冲突时先修订唯一维护来源及其引用，不在不同文件各写一套规则。API 以接口契约为准，权限以角色表为准，状态以状态机为准；正式变更批准后同步更新相应来源。
