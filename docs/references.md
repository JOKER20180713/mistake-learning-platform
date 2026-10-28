# 参考来源与采用说明

本项目规范参考用户提供的课程资料，按培训机构错题与练习场景重新制定。来源提供方法，不代表已认可本项目技术、业务阈值或上线状态。

| 来源 | 采用内容 |
| --- | --- |
| [可执行规约怎么写](https://devops.hello1023.com/harness/spec/writing.html) | 前置条件、操作、可观察结果，失败路径与可复核证据 |
| [产品设计规范](https://devops.hello1023.com/课件/第7课-课件-小组项目启动/pages/std-product.html) | 页面与状态、术语、视觉、导入导出检查 |
| [后端开发规范](https://devops.hello1023.com/课件/第7课-课件-小组项目启动/pages/std-backend.html) | 分层、接口契约、幂等与事务边界 |
| [安全开发规范](https://devops.hello1023.com/课件/第7课-课件-小组项目启动/pages/std-security.html) | 逐资源授权、机密与不可信内容边界 |
| [测试规范](https://devops.hello1023.com/课件/第7课-课件-小组项目启动/pages/std-testing.html) | 场景映射、数据隔离与行为断言 |
| [部署运维规范](https://devops.hello1023.com/课件/第7课-课件-小组项目启动/pages/std-ops.html) | 可复现发布、健康、观测及恢复 |
| [AI 应用规范](https://devops.hello1023.com/课件/第7课-课件-小组项目启动/pages/std-ai.html) | 版本化、输出校验、评测与成本 |
| [数据规范](https://devops.hello1023.com/课件/第7课-课件-小组项目启动/pages/std-data.html) | 结构、迁移与多副本生命周期 |
| [评审与完成定义](https://devops.hello1023.com/课件/第7课-课件-小组项目启动/pages/std-review.html) | 完成条件、责任、例外和证据 |
| [规范案例](https://devops.hello1023.com/课件/第7课-课件-小组项目启动/pages/case.html) | 项目事实与判据相连，按生效时机组织规则与流程 |
| [变更动因](https://devops.hello1023.com/课件/第7课-课件-小组项目启动/pages/spec-case.html)、[设计](https://devops.hello1023.com/课件/第7课-课件-小组项目启动/pages/spec-case2.html)、[需求](https://devops.hello1023.com/课件/第7课-课件-小组项目启动/pages/spec-case3.html)、[任务](https://devops.hello1023.com/课件/第7课-课件-小组项目启动/pages/spec-case4.html) | 一次变更的四份可追溯文件 |

## 采用时处理的差异

- 班级隔离扩展为机构隔离、教师教学授权与学生本人归属；同机构共享题库另定义可见范围。
- 保留统一 HTTP 与业务码契约，测试同时核对两者，不接受用 HTTP 200 隐藏业务失败。
- 例子中的技术栈、阈值和性能数值未直接视为本项目实测结果；本项目数值均注明首期约定或待验证目标。
- “不存在”与“越权”比较业务响应字段，request_id 等请求元数据可以不同。
- 必需配置缺失失败与选填配置默认值分开，不同时要求某字段“缺失失败”又“自动用默认值”。
- 数据删除包含备份生命周期与恢复后重放删除清单，不宣称在线删除等于所有历史介质瞬间清空。

未复制课件全文，未将既有本地参考项目的源码、题库、学生资料或内部路径提交本仓库。
