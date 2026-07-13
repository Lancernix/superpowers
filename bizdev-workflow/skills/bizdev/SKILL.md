---
name: bizdev
description: 当用户要按需求做业务开发、给了钉钉/Confluence 需求文档链接、或说"开始业务开发/跑开发流程"时使用。一键串联 需求获取→澄清→计划→评审→派发→CR→验证，含人工闸门与 CR 闭环。
---

# BizDev Workflow（编排入口）

你是业务开发流程的编排者。按下面阶段顺序推进，每阶段调用对应子 skill/agent，并维护状态文件。

## 状态文件协议

- 路径：`~/.bizdev/<project-key>/<session>.md`，`project-key` = 仓库根目录 basename（如 `/Users/x/code/bizdev-workflow` → `bizdev-workflow`）。若存在同名仓库碰撞，手动在状态文件中修改路径。
- SDD 的任务级账本写姊妹文件 `~/.bizdev/<project-key>/<session>-ledger.md`（SDD 原生纯文本 append），本编排层读取它判断任务完成度。

### frontmatter 模板（状态文件完整结构）

```yaml
---
# 流程模式
mode: full | fast         # full=完整7步，fast=快速5步
# 流程状态
stage: intake | clarify | plan | review | dispatch | cr | verify | done   # 快速模式阶段集见下方快速模式阶段表
review_rounds: 0       # 计划评审打回次数（上限 2）；快速模式下恒为 0
cr_rounds: 0           # CR⇄执行闭环轮次（上限 2）；快速模式下恒为 0
# 产物路径（全部落在状态文件同目录 ~/.bizdev/<project-key>/，不进仓库）
requirements_raw: requirements-raw.md
requirements: requirements.md
plan: plan.md
review_report: review-report.md       # 快速模式下不生成
cr_report: cr-report.md               # 快速模式下不生成
verify_evidence: verify-evidence.md
# 已派发任务状态
tasks:
  - id: T1
    status: done | in_progress | blocked
    owner: general-purpose
---

```

### 读写指令

1. **读取**：读 `~/.bizdev/<project-key>/<session>.md` 全文，定位 `---` 分隔的 YAML frontmatter，解析其中字段。
2. **更新**：修改指定字段（如 `stage` → `review`、`review_rounds` → 1），**保持其余字段不变**，写回完整文件。不要整段覆盖 frontmatter——逐字段修改。
3. **新建**：若状态文件不存在/损坏，视为新会话，用上述模板创建（`stage: intake`，轮次归零，产物路径预填，`tasks: []`，`mode` 按当前评估结果写入）。
4. **启动检查**：先读状态文件。若存在且 `stage` 非初始态，从 `stage` 指示的阶段继续（断点续跑），不重跑已完成节点。
5. **mode 感知读取**：读取状态文件时，同时读取 `mode` 字段。若无 `mode` 字段（旧状态文件），视为 `mode: full`。
6. **mode 感知阶段流转**：根据 `mode` 决定下一阶段的映射：
   - full 模式：intake → clarify → plan → review → dispatch → cr → verify → done
   - fast 模式：intake → plan → dispatch → verify → done
   从 `stage` 跳转到对应 mode 下的下一阶段。


## 复杂度评估与模式路由

在开始任何阶段之前，先判断走哪条路径。

### 用户显式触发

如果用户说"简单需求"、"quick fix"、"小改动"、"快速模式"等，直接进入快速模式：

```
mode = fast
SKIP 复杂度评估
```

### 复杂度评估（仅在非显式触发时运行）

对需求描述做 4 维度评估，满足任意 2 项判定为「简单」：

| 维度 | 简单 | 复杂 |
|------|------|------|
| 来源 | 口头描述 / 短文本 | 钉钉/Confluence PRD 链接 |
| 提及文件数 | ≤3 个 | >3 个 或未指定 |
| 关键词 | 改文案、修 typo、加字段、调样式、改配置、修 bug、更新版本 | 重构、新模块、新接口、迁移、性能优化 |
| 预估影响 | 单一组件 / 单文件 | 跨模块 / 跨服务 |

评估规则：
- 统计需求描述中提及的文件路径/模块名数量 → 文件数维度
- 检查是否包含简化关键词 → 关键词维度
- 来源是链接还是口头/短文本 → 来源维度
- 根据描述推断影响范围 → 影响维度

若 score >= 2：判定为简单，向人建议快速模式：

> "此需求看起来比较简单（来源/范围/关键词），是否跳过澄清和计划评审，直接进入计划→执行→验证？"

