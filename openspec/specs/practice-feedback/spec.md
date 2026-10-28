# 练习发布与结果反馈

- 状态：目标规约 v0.1，2026-10-28，尚未实现。
- 覆盖：教师发布、学生逐题反馈、反馈修订与练习文件授权。
- 检查入口：当前为文档场景核对；未来 CHK-API / CHK-DOMAIN / CHK-E2E，命令尚未建设，见 [检查清单](../../../docs/verification/checks.md)。
- 依据：[角色与权限](../../../docs/product/roles-and-access.md)、[状态机](../../../docs/product/states-and-flows.md)、[接口契约](../../../docs/contracts/api.md)。

## 固定测试背景

所有数据均为合成：机构 A、B；教师 TA 仅有 A 的班级 C 授权，教师 TX 为 A 内无 C 授权教师；S1、S2 为 C 的学生，S3 为 A 内其他班学生，SB 为 B 的学生。草稿 P 属于 TA，version=3，含 5 个可用题项 I1～I5 及已冻结候选版本，五题均以 KP1 为主要知识点，便于核对汇总。各场景从独立背景初始化，不能依赖上一场景残留。

## Requirement: REQ-PF-01 发布必须准确且原子化

### Scenario: SC-PF-01 正常发布
- Given TA 的草稿 P 尚未发布，S1、S2 在其有效授权范围，题项完整。
- When TA 向 `POST /api/v1/practice-sets/P/publications` 提交目标 S1、S2、expected_version=3 和新幂等键 K1。
- Then 返回 201/OK，data 包含 publication_id 和 practice_set_id=P。
- And P 变为 published，恰有 S1、S2 两条分配，题项顺序与版本固定；产生一条发布审计事件。

### Scenario: SC-PF-02 同一次发布被重放
- Given K1 对 P、S1、S2 的发布已成功。
- When TA 用 K1 重发完全相同请求。
- Then 返回原 201/OK 及同一 publication_id，request_id 可不同。
- And 分配仍为两条，不产生第二次发布事件或重复任务。

### Scenario: SC-PF-03 批量目标中存在其他机构学生
- Given P 仍为草稿，没有任何分配。
- When TA 提交目标 S1、SB 发布 P。
- Then 返回 404/RESOURCE_NOT_FOUND，不返回 SB 的身份或机构信息。
- And P 仍是草稿，S1 也不得收到分配；仅有脱敏拒绝审计。

### Scenario: SC-PF-04 无授权教师访问草稿
- Given TX 没有 P 的教学访问资格。
- When TX 请求发布 P。
- Then 返回 404/RESOURCE_NOT_FOUND，P 的内容、状态和分配不变。

### Scenario: SC-PF-05 学生尝试发布班级练习
- Given S1 为学生，未具备教师发布能力。
- When S1 调用班级练习发布接口。
- Then 返回 403/ACCESS_DENIED，不执行资源发布，不返回草稿信息。

### Scenario: SC-PF-06 旧版本发布
- Given TA 有权访问 P，P 当前 version=4，仍为草稿。
- When TA 使用新幂等键并提交 expected_version=3 发布 P。
- Then 返回 412/VERSION_CONFLICT，无分配记录和发布副作用。

## Requirement: REQ-PF-02 正确、错误和未完成状态分别保存

### Scenario: SC-PF-07 提交一轮混合结果
- Given P 已发布给 S1 且未关闭，S1 的轮次 R1 尚未提交。
- When S1 提交 I1=correct、I2=incorrect、I3=partial、I4=unanswered、I5=pending_review，正式判定来源为 student_self，使用 K2。
- Then 返回 201/OK，生成唯一 submission_id，保存 5 条题项结果与本轮快照。
- And I1～I3 为 3 条已判定题项；I4、I5 不计入正确率分母；I2 进入或更新 S1 的错题记录。
- And 汇总事件处理完成后，正确率为 1/3；处理期间页面标记统计更新中；原作答与来源可追溯。

### Scenario: SC-PF-08 缺失结果不得默认为正确
- Given S1 已填写 I1～I4，I5 为 unknown 或缺失。
- When S1 提交 R1。
- Then 返回 422/VALIDATION_FAILED，errors 指向缺少显式结果的题项。
- And 不创建正式反馈、错题更新或统计事件，草稿可保留。

