# 规范基线验证记录

日期：2026-10-28。范围：本次文档与仓库初始化。执行者：项目协作助手；尚无三位成员的独立交叉评审结论。

## 已执行的文档检查

| 检查 | 实际结果 |
| --- | --- |
| 文件范围 | 40 个文件，其中 37 个 Markdown、3 个仓库配置；没有应用源码、运行数据或独立程序脚本 |
| 规范与编号 | 八份规范、88 条规则定义，编号无重复 |
| 可验收场景 | 19 个练习发布/反馈目标场景、5 个本轮文档验收场景，编号无重复 |
| 本地文件链接 | 62 个本地链接全部存在；外链不计入本地存在性检查 |
| 文本与格式 | UTF-8 可读，无替换字符，Markdown 围栏闭合 |
| 补丁格式 | 已执行 `git diff --cached --check` 与 `git diff --check`，退出码均为 0 |
| 公开内容审查 | 文件类型、明显凭据模式和本机路径检查无异常；人工核对未包含真实学生资料或参考项目源码 |
| 检查有效性 | 临时新增一条坏链接后，检查退出码为 1 并定位来源；移除样例后退出码恢复为 0 |

本地链接检查只验证文件目标存在，不声称检查了外部网站可达性或页面排版。简单凭据模式检查不是完整安全扫描，后续工具检查状态见检查清单。

## 契约一致性核对

- 明确机构管理员不隐式获得学生作答正文权限，教师访问依当前有效授权。
- 同步了个人练习规则：学生只能创建本人接收的练习，不借教师班级发布接口扩大范围。
- 将幂等缓存过期后的轮次重复创建写成独立场景，返回 409 并保留唯一约束。
- HTTP 与业务码同时作为契约，未作答不算正确，反馈更正不新增轮次。
- 所有业务接口、测试、模型评测和部署能力明确标记为目标或待建设。

## 可复现的文档检查

在仓库根目录使用 PowerShell 和 Python 3 运行以下只读检查。它不调用网络或模型，也不修改仓库。这里保留实际使用的检查逻辑作为证据，没有建立应用或测试框架。

```powershell
@'
import sys, pathlib, re, json, subprocess, collections
root = pathlib.Path(sys.argv[1]).resolve()
names = subprocess.check_output(["git", "-C", str(root), "ls-files", "--cached", "--others", "--exclude-standard"], text=True, encoding="utf-8").splitlines()
names = sorted(set(names))
errors, rules, scenarios = [], [], []
markdown = [n for n in names if n.endswith(".md")]
links = 0
allowed = {".gitignore", ".gitattributes", ".editorconfig"}
for name in names:
    if not name.endswith(".md") and name not in allowed:
        errors.append({"file": name, "error": "unexpected_non_document"})
for name in markdown:
    p = root / name
    try:
        s = p.read_text(encoding="utf-8")
    except UnicodeError:
        errors.append({"file": name, "error": "invalid_utf8"})
        continue
    if "\ufffd" in s:
        errors.append({"file": name, "error": "replacement_character"})
    if len(re.findall(r"(?m)^\s*```", s)) % 2:
        errors.append({"file": name, "error": "unclosed_fence"})
    prose = re.sub(r"(?ms)^\s*```.*?^\s*```\s*$", "", s)
    for target in re.findall(r"\[[^\]]*\]\(([^)]+)\)", prose):
        if re.match(r"^(https?://|mailto:|#)", target):
            continue
        target = target.split("#", 1)[0]
        links += 1
        dest = (p.parent / target).resolve()
        if not dest.is_relative_to(root) or not dest.exists():
            errors.append({"file": name, "error": "broken_or_external_local_link", "target": target})
    rules += re.findall(r"(?m)^\| ((?:PROD|BE|SEC|TEST|OPS|AI|DATA|REV)-\d+) \|", s)
    if not name.startswith("docs/templates/"):
        scenarios += re.findall(r"(?m)^### Scenario: (SC-[A-Z]+-\d+)", s)
    if re.search(r"(?:C|J|H):[\\/]", s):
        errors.append({"file": name, "error": "local_machine_path"})
    if re.search(r"\b(?:gh[pousr]_[A-Za-z0-9]{20,}|sk-(?:proj-)?[A-Za-z0-9_-]{24,})\b", s):
        errors.append({"file": name, "error": "possible_secret_no_value_printed"})
for kind, values in [("rule", rules), ("scenario", scenarios)]:
    for value, count in collections.Counter(values).items():
        if count > 1:
            errors.append({"error": "duplicate_" + kind, "id": value})
norms = list((root / "docs/standards").glob("[0-9][0-9]-*.md"))
if len(norms) != 8:
    errors.append({"error": "standards_count", "count": len(norms)})
result = {"files":len(names),"markdown":len(markdown),"local_links":links,"standards":len(norms),"rule_definitions":len(rules),"scenario_definitions":len(scenarios),"errors":errors}
print(json.dumps(result, ensure_ascii=False, indent=2))
sys.exit(1 if errors else 0)
'@ | python - .
```

负向检查步骤：在 `docs/verification/` 临时建立 Markdown，令其链接到不存在的文件；运行上面命令，期望非零退出且报告该来源；移除临时文件后重新运行，期望退出码 0。本次已实际完成该步骤，临时文件不入库。

## GitHub 发布核验

已通过 GitHub API 发布并读取核验：仓库 `JOKER20180713/mistake-learning-platform` 为 PUBLIC，默认分支 main，40 个文件和八类规范均存在。首轮内容提交为 `0080da9b187f8cbade1226ca5d8a40ef57805d1d`，文件树为 `664758075809df84fca6b8d1300821cf4a5ee71c`，与本地 Git 文件树完全一致；本地已同步同一提交。

Git HTTPS 推送两次遭连接重置，因此改用已认证的 GitHub API 创建文件树与提交，正常快进更新 main，未强推或覆盖共享历史。导回本地的提交逐个校验对象 SHA，`git fsck --connectivity-only` 通过。随后补充本验证记录与任务完成状态，最终提交可从仓库历史定位。

实际核验使用 `gh repo view JOKER20180713/mistake-learning-platform --json nameWithOwner,isPrivate,defaultBranchRef`、`gh api repos/JOKER20180713/mistake-learning-platform/git/ref/heads/main`、递归文件树查询和本地 `git rev-parse HEAD`。最终发布后再次比对远端提交与本地 HEAD。

## 明确未执行

没有运行应用、数据库迁移、模型调用、业务测试、性能压测、恢复演练、CI 或非作者团队评审；没有邀请协作者、设置 CODEOWNERS 或启用分支保护。未实现项已登记为待建设，不作通过判断。

## 初始发布例外

按用户要求创建公开仓库并建立规范；首次 main 初始化和文档规模例外见 [例外台账](../decisions/exceptions.md)。本轮补充验证记录属于初始化收尾，后续功能与规范迭代使用 PR 与独立评审。
