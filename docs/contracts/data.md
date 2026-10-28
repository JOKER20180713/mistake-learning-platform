# 逻辑数据字典

状态：概念模型与字段语义基线，不是已执行的数据库结构。物理字段类型、索引与迁移在工程设计变更中决定。

## 通用约束

- 外部业务标识统一为不透明字符串；UTC 时间以带时区的 ISO 8601 表达。所有机构业务实体具有 `organization_id`，或有能够严格校验的所属关系。
- 可修改实体具有 `id/created_at/updated_at/created_by/version`；软删除是处理状态，不代表内容已完成物理清除。
- 外键关系同时验证机构一致性，不能仅验证 ID 存在。审计记录只追加，敏感正文按留存策略删除。
- 身份账号与机构内学生档案分离；同一人跨机构的学习资料不自动合并。

## 实体

| 实体 | 关键字段或关系 | 含义与约束 |
| --- | --- | --- |
| accounts | id、认证标识、会话版本 | 全局身份；对外不返回密码摘要或认证凭据 |
| organizations | id、名称、状态 | 机构隔离边界，停用后成员不能继续业务访问 |
| memberships | account_id、organization_id、role、status | 机构角色关系，不直接授予全部学生正文权限 |
| classes / class_memberships | organization_id、class_id、student_id、有效期 | 班级与学生归属；解除关系后重新判定教师访问 |
| teacher_class_grants | organization_id、teacher_id、class_id、status | 教师教学授权，独立于管理员身份 |
| uploads / files | owner_student_id、organization_id、object_key、hash、类型、版本 | 原始材料及私有对象引用，公开响应不泄露内部路径 |
| questions / question_versions | organization_id、owner_id、visibility、source、authorization、version | 题干、题图、选项、参考解答；私有题与审核发布题有明确范围 |
| question_items | question_version_id、item_id、题号、max_score | 多小问的独立题项，题号不是全局 ID |
| knowledge_points / question_knowledge_points | curriculum_id、稳定编号、主次、映射版本 | 同一题可关联多个知识点；变更映射需要统计重算策略 |
| analysis_jobs / analyses | input_version、attempt_id、model_version、prompt_version、status、evidence | 原始输入与输出可追溯，分析不直接替代正式作答判定 |
| question_embeddings | question_version_id、model_id、dimension、index_version、status | 仅当前有效且有权限的版本参与推荐 |
| practice_sets / practice_items | author_id、status、version、固定题序、question_version_id | 发布时冻结题目及答案版本，不受后续题库编辑影响 |
| assignments | organization_id、practice_set_id、student_id | 接收资格，组合唯一，不能靠知道练习 ID 获得访问 |
| submissions / attempts / revisions | student_id、practice_set_id、round_id、item_id、result、judge_source、revision | 学生一轮反馈及逐题记录；修改用修订，不新增轮次计数 |
| mistakes | student_id、question_id、首次/最近错误、复习状态 | 引用作答历史，当前答对不删除历史错误 |
| knowledge_stats | organization_id、student_id、knowledge_point_id、有效统计版本 | 可重建的汇总，不作为作答事实的替代来源 |
| exports | practice_set_version、file_id、status、导出类型 | 练习卷与答案解析分开，下载按原业务对象授权 |
| outbox / idempotency_records | event_id、业务键、载荷版本、状态、重试、请求摘要 | 可靠任务和去重；载荷尽量仅含资源 ID 与版本 |
| audit_events / deletion_jobs | actor、organization_id、对象、操作、版本、时间、原因、各副本状态 | 可追溯管理与清理，不长期复制学生正文 |

## 作答统计口径

当前有效判定为每个题项、每轮的最新有效修订，保留旧修订用于审计。正式判定来源仅 `rule/student_self/teacher`，`model_suggestion` 未确认前不计入。

正确率 = correct 数 ÷（correct + incorrect + partial 数）；partial 单独展示。unanswered、pending_review、unknown 不进入分母；分母为 0 显示“暂无有效结果”，不能显示 0% 表示学不会。得分率仅在题项分值可靠且均有正式得分时计算，不能把部分正确固定折算为 0.5 分。

统计同时提供总轮次数和独立题项数。综合题只有整体结果且无法归因时，只记录整体关联练习事实，不把所有关联知识点都标成错误原因。知识点错误归因需有小问或明确证据映射。

固定验收样例：一轮 5 题，结果为 correct、incorrect、partial、unanswered、pending_review；已判定 3 题，正确率 1/3，明确未作答 1 题、待批改 1 题。把第二题同轮次更正为 correct 后，正确率变 2/3，轮次数和独立题数不增加。

## 数据分级与副本

公开：经授权明确发布的题目和产品说明。内部：组织配置、题库审核信息、运行元数据。敏感/个人：学生身份、原始笔迹、作答、成绩和错因分析。机密：密码摘要、会话凭据、供应商密钥。

同一原始材料可能产生对象文件、识别文本、分析、向量、缓存和 PDF。删除任务必须列出这些副本和备份到期信息；共享题目与个人作答分别判定是否仍有合法保留范围。完整留存规则以数据规范和试点前决策为准。
