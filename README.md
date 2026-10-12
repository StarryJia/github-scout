# github-scout

一个轻量的 GitHub 开源项目调研 Agent Skill（可由智能体加载的任务说明），用于快速了解项目用途、仓库创建时间、Stars 增长和提交活跃情况。默认中文说明，英文标题及表头，优先使用紧凑且带来源的 Markdown 表格。

例如，你想比较几个开源工具是否适合自己的用途时，Agent 会先核实它们的功能，再查询星标、提交和贡献者记录，帮助你看到功能介绍之外的实际项目状态。

## 功能特点

- **单项目概览**：简介、仓库创建时间、主要语言、许可证、Stars（星标数）、Forks（派生仓库数）、Commits（提交数）、Contributors（贡献者记录）、最近提交和最近发布。
- **Stars 发展概览**：比较最近 4 个完整统计周与前 4 周的新增星标量，注明周期和变化率；有可信历史总量快照时，另报净增长。例如，模拟两个周期分别获星 120 和 80 时，报告新增星标量增加 50%，不把 120 当成总星标数净增长。
- **紧凑展示**：单项目和多项目均用横向表格，表头尽量沿用 GitHub 英文原文；日期只显示年月日，详细说明放在表外。每个项目附仓库链接和简短用途说明。
- **自动选择**：推荐、查找、列举、比较或评估 GitHub 开源项目时可主动补充统计；也支持 `$github-scout` 显式调用。
- **明确统计口径**：提交数优先统计默认分支，固定查询时的提交快照；贡献者处理分页、匿名记录和 GitHub 统计限制。
- **如实处理失败**：某个接口限流时，该项显示未知并说明原因，其他成功取得的数据仍可展示。

这是一个说明型 Skill，查询由 Agent 使用已有工具完成。仓库不包含运行脚本、依赖清单、综合评分、代码审计或安全扫描功能。

## 仓库结构

```text
github-scout/
├── SKILL.md
├── README.md
└── LICENSE
```

`SKILL.md` 位于根目录；安装后的技能目录名应为 `github-scout`，与其元数据中的 `name` 一致。工作流程、字段口径和错误处理以 [SKILL.md](SKILL.md) 为准。

## 安装到 Codex

