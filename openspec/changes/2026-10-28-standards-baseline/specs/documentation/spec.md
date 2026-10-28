# 文档基线规约

状态：本轮交付验收；检查入口为文档检查与 Git 远端核验，见 [验证记录](../../../../../docs/verification/2026-10-28-standards-baseline.md)。

## Requirement: REQ-DOC-01 规范可定位且可检查

### Scenario: SC-DOC-01 八类规范齐备
- Given 新仓库只有规范阶段文件。
- When 从 README 打开规范索引。
- Then 能到达产品、后端、安全、测试、运维、AI、数据、评审八份文件，每份标明主责并使用唯一编号列出要求与检查方法。

### Scenario: SC-DOC-02 发现坏链接
- Given 任一 Markdown 的本地文件链接指向不存在路径。
- When 执行文档链接检查。
- Then 检查非零退出，报告来源文件与目标，不把缺失链接标成通过。

## Requirement: REQ-DOC-02 完成状态真实

### Scenario: SC-DOC-03 未实现能力明确标注
- Given 仓库没有应用、测试或 CI 程序。
- When 阅读 README、规约、任务和检查台账。
- Then 业务接口与场景明确为目标，程序检查明确未实现；不出现业务测试通过或已上线的结论。

### Scenario: SC-DOC-04 公开范围受控
- Given 用户选择公开仓库。
- When 检查待发布文件清单。
- Then 只包含原创建规范、模板和仓库配置，没有真实学生材料、凭据、参考项目源码或本机资料路径。

## Requirement: REQ-DOC-03 GitHub 仓库与本地一致

### Scenario: SC-DOC-05 发布规范
- Given 本地完成文档检查并提交。
- When 推送 main 并读取 GitHub 仓库元数据与默认分支。
- Then 仓库为 PUBLIC，默认分支 main，远端 HEAD 等于本地提交，README 和八类规范存在。
