---
description: 在 Obsidian vault 里新增一个项目，生成模板目录并在代码仓库里写入引用
allowed-tools: [Bash, Read, Write, Edit, Glob, AskUserQuestion]
---

# /add-project — 添加一个项目到 vault

用法：`/add-project <项目名> <代码仓库路径>`

例如：`/add-project ai-agents ~/code/ai-agents`

## 执行流程

### 1. 解析参数

- **项目名**：用户传的第一个参数。如果没传，用 AskUserQuestion 问
- **代码路径**：用户传的第二个参数。如果没传，用 AskUserQuestion 问，给出常见候选：
  - `~/code/<项目名>`
  - `~/go/src/<项目名>`
  - `~/projects/<项目名>`
  - 自定义

校验：
- 项目名不能含 `/` 或空格（Obsidian 文件夹名）
- 代码路径必须存在且是目录（用 `[ -d "$path" ]`）

### 2. 找到 vault 路径

按这个优先级：
1. `$OBSIDIAN_VAULT_PATH` 环境变量
2. 找不到 → 用 AskUserQuestion 问，并提醒用户运行 `/init-vault` 先

### 3. 找到 skill 模板

```bash
SKILL_DIR="$HOME/.claude/skills/claude-obsidian-multi-repo-tracker"
TEMPLATE_DIR="$SKILL_DIR/templates/project"
```

### 4. 检查目标项目目录

```bash
PROJECT_DIR="$VAULT/项目/<项目名>"
```

- 已存在 → 用 AskUserQuestion 问：覆盖 / 用其他名字 / 取消
- 不存在 → 创建

### 5. 复制项目模板

将 `templates/project/` 下所有文件复制到 `<vault>/项目/<项目名>/`：

- `概览.md`
- `activeContext.md`
- `progress.md`
- `01-架构/README.md`
- `02-功能/README.md`
- `03-产品/README.md`
- `04-参考资料/README.md`
- `05-测试与调试/README.md`

### 6. 替换占位符

用 Edit 把模板里的 `{{项目名}}` 全部替换成实际项目名。

```bash
# 例如对每个新建的 .md 文件
sed -i '' "s/{{项目名}}/<实际项目名>/g" "<file>"
```

字段：
- `{{项目名}}` → 用户输入的项目名
- `{{代码路径}}` → 代码仓库路径（如 `~/code/ai-agents`）
- `{{语言}}` → 自动检测（见下）
- `{{框架/库}}` → 自动检测（见下）

### 7. 自动检测代码仓库元信息

进入代码路径，自动识别：

- **语言**：
  - 有 `go.mod` → Go
  - 有 `package.json` → Node.js / TypeScript（看 `tsconfig.json`）
  - 有 `pyproject.toml` / `requirements.txt` → Python
  - 有 `Cargo.toml` → Rust
  - 都没有 → 留 "TODO: 待填写"
- **框架/库**：从上述配置文件读主要依赖（前 3-5 个）

如果检测不到就留空，让用户手动填。

### 8. 引导用户填写项目速描

用 AskUserQuestion 向用户提问（这是 CLAUDE.md 里最重要的一行，不能留空白）：

```
这个项目做什么？请用一句话描述：谁在用它、解决什么业务问题。
（例："服务广告投放的 Agent 系统，让操作员通过自然语言完成投放和查询"）
不要只写技术栈——业务定位才是 Claude 开工时最需要的。
```

将用户回答存为 `{{做什么}}` 占位符的值，用于下一步写 CLAUDE.md。

如果用户不想填，写默认值 `_（待补充：谁在用、解决什么问题）_`。

### 9. 在代码仓库根写入 CLAUDE.md（如果没有）

**改为从模板文件读取**，不再内联生成：

```bash
CODE_CLAUDE_MD="<代码路径>/CLAUDE.md"
TEMPLATE_FILE="$SKILL_DIR/templates/project/CLAUDE.md.template"

if [ -f "$CODE_CLAUDE_MD" ]; then
  echo "已存在 CLAUDE.md，跳过"
else
  cp "$TEMPLATE_FILE" "$CODE_CLAUDE_MD"
fi
```

然后用 Edit 替换占位符：

| 占位符 | 替换为 |
|--------|--------|
| `{{vault_path}}` | vault 绝对路径（如 `/Users/x/Desktop/Obsidian Vault`） |
| `{{项目名}}` | 用户传的项目名 |
| `{{语言}}` | 第 7 步检测出的语言 |
| `{{框架/库}}` | 第 7 步检测出的依赖 |
| `{{related_projects_block}}` | 关联项目列表（见下） |

**`{{related_projects_block}}` 生成**：

读 vault `首页.md` 的项目列表，**除当前项目外的其他项目**，每条生成一行：

```markdown
- <项目名>（<一句话角色，从首页表格的"角色"列取，没有就留空>）→ Obsidian: `项目/<名>/`
```

如果首页没有其他项目（首次添加），整个 block 写：

```markdown
- _（暂无关联项目，添加更多项目后此处会自动更新）_
```

模板里的"Obsidian 文档地图"段保持模板默认（占位 _暂无文档_），后续靠 `/sync-docs` 刷新——不要在 add-project 阶段就扫，因为新项目还没文档。

### 9. 更新 vault 首页的项目列表

读 `<vault>/首页.md`，找到 `<!-- 此区域由 /add-project 自动维护 -->` 标记下方的表格，追加一行：

```markdown
| <项目名> | `<代码路径>` | [[项目/<项目名>/概览]] | 进行中 |
```

如果项目名已在表里，**不要重复追加**（更新行）。

### 10. 更新 vault CLAUDE.md 的映射表

同样找标记区域，追加一行到映射表。

### 11. 输出总结

```
✅ 项目 <项目名> 已添加到 vault

vault 路径：<vault>/项目/<项目名>/
代码路径：<代码路径>
检测到语言：<语言>
检测到主要依赖：<框架/库>

已生成：
  - 概览.md（已填代码路径和检测到的语言/依赖）
  - activeContext.md（空模板，请手动填写当前焦点）
  - progress.md（空模板）
  - 5 个分类目录（01-架构 ~ 05-测试与调试）

已更新：
  - 首页.md 项目列表
  - vault CLAUDE.md 项目映射

代码仓库：
  - <代码路径>/CLAUDE.md  ← 已写入引用 vault 文档的指令

下一步：
  - 编辑 概览.md 补全"一句话定位"和"架构速览"
  - 编辑 activeContext.md 填写当前在做什么
  - 用 cd <代码路径> 进入代码仓库继续工作
```

## 错误处理

- vault 不存在 → 提示先运行 `/init-vault`
- 代码路径不存在 → 报错并提示
- 项目名格式错误 → 给出合规示例
- 占位符替换失败 → 报告但不中断（用户可手动改）

## 注意

- 不要假设代码仓库已经是 git repo（可能还没 init）
- 不要往代码仓库里写除 `CLAUDE.md` 以外的文件
- 检测语言时，多个标志同时存在（例如 monorepo）时，优先按用户传入的项目类型判断，或者列出所有检测到的，让用户确认