- 人确认 → `mode = fast`
- 人否定 → `mode = full`

若 score < 2：`mode = full`

### 确定 mode 后的行为

1. 在新建状态文件时写入 `mode` 字段
2. 后续阶段根据 `mode` 选择对应流程表（见下方完整模式 / 快速模式）
3. `mode` 一旦写入不可更改；需切换模式则开启新会话
4. 断点续跑时读 `mode` 字段决定走哪条路径

## 阶段顺序与门控

1. **intake**：调用 `bizdev-workflow:bizdev-intake`（内部调 `bizdev-workflow:prd` agent）读取需求 → `requirements-raw.md`。失败则停在闸门1 前，提示人提供正确来源/配置 MCP。
2. **clarify**：调用 `bizdev-workflow:bizdev-clarify` → 产出待澄清清单。
   - 🔴 **闸门1**：`ask your human partner` 澄清问题，人答后定稿 `requirements.md`，进入下一阶段。
3. **plan**：调用 `bizdev-workflow:writing-plans` → 产出 `~/.bizdev/<project-key>/plan.md`。
4. **review**：调用 `bizdev-workflow:bizdev-plan-review`（内部派 `bizdev-workflow:plan-reviewer`）。
   - 不通过 → 回到 **plan** 重做；`review_rounds += 1`；`review_rounds >= 2` 仍不通过 → 升为 🔴人工闸门（直接问人是否强制继续/放弃）。
   - 通过 → 🔴 **闸门2**：向人展示计划摘要（计划文件路径、任务总数、关键依赖关系、已识别风险项、预计影响范围），`ask your human partner` 确认派发。人确认后进入 dispatch；人拒绝则停在闸门2 等修改指令。
5. **dispatch**：调用 `bizdev-workflow:subagent-driven-development` 逐任务执行。SDD 内部按 `implementer-prompt.md` 模板派 `general-purpose` subagent 实现每个任务（等效 `@fixer`）；运行时判断某任务需 specialist 则在派发 prompt 注明改用对应 specialist agent。默认逐任务串行；仅 plan 标注"可并行"的独立任务才并行派（见 Global Constraints）。
6. **cr**：调用 `bizdev-workflow:requesting-code-review` + 按 `bizdev-workflow:receiving-code-review` 对待反馈 → CR 报告（P0/P1/P2 分级）。
   - 含 P0/P1 → 回到 **dispatch**，让执行 agent 只修 CR 清单项（dispatch⇄CR 小闭环）；`cr_rounds += 1`；`cr_rounds >= 2` 仍含 P0/P1 → 升为 🔴人工闸门（闸门3 前介入，问人是否强制收尾）。
   - 仅 P2 → 进入 verify。
7. **verify**：调用 `bizdev-workflow:verification-before-completion` 跑测试/构建，产出 fresh 验证证据到 `verify-evidence.md`。
   - 🔴 **闸门3**：`ask your human partner` 确认收尾（证据已新）。确认后 `stage: done`。

### 快速模式阶段（mode = fast）

1. **intake**：调用 `bizdev-workflow:bizdev-intake`（快速分支，见 bizdev-intake SKILL.md）→ `requirements-raw.md`。
2. **plan**：调用 `bizdev-workflow:writing-plans` → 产出 brief plan（任务数 ≤3，无 formal 分解）→ `~/.bizdev/<project-key>/plan.md`。**不调用 plan-reviewer**。
   - 🔴 **闸门2**：向人展示简略计划摘要（计划路径、任务数、预估影响文件），`ask your human partner` 确认派发。
3. **dispatch**：调用 `bizdev-workflow:subagent-driven-development` 逐任务执行。
4. **verify**：调用 `bizdev-workflow:verification-before-completion` 产出 fresh 验证证据。
   - 🔴 **闸门3**：`ask your human partner` 确认收尾。确认后 `stage: done`。

**跳过的阶段**：clarify（闸门1）、review（plan-reviewer）、cr（dispatch⇄CR 闭环）

## 规则

- 只做编排与状态管理；具体逻辑在子 skill/agent。
- 任何子步骤失败/超时：在状态文件标记对应任务 `blocked`，停在该阶段等人工介入，不静默跳过。
- 轮次超限一律升人工闸门，绝不死循环。
- 状态文件缺失/损坏：视为新会话从头跑。
- 派发执行（步骤⑤）：fixer = `general-purpose` + `implementer-prompt.md` 模板，非具名 agent；并行仅限 plan 标注且无共享状态的任务，否则串行。
