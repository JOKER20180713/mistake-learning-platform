# 错题诊断与个性化练习平台

面向培训机构，由教师组织学生完成“上传与确认—错因分析—针对性练习—结果反馈—复习”的学习流程。正确与错误作答共同关联知识点，教师可查看授权教学范围内的学习记录。

## 项目成员

| 成员 | 主责 | 规范维护范围 |
| --- | --- | --- |
| 孙翌程 | 产品交互与前端 | 产品设计、页面与术语；参与端到端验收 |
| 司浩鸣 | 后端业务与数据 | 后端、安全、数据、部署运维 |
| 周潇洋 | 模型分析与检索 | AI 应用、模型评测与推荐质量 |
| 三人共同 | 联调、评审与交付 | 测试规范、完成定义、变更评审与证据归档 |

以上为职责分工，不表示已邀请 GitHub 协作者或配置 CODEOWNERS；GitHub 用户名映射待补充。

## 当前阶段

2026-10-28 建立规范基线 v0.1。本仓库当前只包含产品与工程文档、协作模板和仓库基础配置，没有应用代码、运行服务、模型密钥、学生数据或题库文件。

本版规则是后续实现的共同起点。标有“待决”的业务选择需要在依赖它的功能开始前确定；其余无依赖的工作可以继续。文档中的拟定接口、检查项和发布指标是待实现的契约，不代表接口已存在、测试已通过或服务已上线。

## 阅读入口

1. [产品定位与范围](docs/product/brief.md)、[角色与权限](docs/product/roles-and-access.md)。
2. [术语表](docs/product/glossary.md)、[状态与流程](docs/product/states-and-flows.md)。
3. [八类规范索引](docs/standards/README.md)。
4. [接口契约](docs/contracts/api.md)、[数据字典](docs/contracts/data.md)。
5. [协作流程](CONTRIBUTING.md)、[检查入口与实施状态](docs/verification/checks.md)。
6. [待决事项](docs/decisions/open-questions.md)、[全部文档索引](docs/README.md)。

## 规范与规约如何配合

- `docs/standards/`：长期遵守的八类工程要求，每条有编号和检查方式。
- `openspec/specs/`：产品行为的目标基线，保存可验收场景。
- `openspec/changes/`：当前变更的动因、设计、增量规约和任务。
- `docs/workflows/`：一次变更、发布、迁移和模型调整的执行步骤。
- `docs/verification/`：检查能力清单和真实执行证据。

采用 OpenSpec 风格组织文件；当前没有安装或配置 OpenSpec CLI，不宣称通过 `/opsx-validate` 等工具校验。

## 本轮交付

本轮只完成规范与文档基线。详见 [本次变更](openspec/changes/2026-10-28-standards-baseline/proposal.md) 和 [验证记录](docs/verification/2026-10-28-standards-baseline.md)。

当前可直接运行的仓库检查为 `git diff --check` 与 `git diff --cached --check`。它们仅检查补丁格式，不验证业务或安全。业务测试、模型评测和 CI 的建设要求及未实现状态统一记录在检查清单中。

## 内容与公开范围

本仓库为公开仓库，面向老师与协作者开放规范。真实学生作答、个人信息、密钥、机构私有题库和未获授权的资料不得提交。公开可读不等于授予开源许可；项目许可证与题库授权安排见待决事项。

课程参考链接列于 [来源与采用说明](docs/references.md)。规范按本项目业务重新编写，未将参考项目源码或资料导入本仓库。
