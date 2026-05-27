# 📂 Vault Watcher

[🌐 English](./README_EN.md) | 简体中文

**Obsidian 笔记库文件变更实时监控 → 飞书/Lark 群通知**

当你的 Obsidian 笔记库中有文件发生变化时，自动推送一张结构化的飞书消息卡片到指定群聊，告诉你**哪些文件变了、怎么变的**。

---

## 解决什么问题？

如果你和我一样，有一个 AI Agent（比如 CoPaw）在后台帮你整理笔记、归档文件、更新待办——你会遇到一个问题：

> **AI 说"帮你改好了"，但改了什么、改了哪里，你完全不知道。**

Vault Watcher 就是为了解决这个问题：

- ✅ AI 往你 Inbox 里写了新文件 → 你收到通知
- ✅ AI 修改了你的待办汇总 → 你收到通知 + **具体修改了哪几行**
- ✅ 文件被归档了 → 你知道从哪移到了哪
- ✅ 文件被删了 → 你也知道

---

## 🚀 AI 一键安装

> [!TIP]
> **复制下面这段话，直接发给你的 AI Agent（Claude Code / Cursor / Codex 等），它会自动帮你完成全部安装和配置。**
>
> ```
> 帮我安装 vault-watcher。
> 这是一个 Obsidian 笔记库文件变更监控工具，会把变更推送到飞书群。
> 项目地址：https://github.com/manwithshit/vault-watcher
> 请引导我完成配置（vault 路径、监控目录、飞书凭证），然后一键部署。
> ```
>
> 如果你的 Agent 已安装了 vault-watcher skill，它会自动识别并进入引导式安装流程。
> 如果没有安装 skill，把上面这段话发给它，它也能参照 README 手动完成安装。

---

## 效果预览

飞书群里收到的消息卡片长这样：

```
┌─────────────────────────────────────────┐
│ 📂 Vault 变更 · 14:30 · 3 项            │
├─────────────────────────────────────────┤
│ ➕ 新增 (1)                              │
│ • Inbox/260527_待办_明早买早饭.md         │
├─────────────────────────────────────────┤
│ ✏️ 修改 (1)                              │
│ • Inbox/[待办汇总].md                    │
├─────────────────────────────────────────┤
│ 📋 待办汇总 · 变更详情                    │
│ +1行 / -0行                              │
│                                         │
│ 🟢 Vault Watcher 巡检机制上线            │
│    [260526 已完成]（launchd + fswatch    │
│    监控变更推送飞书卡片通知）              │
├─────────────────────────────────────────┤
│ 📦 归档 (1)                              │
│ • old-todo.md                           │
│   ↳ Inbox/ → archive/                  │
├─────────────────────────────────────────┤
│ ⏱️ 14:30:25 · Vault Watcher · launchd   │
└─────────────────────────────────────────┘
```

---

## 核心特性

| 特性 | 说明 |
|------|------|
| **5 类事件** | 新增 / 修改 / 归档 / 删除 / **Diff 详情** |
| **Diff 追踪** | 对指定的高频文件，展示具体新增/删除了哪些行 |
| **智能去重** | 短时间内多次修改合并为一条通知（可配置窗口） |
| **归档识别** | 区分"被移到归档目录"和"真删除" |
| **iCloud 降噪** | 自动过滤 `.icloud` 占位符、临时文件等噪声 |
| **零依赖** | 仅需 `fswatch`（brew）+ Python 3 + bash 3.2 |
| **崩溃自愈** | macOS `launchd` 托管，挂了自动重启 |
| **隐私安全** | 纯本地运行，只有文件路径发送到你自己的飞书 Bot |

---

## 工作原理

```
┌──────────────────────────────────────────────────┐
│  macOS launchd（开机自启 · 崩溃重启）             │
│                                                  │
│  ┌──────────┐   ┌─────────────┐   ┌───────────┐ │
│  │ fswatch  │──▶│ watcher.sh  │──▶│ notify.py │ │
│  │(8s 收集) │   │(10s 去抖    │   │(飞书 Bot  │ │
│  │          │   │ +分类+diff) │   │ API 发卡) │ │
│  └──────────┘   └─────────────┘   └───────────┘ │
└──────────────────────────────────────────────────┘
        │                                    │
        ▼                                    ▼
  Obsidian 笔记库                    飞书/Lark 群聊
  (本地 Markdown 文件)              (Interactive Card)
```

**事件分类逻辑：**

| 事件 | 怎么判断的 |
|------|-----------|
| ➕ 新增 | 文件存在 + 不在已知文件快照中 |
| ✏️ 修改 | 文件存在 + 在快照中 |
| 📦 归档 | 文件消失 + 在归档目录里找到了同名文件 |
| 🗑️ 删除 | 文件消失 + 归档目录也没有 |
| 📋 Diff | 对指定文件，维护一份"影子副本"做对比 |

---

## 环境要求

- **macOS**（使用 `launchd` 和 `fswatch`）
- **fswatch**：`brew install fswatch`
- **Python 3**：macOS 自带
- **飞书/Lark 自建应用**：需要有 `im:message` 权限的 Bot

