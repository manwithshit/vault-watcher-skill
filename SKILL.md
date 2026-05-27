---
name: vault-watcher
description: >
  Obsidian 笔记库文件变更实时监控 → 飞书/Lark 群通知。基于 macOS launchd + fswatch，
  检测新增/修改/归档/删除事件并推送飞书卡片，支持对高频文件做 diff 追踪。
  当用户说"安装 vault-watcher""部署 vault watcher""帮我装文件监控""vault 变更通知"
  "文件监控推飞书""我想知道 vault 什么时候有变化"时使用。
  也适用于已安装后的管理操作："vault-watcher 状态""停掉 vault-watcher"
  "vault-watcher 日志""重启 vault-watcher""卸载 vault-watcher"
  "修改监控目录""换飞书群"。
  仅支持 macOS。
---

# Vault Watcher

Obsidian 笔记库文件变更实时监控 → 飞书/Lark 群通知。

## 工作原理

```
fswatch (8s收集) → vault-watcher.sh (10s去抖+分类+diff) → vault-notify.py (飞书卡片)
```

由 macOS `launchd` 托管：开机自启、崩溃自愈。

检测 5 类事件：➕ 新增 / ✏️ 修改 / 📦 归档 / 🗑️ 删除 / 📋 Diff 详情

## 路由决策

根据用户意图选择分支：

| 用户意图 | 动作 |
|----------|------|
| 安装 / 部署 / 首次使用 | → [安装流程](#安装流程) |
| 查状态 / 看日志 / 启停 / 卸载 | → 读取 `references/management.md` |
| 修改配置（换目录/换群/换凭证） | → [配置变更](#配置变更) |

## 安装流程

读取 `references/setup-guide.md` 获取完整引导流程，然后按步骤交互式收集配置。

### 简要步骤

1. **环境检查** — 确认 macOS、fswatch、Python 3
2. **收集配置** — 通过对话逐步收集：
   - Vault 路径（自动检测当前工作目录）
   - 监控目录列表（列出 vault 根目录供用户选择）
   - 归档目录（用户指定用于区分归档和删除事件的目录）
   - Diff 追踪文件（可选，用户指定高频修改的文件）
   - 飞书凭证（App ID / App Secret / Chat ID）
3. **执行安装** — 设置环境变量，运行 `scripts/install.sh`
4. **验证** — 创建测试文件，确认飞书收到通知

### 凭证复用检测

安装前检查是否已有飞书凭证可复用：

```bash
echo $FEISHU_APP_ID
cat ~/.vault-watcher/vault-notify.py 2>/dev/null | grep APP_ID
```

找到已有凭证时，确认用户是否复用。

## 配置变更

修改已安装的 vault-watcher 配置：

1. 确认安装目录（默认 `~/.vault-watcher`）
2. 用 StrReplace 修改对应脚本文件中的配置值
3. 重启服务：`launchctl unload <plist> && launchctl load <plist>`

或者重新运行 `scripts/install.sh` 覆盖安装（需重新收集全部配置）。

## 文件结构

安装后的文件位置：

| 文件 | 路径 |
|------|------|
| 监控脚本 | `~/.vault-watcher/vault-watcher.sh` |
| 通知脚本 | `~/.vault-watcher/vault-notify.py` |
| launchd 配置 | `~/Library/LaunchAgents/com.vault-watcher.plist` |
| 运行日志 | `~/.vault-watcher/logs/vault-watcher.log` |
| 文件快照 | `~/.vault-watcher/logs/vault-watcher-snapshot.txt` |
| Diff 影子副本 | `~/.vault-watcher/logs/vault-watcher-todo-shadow.md` |

## 依赖

- macOS（launchd + FSEvents）
- fswatch（`brew install fswatch`）
- Python 3（macOS 自带）
- 飞书/Lark 自建应用（`im:message` 权限）
