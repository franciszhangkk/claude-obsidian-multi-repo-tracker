---
description: 在 Obsidian vault 里新增一个项目。自动检测 vault（不存在则 inline 初始化）、分析代码结构生成架构草稿、写入 CLAUDE.md。通常是接触新项目时唯一需要运行的命令。
allowed-tools: [Bash, Read, Write, Edit, Glob, AskUserQuestion]
---

# /add-project — 添加项目到 vault（含代码分析）

用法：`/add-project <项目名> <代码仓库路径>`

例如：`/add-project ai-agents ~/code/ai-agents`

> 这是新建项目时唯一需要调的命令。Vault 不存在会自动初始化，代码结构会自动分析生成架构草稿。

## 执行流程

### 1. 解析参数

- **项目名**：用户传的第一个参数。没传则 AskUserQuestion
- **代码路径**：用户传的第二个参数。没传则 AskUserQuestion，给出常见候选：
  - `~/code/<项目名>` / `~/go/src/<项目名>` / `~/projects/<项目名>` / 自定义

校验：
- 项目名不能含 `/` 或空格
- 代码路径必须存在且是目录（`[ -d "$path" ]`）

### 2. 检测 vault，不存在则 inline 初始化

```bash
VAULT_PATH="${OBSIDIAN_VAULT_PATH:-}"
```

**情况 A：vault 路径已配置且存在** → 直接继续

**情况 B：vault 路径未配置 / 不存在** → 用 AskUserQuestion 问：

```
未找到 Obsidian vault。选择：
  [1] 指定 vault 路径（已有 vault）
  [2] 新建 vault（我会帮你初始化基础结构）
  [3] 取消
```

选 2 时：**直接执行 `/init-vault` 的全部流程**（步骤 1-7），然后继续当前流程。用户无需退出重来。

### 3. 找到 skill 模板

```bash
SKILL_DIR="$HOME/.claude/skills/claude-obsidian-multi-repo-tracker"
TEMPLATE_DIR="$SKILL_DIR/templates/project"
```

### 4. 检查目标项目目录

```bash
PROJECT_DIR="$VAULT_PATH/项目/<项目名>"
```

- 已存在 → AskUserQuestion：覆盖 / 换名字 / 取消
- 不存在 → 创建

### 5. 复制项目模板

复制 `templates/project/` 到 `<vault>/项目/<项目名>/`：
`概览.md` / `activeContext.md` / `progress.md` / `01-架构 ~ 05-测试与调试` 各目录

### 6. 检测语言和依赖

```bash
# 语言检测（优先级顺序）
[ -f go.mod ]           → Go，读 go.mod require 取主要依赖
[ -f package.json ]     → Node.js/TypeScript，读 dependencies
[ -f pyproject.toml ]   → Python，读 [tool.poetry.dependencies] 或 [project]
[ -f requirements.txt ] → Python，读前 5 行
[ -f Cargo.toml ]       → Rust，读 [dependencies]
都没有                  → "TODO: 待填写"
```

### 7. 扫描代码结构，生成架构草稿

**目标**：自动填充 `概览.md` 的"架构速览"和"关键模块"，让用户拿到手就有内容可改，而不是空白占位符。

#### 7a. 扫目录树

```bash
find "$CODE_ROOT" -maxdepth 3 -type d \
  ! -path "*/.*" \
  ! -path "*/node_modules/*" \
  ! -path "*/__pycache__/*" \
  ! -path "*/vendor/*" \
  | sort
```

#### 7b. 按语言识别关键模块

**Go**：
```bash
# 识别有意义的包目录（含 .go 文件）
find "$CODE_ROOT" -maxdepth 3 -name "*.go" | xargs dirname | sort -u
# 读每个主目录下第一个 .go 文件的 package 注释
```
重点看：`cmd/` `biz/` `pkg/` `internal/` `api/` `service/` `handler/`

**Python**：
```bash
# 找主文件
ls "$CODE_ROOT"/*.py 2>/dev/null
# 找子包
find "$CODE_ROOT" -name "__init__.py" -maxdepth 3 | xargs dirname
```
重点看：`app.py` `main.py` `server.py` `cli.py`