---

## 手动安装

> 推荐使用上方的 **AI 一键安装**，以下为手动安装步骤。

### 1. 下载

```bash
git clone https://github.com/manwithshit/vault-watcher.git
cd vault-watcher
```

### 2. 配置

编辑 `vault-watcher.sh`，设置你的 vault 路径和监控目录：

```bash
VAULT_ROOT="$HOME/path/to/your/obsidian/vault"
WATCH_DIRS=(
  "${VAULT_ROOT}/Inbox"
  "${VAULT_ROOT}/Sources"
  "${VAULT_ROOT}/Reference"
)
ARCHIVE_DIR="${VAULT_ROOT}/Archive"

# Diff 追踪：设置你最关心的那个高频文件
DIFF_TARGET="${VAULT_ROOT}/Inbox/todo-summary.md"
```

设置飞书凭证（二选一）：

**方式 A：环境变量**（推荐）

```bash
export FEISHU_APP_ID="cli_xxxxx"
export FEISHU_APP_SECRET="your_secret"
export FEISHU_CHAT_ID="oc_xxxxx"
```

**方式 B：直接编辑 `vault-notify.py`**

### 3. 安装

```bash
chmod +x vault-watcher.sh vault-notify.py

# 复制到固定位置
mkdir -p ~/.vault-watcher
cp vault-watcher.sh vault-notify.py ~/.vault-watcher/

# 安装 launchd 配置
cp com.vault-watcher.plist ~/Library/LaunchAgents/
# 修改 plist 里的脚本路径
sed -i '' "s|/path/to/vault-watcher.sh|$HOME/.vault-watcher/vault-watcher.sh|" \
  ~/Library/LaunchAgents/com.vault-watcher.plist

# 启动服务
launchctl load ~/Library/LaunchAgents/com.vault-watcher.plist
```

### 4. 验证

```bash
# 检查是否在运行
launchctl list | grep vault-watcher

# 创建一个测试文件
echo "test" > ~/path/to/vault/Inbox/test.md

# 约 20 秒后查看日志
tail -5 ~/.vault-watcher/logs/vault-watcher.log
```

---

## 配置项

### 时间参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| fswatch latency | 8 秒 | 事件收集窗口 |
| DEBOUNCE_SEC | 10 秒 | 收到事件后等待更多事件再推送 |
| 整体延迟 | ~18 秒 | 从文件变更到收到通知的最大延迟 |

### 噪声过滤

自动忽略：

| 模式 | 原因 |
|------|------|
| `*.icloud` | iCloud 同步占位文件 |
| `*~` | 编辑器备份文件 |
| `*.tmp` | 临时文件 |
| `.DS_Store` | macOS 元数据 |
| 隐藏文件（`.*`） | 系统/应用配置 |
| `.obsidian/*` | Obsidian 内部配置 |
| 目录 | 只跟踪文件变更 |

---

## Diff 追踪原理

对于你最关心的那个高频文件（比如待办汇总），Vault Watcher 会：

1. 在首次启动时，保存一份该文件的"影子副本"
2. 每次检测到修改，用 `diff` 对比 影子副本 vs 当前文件
3. 提取新增行和删除行，去除 Markdown 格式噪声
4. 把可读的 diff 摘要放进飞书卡片
5. 更新影子副本为最新版本

这意味着你能看到**自上次推送以来**具体改了什么，而不是自上次 git commit 以来的全部变更。

---

## 管理命令

```bash
# 临时停止（下次开机还会自启）
launchctl unload ~/Library/LaunchAgents/com.vault-watcher.plist

# 重新启动
launchctl load ~/Library/LaunchAgents/com.vault-watcher.plist

# 永久移除
launchctl unload ~/Library/LaunchAgents/com.vault-watcher.plist
rm ~/Library/LaunchAgents/com.vault-watcher.plist

# 查看日志
tail -f ~/.vault-watcher/logs/vault-watcher.log

# 重建快照（如果"新增"检测不准）
rm ~/.vault-watcher/logs/vault-watcher-snapshot.txt
# 然后重启服务

# 重置 diff 基线（重新开始对比）
rm ~/.vault-watcher/logs/vault-watcher-todo-shadow.md
# 然后重启服务
```

---

## 设置飞书 Bot

1. 访问 [飞书开放平台](https://open.feishu.cn/) → 创建应用
2. 开启 **机器人** 能力
3. 添加权限：`im:message`（发送消息）
4. 获取 `App ID` 和 `App Secret`
5. 创建群聊，把机器人加进去
6. 获取群的 `chat_id`

如果用的是 Lark 国际版：
```bash
export FEISHU_API_BASE="https://open.larksuite.com"
```

---

## 局限性

- **仅支持 macOS**（依赖 `launchd` + `fswatch`）
- **Diff 仅追踪单个文件**（最关心的那个高频文件）
- **不展示文件内容变更**（除了 Diff 追踪的那个文件外）
- **bash 3.2 兼容性约束**（macOS 自带版本，不支持关联数组）

---

## License

MIT
