# Vault Watcher 管理命令

## 状态检查

```bash
# 检查服务是否运行
launchctl list | grep vault-watcher

# 查看最近日志
tail -20 ~/.vault-watcher/logs/vault-watcher.log

# 查看 stderr 日志（排错用）
tail -20 /tmp/vault-watcher-stderr.log
```

## 启停控制

```bash
# 获取 plist 路径（根据 label 查找）
LABEL="com.vault-watcher"  # 默认，自定义安装可能不同
PLIST="$HOME/Library/LaunchAgents/${LABEL}.plist"

# 停止（下次开机仍自启）
launchctl unload "$PLIST"

# 启动
launchctl load "$PLIST"

# 重启
launchctl unload "$PLIST" && launchctl load "$PLIST"
```

## 永久卸载

```bash
LABEL="com.vault-watcher"
PLIST="$HOME/Library/LaunchAgents/${LABEL}.plist"

# 1. 停止服务
launchctl unload "$PLIST" 2>/dev/null

# 2. 删除 plist
rm -f "$PLIST"

# 3. 删除安装目录（含日志和脚本）
rm -rf ~/.vault-watcher

# 4. 清理临时文件
rm -f /tmp/vault-watcher-*.txt /tmp/vault-watcher-*.pid /tmp/vault-watcher-*.log
```

## 配置修改

修改监控目录、飞书凭证等需编辑安装后的脚本：

```bash
# 编辑监控脚本（改 WATCH_DIRS / ARCHIVE_DIR / DIFF_TARGET / DEBOUNCE_SEC）
vim ~/.vault-watcher/vault-watcher.sh

# 编辑通知脚本（改飞书凭证）
vim ~/.vault-watcher/vault-notify.py

# 修改后重启生效
launchctl unload ~/Library/LaunchAgents/com.vault-watcher.plist
launchctl load ~/Library/LaunchAgents/com.vault-watcher.plist
```

也可以重新运行 install.sh 覆盖安装。

## 快照与 Diff 重置

```bash
# 重建文件快照（修复"新增"误报）
rm ~/.vault-watcher/logs/vault-watcher-snapshot.txt
# 重启服务后自动重建

# 重置 Diff 基线（重新开始 diff 对比）
rm ~/.vault-watcher/logs/vault-watcher-todo-shadow.md
# 重启服务后自动重建
```

## 常见问题

| 症状 | 排查 |
|------|------|
| 飞书没收到通知 | 检查 stderr 日志 + 凭证是否正确 + 机器人是否在群内 |
| 新文件被识别为"修改" | 快照文件已包含该路径，删除快照重建 |
| 大量 iCloud 噪声 | 检查 should_ignore 函数，确认 .icloud 过滤生效 |
| 服务不自启 | 确认 plist 在 ~/Library/LaunchAgents/ 且 `launchctl list` 有输出 |
| fswatch 找不到 | 确认 PATH 包含 /opt/homebrew/bin（Apple Silicon）或 /usr/local/bin（Intel） |
