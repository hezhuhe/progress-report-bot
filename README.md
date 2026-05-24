# progress-report-bot

> 面向 **Cursor / Claude Code / CLI 自动化** 的飞书项目（Meego）进度对账工具。  
> 自动生成版本进度周报、识别飞书状态与 Git 实际进展差异，并可按需回写飞书评论。

默认安全：所有命令先生成本地 `data/*.md`；只有显式 `--apply` 才会写回飞书。

---

## 这个工具适合谁

- 项目经理 / 交付经理：想快速拿到“老板视角”进度周报  
- 研发负责人：想核验飞书状态与代码真实推进是否一致  
- 团队成员：想减少人工周报汇总成本  
- 平台/效能团队：想把项目进度分析纳入定时任务或自动化流程

---

## 核心功能

- 飞书工作项采集（mine / project / all 三种范围）
- Git 进度增强（本地多仓、GitLab、GitHub）
- 对账分析（假完成 / 状态滞后 / 节点停滞 / 分支不存在）
- 报告渲染（`report*.md` + `diff*.md`）
- 可选飞书回写评论（`--apply`）
- 可选节点流转同步（`sync --apply`）

---

## 亮点与优势

- **证据化管理**：不是“口头进度”，而是“飞书状态 + Git 活动证据”联合判断
- **多仓友好**：支持一个工作目录下多个 git 仓库自动扫描
- **低门槛接入**：最小化配置，默认本地输出，不侵入业务系统
- **可自动化运行**：支持 `--workspace`，可直接做 cron/定时任务
- **安全可控**：写飞书需要明确 `--apply`，避免误操作

---

## 自然语言触发示例（Agent 场景）

下列表达都可触发 skill：

- “生成版本进度同步”
- “同步版本进度”
- “看下版本进度”
- “跑一下飞书项目周报”
- “看飞书和代码进度是否一致”
- “给我一版老板视角周报”
- “PR 合并到 test 后帮我同步节点”

---

## 安装（作为 Skill）

### 1) 克隆仓库

```bash
git clone https://github.com/1920570209/progress-report-bot.git
cd progress-report-bot
```

### 2) 执行安装脚本

**Windows (PowerShell)**

```powershell
./scripts/install-skill.ps1
```

**macOS / Linux**

```bash
chmod +x ./scripts/install-skill.sh
./scripts/install-skill.sh
```

安装脚本会做两件事：

1. `pip install -e .`（使 `progress-report-bot` 命令全局可用）
2. 把 skill 链接到 Cursor/Claude 目录：
   - `~/.cursor/skills/progress-report-bot`
   - `~/.claude/skills/progress-report-bot`

> 若想“项目级安装”（随仓库共享），可用：  
> `./scripts/install-skill.ps1 -ProjectScope` 或 `./scripts/install-skill.sh --project`

### 3) 重启 Cursor / Claude

重启后，Agent 会读取本仓库 `SKILL.md` 并可按自然语言触发。

---

## 快速开始（推荐）

### A. 对话式（Cursor / Claude）

直接在目标工作目录里说：  
“生成版本进度同步（老板视角）”

Agent 会引导你仅输入一次 `MEEGO_MCP_TOKEN`，其余配置通过对话选项完成。

### B. CLI 手工运行

```bash
# 首次可用 init（人工模式）
progress-report-bot init

# 只生成本地报告（安全）
progress-report-bot run-all
```

---

## 工作区（workspace）机制

默认用当前目录作为工作区（读取 `.env`，输出 `data/*`）。

当你想做通用脚本/定时任务，建议显式指定：

```bash
progress-report-bot run-all --workspace /path/to/workspace --scope project
```

也支持交互选择：

```bash
progress-report-bot run-all --choose-workspace
```

如果当前目录不是 git 工作区，交互运行时会提示你输入工作目录路径。

---

## 主要命令

### 发现与配置

```bash
progress-report-bot ping
progress-report-bot projects --json
progress-report-bot types --project-key <key> --json
progress-report-bot carriers --project-key <key> --json
progress-report-bot repos
progress-report-bot fetch-repos
```

### 报告与对账

```bash
progress-report-bot fetch
progress-report-bot report
progress-report-bot diff
progress-report-bot run-all
```

### 写回与同步（有副作用）

```bash
progress-report-bot push --apply
progress-report-bot sync --apply
```

> 建议先 dry-run（不加 `--apply`）确认结果后再执行。

---

## Scope 与输出文件命名

| Scope | 含义 | 周报文件 | 对账文件 |
|---|---|---|---|
| `mine` | 仅本人工作项 | `data/report.md` | `data/diff.md` |
| `project` | 全项目（老板视角） | `data/report-boss.md` | `data/diff-boss.md` |
| `all` | 本人+全项目合并去重 | `data/report-all.md` | `data/diff-all.md` |

示例：

```bash
progress-report-bot run-all --scope project --workspace /path/to/ws
progress-report-bot diff --scope project --use-cache --workspace /path/to/ws
```

---

## 定时任务示例（cron）

每周一早上 9:00 生成老板视角周报：

```cron
0 9 * * 1 /usr/bin/env progress-report-bot run-all --scope project --workspace /path/to/workspace >> /path/to/workspace/data/cron.log 2>&1
```

---

## 配置说明（最小 .env）

`.env` 放在工作区目录（默认当前目录或 `--workspace` 指定目录）：

```env
MEEGO_MCP_URL=https://project.feishu.cn/mcp_server/v1
MEEGO_MCP_TOKEN=m-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
MEEGO_PROJECT_KEY=your-feishu-project-key
DEFAULT_SCOPE=mine
GIT_PROVIDER=local
LOCAL_GIT_REPO_ROOT=/path/to/your/workspace
LOCAL_GIT_REMOTE_PREFIX=origin/
```

完整字段见 [reference.md](reference.md)。

---

## 安全规则（强烈建议）

1. 不确认就不要加 `--apply`  
2. 先看本地 `report*.md` / `diff*.md` 再决定是否回写  
3. 定时任务优先使用 `--workspace`，避免上下文漂移

---

## 卸载

```bash
rm ~/.cursor/skills/progress-report-bot
rm ~/.claude/skills/progress-report-bot
pip uninstall progress-report-bot
```

Windows 用 `Remove-Item` 删除 junction。

---

## 参考文档

- Skill 触发与 Agent 约束：[`SKILL.md`](SKILL.md)
- 详细配置和对账规则：[`reference.md`](reference.md)

---

## License

MIT — see [LICENSE](LICENSE).
