以下是一套成熟的 **Issue 全生命周期管理流程（SOP）**，涵盖了从提出到关闭的 5 个阶段，以及 6 种常见分支场景的处理。

---

### 核心状态流转图 (Status Lifecycle)

首先定义 Issue 的生命状态，建议使用 GitHub Labels 对应：

`New` (新提出) ➡️ `Triage` (分拣中) ➡️ `Confirmed` (确认是Bug) ➡️ `In Progress` (修复中) ➡️ `Done/Fixed` (已修复) ➡️ `Closed`

---

### 第一阶段：拦截与标准化 (Pre-Triage)
**目标：** 阻止无效 Issue，确保有效 Issue 信息齐全。

1.  **强制模版 (Issue Forms):**
    *   使用 `.github/ISSUE_TEMPLATE/bug_report.yml`。
    *   **必填项：** AstrBot 版本号、部署方式（Docker/源码）、使用的 LLM 模型、使用的 IM 平台（QQ/微信/TG）、**完整的日志截图/文本**（不填直接无法提交）。
2.  **自动打标 (Auto Labeling):**
    *   利用 GitHub Actions 根据用户勾选的选项自动打标：如 `platform: wechat` 或 `provider: openai`。

---

### 第二阶段：人工分拣 (Manual Triage)
**目标：** 在 24-48 小时内对 Issue 定性。这是最关键的一步，决定了 Issue 的去向。

**分拣员 (Triager) 的判决路径：**

#### 场景 A：信息缺失 (The "It doesn't work" Report)
*   **现象：** 用户只说“报错了”，没贴日志，或者没说环境。
*   **操作：**
    1.  打标 `needs-more-info`。
    2.  Bot 自动回复：“请提供日志和复现步骤，否则无法排查。”
    3.  **后续：** 如果 7 天内用户未回复，Bot 自动关闭 (Stale)。

#### 场景 B：非项目 Bug / 使用问题 (Question/Support) （按照我们 @想法管理 的设计，不会出现这种情况）
*   **现象：** “Key 怎么填？”、“Docker 怎么安装？”、“为什么 LLM 回答我不准？”
*   **操作：**
    1.  打标 `type: question` 或 `invalid`。
    2.  Bot 回复：“此类问题请移步 Discussions 或查看文档 [链接]。”
    3.  **动作：** 直接 **Close**，或使用 "Convert to Discussion" 功能将其移出 Issue 列表。

#### 场景 C：上游依赖问题 (Upstream Issue) —— **AstrBot 高频场景**
*   **现象：** “QQ 登录不上（协议库挂了）”、“DeepSeek 接口 502”。
*   **操作：**
    1.  打标 `upstream` (上游问题)。
    2.  回复：“这是 OneBot/OpenAI 的问题，非 AstrBot 自身 Bug，请关注上游仓库 [Link]。”
    3.  **动作：** **Close**。不要让 AstrBot 为上游背锅，否则 Issue 会积压成千上万。

#### 场景 D：重复 Bug (Duplicate)
*   **现象：** 多人反馈同一个崩溃。
*   **操作：**
    1.  找到最早/信息最全的那个 Issue 作为“主 Issue”。
    2.  在当前 Issue 回复：“Duplicate of #123”。
    3.  打标 `duplicate`。
    4.  **动作：** **Close**。

#### 场景 E：有效 Bug (Valid Bug)
*   **现象：** 复现成功，确定是代码逻辑错误。
*   **操作：**
    1.  打标 `type: bug`。
    2.  评估优先级：`p0-critical` (崩溃/阻断性), `p1-high` (核心功能异常), `p2-medium` (边缘功能), `p3-low` (UI/文案)。
    3.  **进入下一阶段。**

---

### 第三阶段：开发与修复 (Development)
**目标：** 追踪修复进度，避免撞车。

1.  **认领 (Assignment):**
    *   **核心团队修：** Assign 给对应模块的维护者。
    *   **社区修：** 打标 `help wanted` 或 `good first issue`。
    *   **社区认领规则：** 贡献者在评论区输入 `/claim`，Bot 自动 Assign 给他（防止多人同时修同一个 Bug）。
2.  **开发中 (In Progress):**
    *   状态流转为 `status: in-progress`。
    *   开发者提交 PR，并在 PR 描述中关联 Issue（使用关键词 `Fixes #123`）。

---

### 第四阶段：验收与关闭 (Verification & Closure)
**目标：** 确保修复有效，且不引入新 Bug。

1.  **自动化关闭 (Auto-Close):**
    *   当 PR 被合并进 `main` 分支，关联的 Issue 会自动关闭。
    *   **注意：** 此时 Issue 虽然 Closed，但用户可能还没用上修复版。
2.  **版本里程碑 (Milestones):**
    *   合并 PR 时，将 PR 和 Issue 加入同一个 Milestone（如 `v2.5.0`）。
    *   当发布 Release `v2.5.0` 时，Bot 会自动在所有该 Milestone 的 Issue 下评论：“此修复已在 v2.5.0 发布，请更新。”——**这一步用户体验极佳。**

---

### 第五阶段：僵尸清理 (Stale Management)
**目标：** 维护 Issue 列表的“新鲜度”。

*   **配置 Stale Bot (GitHub Actions):**
    *   **规则：** `type: bug` 且 60 天无活动的 Issue。
    *   **动作 1：** 标记 `stale`，评论：“此 Issue 很久没有活动了，如果是已修复或不再重现，请忽略；如果问题依旧，请评论。”
    *   **动作 2：** 再过 14 天无回复，**Close** 并标记 `completed`（或 `not-planned`）。

---

### 针对 AstrBot 的特殊策略 (Summary for Your Project)

在上述通用流程中，你需要特别强化 **分拣阶段（Stage 2）** 的话术库：

1.  **协议甩锅话术：** “经检测这是 IM 协议端的风控拦截，AstrBot 无法解决风控问题，请尝试更换协议库或挂起账号。” -> 对应标签 `cant-fix` + `upstream`。
2.  **配置错误拦截话术：** “这是 YAML 格式错误，请检查缩进。” -> 对应标签 `invalid`。

### 最终产出：Issue 状态流转清单

你可以将这个清单做成 `CONTRIBUTING.md` 的一部分：

1.  🟢 **Open (New):** 等待分拣。
2.  🟡 **Open (Needs Info):** 等待用户补充信息（Bot 倒计时中）。
3.  🔴 **Open (Confirmed):** 确认是 Bug，等待修复。
4.  🔵 **Open (In Progress):** 正在修复中，已有 PR 关联。
5.  🟣 **Closed (Fixed):** 代码已合并，等待发版。
6.  ⚪ **Closed (Stale/Invalid):** 过期或无效。

这套流程能确保核心团队只关注那 **20% 的有效 Bug**，而将 80% 的无效/重复/上游问题通过 Bot 和规范快速过滤掉。