### Scenario: SC-PF-09 未分配学生不能提交
- Given S3 未被分配 P。
- When S3 用 P 的编号提交反馈。
- Then 返回 404/RESOURCE_NOT_FOUND，不产生反馈或统计；不展示 P 的题干和接收人。

### Scenario: SC-PF-10 反馈重试不重复计数
- Given K2 的 R1 反馈已成功。
- When S1 用 K2 重发相同请求。
- Then 返回原 201/OK 与同一 submission_id，5 条有效题项及 1 个轮次不变。

### Scenario: SC-PF-19 幂等记录过期后仍不能重复创建轮次
- Given R1 反馈已存在，K2 的幂等缓存已过期。
- When S1 使用新幂等键再次创建 R1 的正式反馈。
- Then 返回 409/STATE_CONFLICT，不新增反馈或第二轮，原有题项结果不变。

### Scenario: SC-PF-11 同一幂等键提交不同内容
- Given K2 对 R1 已保存成功请求摘要。
- When S1 用 K2 把 I2 改为 correct 后重发。
- Then 返回 409/IDEMPOTENCY_CONFLICT，原结果不变；更正需使用修订入口。

### Scenario: SC-PF-12 关闭后提交
- Given S1 被分配 P，但 P 已 closed。
- When S1 新增普通反馈。
- Then 返回 409/STATE_CONFLICT，data 给出当前 closed 状态及允许的安全操作。
- And 不新增反馈、轮次或正式判定，历史记录不变。

## Requirement: REQ-PF-03 更正保留历史且不重复记账

### Scenario: SC-PF-13 更正本人自评
- Given S1 的 R1 version=1，结果为场景 SC-PF-07 的混合状态，尚无教师覆盖判定。
- When S1 对原 submission 提交修订，将 I2 改为 correct，提供 expected_version=1 和更正理由。
- Then 返回 201/OK，新增修订 version=2，旧修订、理由和行为人保留。
- And 汇总完成后正确率变为 2/3；轮次数仍为 1，独立题项数不增加；原错误历史仍可追溯。

### Scenario: SC-PF-14 并发更正不覆盖
- Given 同一反馈当前 version=1，两个合法修改都基于 version=1。
- When 两个请求提交不同修订。
- Then 只有一个返回 201 并产生 version=2，另一个返回 412/VERSION_CONFLICT。
- And 不存在两个同时有效的 version=2，先成功的结果不被静默覆盖。

### Scenario: SC-PF-15 学生不能改写教师判定
- Given I2 的当前有效判定来源为 teacher。
- When S1 试图直接修改该教师判定。
- Then 返回 403/ACCESS_DENIED，原判定不变；学生可通过另行定义的纠错请求提交说明。

## Requirement: REQ-PF-04 下载与反馈保持同一权限和题号

### Scenario: SC-PF-16 合法下载
- Given S1 被分配 P，P 的题目卷导出 E 已 succeeded。
- When S1 请求 P 下 E 的 file 接口。
- Then 返回 200 和 application/pdf，X-Request-Id 存在。
- And PDF 中 I1～I5 的固定题号、题目版本及练习编号与反馈页面对应，题目卷不混入答案解析。

### Scenario: SC-PF-17 跨机构访问导出
- Given SB 不属于机构 A，也未被分配 P。
- When SB 使用 P 和 E 的编号请求下载。
- Then 返回 404/RESOURCE_NOT_FOUND，不返回文件字节、重定向地址或签名 URL。

### Scenario: SC-PF-18 匿名访问导出
- Given 请求无有效会话。
- When 请求 P 下 E 的文件接口。
- Then 返回 401/AUTH_UNAUTHENTICATED，不返回任何 PDF 内容。

## 实施边界

上述 19 个场景是待实现的验收目标，没有假定应用、数据库或测试已存在。真正实施时按身份授权、练习发布、反馈修订和导出拆成可独立评审的变更；人工评阅、消息通知和商业计费不因该样例自动纳入当前开发范围。
