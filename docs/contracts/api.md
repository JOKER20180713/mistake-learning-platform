# API 契约基线

状态：目标契约，接口尚未实现。所有业务接口使用 `/api/v1/`；角色、数据范围及状态分别引用产品权限表和状态机。

## 统一响应

JSON 响应包含 `code`（字符串业务码）、`message`（可读说明）、`data`（成功数据或 null）、`request_id`（当前请求标识）。失败可增加 `errors` 数组，元素只含安全的 `field/code/message`，不回显原始敏感输入。HTTP 状态与业务码共同构成契约，不采用 HTTP 200 包装业务失败。

成功代码固定 `OK`，创建资源通常 201，读取和修改 200，可靠接受异步任务 202。下载成功返回文件字节和正确的 Content-Type/Content-Disposition，响应头携带 `X-Request-Id`；下载失败仍返回 JSON 错误。

例如不存在或无权读取指定对象：HTTP 404，`code=RESOURCE_NOT_FOUND`、`message=资源不存在`、`data=null`。不同请求的 request_id 可不同，其他业务字段一致。

## 错误码

| HTTP | code | 含义与固定 message |
| --- | --- | --- |
| 401 | AUTH_UNAUTHENTICATED | 会话无效，请重新登录 |
| 403 | ACCESS_DENIED | 当前账号无此操作权限 |
| 404 | RESOURCE_NOT_FOUND | 资源不存在 |
| 409 | STATE_CONFLICT | 当前状态不允许此操作 |
| 409 | IDEMPOTENCY_CONFLICT | 相同提交标识不能用于不同内容 |
| 412 | VERSION_CONFLICT | 内容已被更新，请刷新后重试 |
| 413 | PAYLOAD_TOO_LARGE | 上传内容超过限制 |
| 415 | UNSUPPORTED_MEDIA_TYPE | 文件格式不受支持 |
| 422 | VALIDATION_FAILED | 输入不符合要求，请检查标记字段 |
| 429 | RATE_LIMITED | 请求次数超过限制，请稍后重试 |
| 503 | DEPENDENCY_UNAVAILABLE | 服务暂不可用，请稍后重试 |
| 500 | INTERNAL_ERROR | 处理失败，请提供请求编号联系支持 |

仅已通过授权的状态冲突才返回 `data.current_status` 和允许的操作，不向越权请求暴露对象状态。429 提供整数秒 `Retry-After`；不能保证恢复时间时不随意给出确定承诺。业务错误不会返回堆栈、数据库错误或供应商密钥。

## 拟定资源接口

这些路径为首期契约草案，新增或调整必须同步规约，不是当前可调用地址。

| 方法与路径 | 行为 | 核心约束 |
| --- | --- | --- |
| `POST /api/v1/uploads` | 接收图片并创建上传记录 | 当前学生身份、大小/格式限制、幂等键 |
| `POST /api/v1/uploads/{id}/confirmations` | 确认识别内容版本 | 本人、指定版本和完整题目内容 |
| `POST /api/v1/analysis-jobs` | 接受分析任务 | 已确认的可访问上传/题目版本、幂等键 |
| `GET /api/v1/jobs/{id}` | 查询任务状态 | 任务所属资源可访问，不回传供应商原始错误 |
| `GET /api/v1/questions` | 列表或检索题目 | 授权范围、题库版本、白名单筛选 |
| `POST /api/v1/recommendations` | 请求相似练习候选 | 合法题目版本、训练目标与权限；无候选返回空及原因 |
| `GET /api/v1/mistakes` | 查看个人或授权教学范围错题 | 不根据客户端 student_id 放宽授权 |
| `POST /api/v1/practice-sets` | 创建练习 | 教师创建草稿；学生仅创建本人个人练习并冻结发布，最多 50 个题项 |
| `POST /api/v1/practice-sets/{id}/publications` | 发布练习 | 教师作者、目标授权、版本、幂等键，最多 200 名目标学生 |
| `POST /api/v1/practice-sets/{id}/exports` | 创建 PDF 导出任务 | 已授权练习对象、指定快照与导出类型 |
| `GET /api/v1/practice-sets/{id}/exports/{exportId}/file` | 下载已完成导出 | 每次下载重新核验练习权限 |
| `POST /api/v1/practice-sets/{id}/submissions` | 提交本人本轮反馈 | 分配资格、开放状态、显式逐题结果、幂等键 |
| `POST /api/v1/submissions/{id}/revisions` | 更正录入或教师评阅 | 当前版本、权限、理由，保留历史 |

单次发布超过上限应显式拒绝并要求拆分，不可部分发布。班级目标在发布时解析为固定学生集合；后续新入班学生通过新练习或明确的后续分配变更处理，不悄悄改历史快照。

## 分页与参数

`page` 从 1 开始，`page_size` 默认 20、范围 1～100。未知筛选字段、无效排序字段、类型不符或超出上限均 422。排序字段非唯一时追加 ID；默认 `created_at DESC, id DESC`。超出末页返回 200 和空 `items`，计数仅包含当前有权访问的对象。

可选配置的默认值在配置登记中明确；必需配置缺失启动失败。字符串业务 ID 与参数长度有界，框架自带校验错误统一转为本契约格式。

## 幂等与并发

- 任务创建、发布、反馈提交使用 `Idempotency-Key`；客户端为一次逻辑操作生成 UUID，网络重试复用，真正的新轮次生成新值。
- 先认证和授权，再查幂等记录。作用域为机构、行为人、方法、规范化资源路径和键；记录请求内容摘要、最终结果与资源 ID，保存至少 24 小时。
- 同键同体重放返回原 HTTP 业务结果和 data，可生成当前 request_id；同键异体返回 409。进行中返回原受理任务引用，不能再建任务。
- 过期幂等键不等于可以重复创建正式反馈：存储仍按学生、练习、轮次施加唯一性约束；使用新键重复创建已有轮次返回 409/STATE_CONFLICT，同一轮次修改走修订接口。
- 编辑和发布携带 `If-Match` 或契约指定的 `expected_version`，两者不同时使用。首期 JSON 写入统一使用 `expected_version`，缺失返回 422，旧版本返回 412。
- 发布与分配、反馈与错题更新同事务完成；派生汇总与任务通过可靠事件推进。模型或文件操作不在事务内。

## 异步结果

202 的 data 包含 `job_id/status`，并提供状态查询地址。任务失败不修改已经发送的 HTTP 响应，在任务结果内返回安全的 `failure_code`、重试建议和时间。结果校验失败必须显式失败，不能返回空 data 冒充成功。

无推荐命中返回 200、`items=[]` 和 `empty_reason`，取值 `no_candidates / below_threshold / filtered_by_scope`；只对当前授权范围解释，不泄露其他机构候选的存在。检索依赖故障与“没有命中”严格区分。