按当前 [Codex 官方技能文档](https://learn.chatgpt.com/docs/build-skills)，用户级技能放在 `~/.agents/skills/`，项目级技能放在项目的 `.agents/skills/`。根目录存放 Skill 是发布结构，使用时需放入上述发现目录。

### 手动安装，不需要额外安装工具

需要已有 Git。以下远端命令适用于仓库内容上传 GitHub 后；目标目录应尚不存在。若已安装，直接更新对应目录，不要再次克隆覆盖。

macOS / Linux：

```sh
mkdir -p "$HOME/.agents/skills"
git clone https://github.com/StarryJia/github-scout.git "$HOME/.agents/skills/github-scout"
```

Windows PowerShell：

```powershell
New-Item -ItemType Directory -Force -Path (Join-Path $HOME '.agents/skills') | Out-Null
git clone https://github.com/StarryJia/github-scout.git (Join-Path $HOME '.agents/skills/github-scout')
```

只想在一个项目使用时，在该项目目录执行：

```sh
git clone https://github.com/StarryJia/github-scout.git .agents/skills/github-scout
```

也可将本仓库三个文件复制到上述 `github-scout` 目录，确保得到 `github-scout/SKILL.md`，不要多套一层同名目录。发布前可用这种方式安装当前本地文件。Codex 通常自动发现变更；如果没有出现，重启 Codex，在技能选择器查看或输入 `$github-scout` 验证。

手动克隆安装的更新方法，路径按安装位置调整：

```sh
git -C "$HOME/.agents/skills/github-scout" pull --ff-only
```

PowerShell 可将路径换为 `(Join-Path $HOME '.agents/skills/github-scout')`。更新前先检查本地改动，避免覆盖个人修改。不同设备分别安装，更新后使用相同版本即可。

### 通用 Agent Skills 安装工具，可选

已有 Node.js 和 `npx` 时，可使用第三方 [skills 工具](https://github.com/vercel-labs/skills)。它支持发现仓库根目录的 `SKILL.md`。以下命令同样需要仓库内容已上传：

```sh
npx skills add StarryJia/github-scout --skill github-scout --agent codex --global
```

在本仓库目录中，可直接安装本地文件：

```sh
npx skills add . --skill github-scout --agent codex --global
```

安装到其他兼容 Agent 时，按该工具支持的名称替换 `--agent codex`；也可复制到目标 Agent 文档指定的技能目录。技能格式兼容不代表所有 Agent 的发现目录和工具能力相同。自动选择取决于宿主模型和描述匹配，无法保证每次触发；显式调用可用于验证。

## 使用示例

安装后向 Codex 发出以下请求。它们是测试输入示例，不包含预先查询的统计结果。

```text
$github-scout 帮我调研 https://github.com/cli/cli
```

预期：获取真实数据，列出简介、仓库创建时间、语言、许可证及各项指标，标注查询时间和来源，用一两句话介绍用途、Stars 发展概览及提交活跃情况。

```text
对比 cli/cli 和 git/git 的热度及近期更新情况
```

预期：自动选择 Skill，以短英文表头的横向表格比较指标，日期单元格只写 `YYYY-MM-DD`；分别注明默认分支范围，在表外简短介绍用途及最近发布。

```text
推荐几个 Codex 多账号管理的开源项目
```

预期：先核实候选是否满足用途，再为最终入选项目补充统计；不把 Stars 高直接当作更适合的证据。

```text
这个仓库现在还有人在维护吗？https://github.com/octocat/Hello-World
```

预期：根据默认分支提交日期及归档状态给出活跃标签，说明提交情况不能单独证明维护意图。

普通 Git 操作、代码修改和 GitHub 工单管理不应无关地触发此 Skill。

```text
$github-scout 看看 sharkdp/bat 什么时候创建，最近 Stars 增长怎么样
```

预期：显示 GitHub 仓库创建日期，比较两个连续 4 周周期的新增 Stars，附历史接口来源；历史不可用或项目观察期不足时写未知，不推算增速。

### Output format

表头默认如下，单项目一行，多项目多行；说明文字仍使用中文：

```text
Repository | Created | Stars | New Stars | Growth | Forks | Commits | Contributors | Last Commit
```

`Created` 对应仓库的 `created_at`；`New Stars` 为最近 4 个完整统计周新增星标量，`Growth` 为相较前 4 周的新增量变化率，两者是 Skill 汇总指标。日期只写年月日，贡献者列只写记录数，统计口径、失败原因和来源集中在表外。例如，贡献者请求失败时单元格写 `Unknown`，不填长解释或 0。

Markdown 的实际列宽由显示界面决定，不能指定固定宽度；短表头和简短内容可减少撑宽，不用空格补齐列宽。

## 数据来源与查询方式

优先使用 GitHub 官方数据及已有连接工具；必要时通过 GitHub REST API（官方 HTTP 数据接口）查询。已有可用登录状态时可使用 GitHub CLI（命令行工具 `gh`）；没有登录状态时可匿名查询公开仓库，不强制要求 Token（认证令牌）。

| 数据 | 官方来源 |
| --- | --- |
| 简介、仓库创建时间、语言、许可证、Stars、Forks、默认分支、归档状态 | [仓库元信息](https://docs.github.com/en/rest/repos/repos#get-a-repository) |
| 按周统计的新增 Stars | [星标历史](https://docs.github.com/en/rest/activity/starring#get-repository-star-history)，公开仓库可匿名查询 |
| 默认分支提交数和提交者日期 | [提交列表](https://docs.github.com/en/rest/commits/commits#list-commits) |
| 账户及匿名作者记录 | [贡献者列表](https://docs.github.com/en/rest/repos/repos#list-repository-contributors) |
| 最近发布，包含预发布 | [发布列表](https://docs.github.com/en/rest/releases/releases#list-releases) |

每次查询标注实际时间和时区，结果附实际接口或仓库记录链接。星标历史只读取覆盖比较周期的周数据，默认排除可能未结束的最新周，不遍历所有用户；贡献者及发布记录沿响应头分页。提交计数用固定提交快照下每页一条的末页信息确认，无法采用该方法时完整翻页。详细执行步骤、分页预算和重试规则见 [SKILL.md](SKILL.md)。

## 测试方法

1. **发现与格式**：确认安装目录存在 `github-scout/SKILL.md`，文件开头元数据包含 `name: github-scout` 和有效 `description`；在 Codex 技能选择器确认可见。
2. **真实单项目**：运行上面的 `cli/cli` 请求。对照输出来源中的 `stargazers_count`、`forks_count` 及实际提交记录；所有数值应来自本次成功响应，不能来自此文档示例。
3. **多项目与自动选择**：不带 `$github-scout` 运行上面的两项目比较。应取得两组独立数据，以一张表格比较，不混用分支和查询时间。
4. **失败情形**：在 Agent 无网络的环境中查询同一仓库。应显示未获取并说明需要网络访问，不能把统计写成 0。测试用模拟 `403` 限流及贡献者第二页失败响应时，该项应显示未知，已经成功取得的指标应保留。
5. **规则边界**：使用明确标注为模拟的数据验证恰好 30 天、超过 30 天、恰好 180 天、超过 180 天及已归档情况；例如 30 天零 1 秒属于低频或近期更新较少。
6. **创建时间与 Stars 趋势**：运行上面的 `sharkdp/bat` 请求，核对 `created_at` 和星标历史的 `week`、`total`。模拟前一期为 0 时应只报绝对变化；缺周、分页失败或观察期不足时应写未知，不补零；模拟两个可比总量快照从 1,000 变为 1,080 时，净增为 80，不能与获星量混用。
7. **紧凑输出**：单项目和多项目都应使用上面的英文表头；日期不附长解释，Contributors 为纯数值，失败项为 `Unknown`，原因和指标含义保留在表外，不以压缩表格为由丢失统计口径。

如已具备 Agent Skills 的参考验证工具，可额外执行 `skills-ref validate /path/to/github-scout` 检查格式；这不是查询依赖，也不能替代实际行为测试。

## 项目限制

- Skill 需要宿主 Agent 的网络访问及读取响应能力，文件本身不会自动联网，也不会定时更新数据。
- Stars 和 Forks 会持续变化；分页请求不是所有指标同时取得的原子快照。GitHub 可能缓存部分数据。
- 仓库创建时间不等于项目诞生或首次开源时间。例如，旧项目迁入 GitHub 后，`created_at` 反映新仓库的创建日期。
- Stars 历史是 GitHub 统计周的获星数据，不等于总星标数净增长；准确净变化需要两个可比总量快照。比较周期为完整周，不是精确近 30 天，统计周边界也不保证与协调世界时对齐。
- 新仓库没有足够的完整比较周期时，只展示已有周数据。Stars 增长表示关注变化，不代表项目质量或维护状态。
- Contributors 输出的是接口记录数，不保证独立自然人数；可能包含机器人，并受匿名作者及前 500 个作者邮箱映射限制影响。例如，同一个人使用多个未关联邮箱时可能对应多条记录。
- Commits 默认只覆盖默认分支可达历史。例如，其他分支昨天有提交但默认分支 200 天未更新，默认标签仍为长期未更新，并注明范围。
- 活跃标签仅反映可观察提交情况。低频、长期未更新或最近有提交都不能单独证明维护意图、代码质量或安全性。
- 未识别许可证不能据此断言没有许可证；公开可访问也不必然具备开源使用授权。
- 限流、仓库过大、超过默认分页预算或工具响应截断时，部分指标会显示未知。无发布记录与发布查询失败会分别显示无 Release、未知。
- 默认最近发布包含预发布。通用安装工具为可选第三方工具；远端安装需要仓库可访问且已包含技能文件。

## 规范与许可证

文件采用 [Agent Skills 标准](https://agentskills.io/specification)，遵循 [Codex 技能发现及编写说明](https://learn.chatgpt.com/docs/build-skills)。根目录技能无需附加配置即可支持描述匹配的自动选择。

本项目使用 [MIT License](LICENSE)。
