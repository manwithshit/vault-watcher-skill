# Vault Watcher 安装引导流程

本文档定义 AI 引导用户完成 Vault Watcher 安装的交互流程。

## 前置条件检查

运行以下命令确认环境：

```bash
# 检查 macOS
uname -s  # 必须输出 Darwin

# 检查 fswatch
command -v fswatch && echo "OK" || echo "MISSING"

# 检查 Python 3
python3 --version

# 检查 Homebrew（fswatch 缺失时需要）
command -v brew && echo "OK" || echo "MISSING"
```

如果 fswatch 缺失且 brew 可用，install.sh 会自动安装。否则提示用户先安装 Homebrew。

## 配置收集清单

按顺序向用户收集以下信息。每一步都应有默认值建议。

### Step 1: Vault 路径

询问用户 Obsidian 笔记库的绝对路径。

- 自动检测：检查当前工作目录是否为 Obsidian vault（含 `.obsidian/` 目录）
- 默认值：当前工作目录（如果检测到是 vault）
- 验证：路径存在且包含 `.obsidian/` 目录

### Step 2: 监控目录

询问用户要监控哪些目录（相对于 vault 根目录）。

- 自动检测：列出 vault 根目录下的文件夹供用户选择
- 无默认值，必须由用户确认
- 格式：冒号分隔
- 建议：选择用户日常写入最频繁的目录（如 Inbox、Sources）

### Step 3: 归档目录

询问归档目录路径（用于区分"归档"和"删除"事件）。

- 无默认值，让用户指定（通常是 Archive 相关的目录）
- 验证：目录存在

### Step 4: Diff 追踪文件（可选）

询问用户是否要对某个高频文件进行 diff 追踪。

- 解释：每次该文件变更时，通知会包含具体增删了哪些行
- 典型用例：待办汇总文件
- 默认值：留空（禁用）
- 示例：用户最常修改的文件（如待办汇总）

### Step 5: 飞书凭证

需要 3 个值：

1. **App ID** (`cli_xxxxx`)
2. **App Secret**
3. **Chat ID** (`oc_xxxxx`)

提示用户：
> 需要在 [飞书开放平台](https://open.feishu.cn/app) 创建应用并开启机器人能力。
> 1. 创建应用 → 开启「机器人」能力
> 2. 权限管理 → 添加 `im:message`（发送消息）权限
> 3. 版本管理 → 创建版本 → 发布
> 4. 创建群聊 → 把机器人加入群
> 5. 获取 App ID、App Secret、Chat ID

如果用户已有其他飞书自建应用的凭证，可以直接复用。

**检测已有凭证**：检查以下位置是否已存在飞书凭证可复用：
```bash
# 检查环境变量
echo $FEISHU_APP_ID $FEISHU_APP_SECRET $FEISHU_CHAT_ID

# 检查现有 vault-watcher 部署
cat ~/.vault-watcher/vault-notify.py 2>/dev/null | grep -E "APP_ID|APP_SECRET|CHAT_ID"
```

### Step 6: API 基础 URL（可选）

- 默认值：`https://open.feishu.cn`（国内版飞书）
- 国际版 Lark：`https://open.larksuite.com`

### Step 7: 安装目录（可选）

- 默认值：`~/.vault-watcher`
- 高级用户可自定义

## 执行安装

收集完毕后，设置环境变量并运行 install.sh：

```bash
export VW_VAULT_ROOT="<collected>"
export VW_WATCH_DIRS="<collected>"
export VW_ARCHIVE_DIR="<collected>"
export VW_DIFF_TARGET="<collected>"
export VW_APP_ID="<collected>"
export VW_APP_SECRET="<collected>"
export VW_CHAT_ID="<collected>"
export VW_API_BASE="<collected>"

bash "<skill_dir>/scripts/install.sh"
```

## 安装后验证

1. 检查服务状态：`launchctl list | grep vault-watcher`
2. 创建测试文件：`echo "test" > <vault>/00_Inbox/vault-watcher-test.md`
3. 等待 ~20 秒，确认飞书群收到通知
4. 删除测试文件：`rm <vault>/00_Inbox/vault-watcher-test.md`
5. 确认收到删除通知

验证成功后告知用户安装完成。