**Node.js/TS**：
```bash
# 读 package.json scripts
cat "$CODE_ROOT/package.json" | python3 -c "import json,sys; d=json.load(sys.stdin); print('\n'.join(d.get('scripts',{}).keys()))"
# 找主目录
ls "$CODE_ROOT/src/" 2>/dev/null || ls "$CODE_ROOT/lib/" 2>/dev/null
```

**通用**：读 `README.md` 第一段（通常是最准确的项目描述）

#### 7c. 生成草稿

基于上面信息，为 `概览.md` 的以下两段生成草稿：

**架构速览**（1-2 句话，说清楚核心架构）：
- 有 `cmd/` + `biz/` + `pkg/` → "分层架构：cmd 入口 / biz 业务逻辑 / pkg 基础库"
- 有 `app.py` + `server.py` → "本地 Python 服务 + 前端静态页面"
- 有 `src/` + `package.json` → "前端应用，webpack/vite 构建"
- 读 README 第一段 → 直接作为基础

**关键模块**（表格，top 3-5 条）：
每个模块：路径 + 职责（从目录名 + 包注释 + README 推断）

#### 7d. 展示草稿让用户确认

```
代码结构分析完成，以下是自动生成的草稿：

── 架构速览 ──────────────────────────────
分层架构：cmd/ 入口 → biz/ 业务（agent/bi/ad/insight）→ pkg/ 基础库

── 关键模块 ──────────────────────────────
biz/agent/ad/    广告操作 Agent（SKILL.md 拆分子场景）
biz/agent/bi/    BI 数据分析
biz/agent/base/  Agent 框架基础设施
pkg/             公共基础库

→ 确认使用 / 输入修改 / s 跳过（手动填）
```

用 AskUserQuestion，选项：确认 / 自定义 / 跳过

确认后写入 `概览.md` 对应段落，跳过则保留空白占位符。

### 8. 问"做什么"，填项目速描

```
这个项目做什么？一句话：谁在用、解决什么业务问题。
（例："服务广告投放的 Agent 系统，让操作员通过自然语言完成投放和查询"）
不要只写技术栈——业务定位才是 Claude 开工时最需要的。
```

### 9. 替换占位符

| 占位符 | 替换为 |
|--------|--------|
| `{{vault_path}}` | vault 绝对路径 |
| `{{项目名}}` | 用户输入的项目名 |
| `{{语言}}` | 第 6 步检测 |
| `{{框架/库}}` | 第 6 步检测 |
| `{{做什么}}` | 第 8 步用户输入 |
| `{{related_projects_block}}` | 见下 |

**`{{related_projects_block}}`**：读 vault 首页其他项目，生成关联项目列表。首次添加则写"暂无关联项目"。

### 10. 在代码仓库根写入 CLAUDE.md（如不存在）

从 `templates/project/CLAUDE.md.template` 复制并替换占位符。已存在则跳过。

文档地图段保持模板默认（`_暂无文档_`），后续靠 `/sync-docs` 刷新。

### 11. 更新 vault 首页 + CLAUDE.md 映射表

追加项目行到首页表格和 vault CLAUDE.md 映射表。已存在则更新不重复追加。

### 12. 输出总结

```
✅ 项目 <项目名> 已添加到 vault

vault：<vault>/项目/<项目名>/
代码：<代码路径>
语言：<语言> / 主要依赖：<框架/库>

已生成：
  - 概览.md ← 架构速览 + 关键模块已自动填充
  - activeContext.md（当前焦点待填，可用 /update-active 推断）
  - progress.md
  - 5 个分类目录

代码仓库：
  - CLAUDE.md ← v0.5 结构（路由表 + 项目速描）

下一步：
  - 检查 概览.md 的架构速览是否准确，按需修改
  - 用 /update-active 更新当前焦点
  - 开始开发，提交时用 /commit
```

## 错误处理

- 代码路径不存在 → 报错
- 项目名含非法字符 → 给示例
- 目录扫描权限不足 → 跳过该目录，不中断
- 代码结构无法识别 → 架构草稿跳过，保留空白（不影响其他步骤）

## 注意

- 不要假设代码仓库是 git repo（tapmeme 等工具项目可能没有）
- 不要往代码仓库写除 `CLAUDE.md` 以外的文件
- monorepo 场景（多语言标志同时存在）：列出检测到的所有语言让用户确认
- 架构草稿是辅助，不是真相——用户确认后才写入，不强制覆盖
