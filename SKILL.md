---
name: claude-local-cleanup
description: >
  Inventory Claude local data on the user's own Windows or Mac, list every
  login, device id, cache, chat, and program path a cleanup would delete, and
  delete nothing until the user agrees to one tier. Use when the user pastes
  this skill, asks to clean a local Claude login, or asks what a Claude
  cleanup would remove. Does not change ban outcomes and does not touch cloud chats.
---

# 清理这台电脑上的 Claude

你正在用户自己的电脑上执行。用户的要求是：先只做只读检查，把会删掉的内容列出来，同意之前不要删除。

会删掉的内容视档位而定，包括登录、设备标识、缓存、聊天记录和程序。不要只列聊天记录。

路径会随版本变。下面是要逐个探测的常见位置，不是每台电脑都有。探测为不存在就跳过，不要为了检查或删除去创建目录。

## 只读检查

先做完这一节，把清单交给用户，然后停。这一节里不要退出登录，不要结束进程，不要删除，不要修改。

1. 判断操作系统。只跑当前系统的命令。不要把另一套系统的路径拿来试。Linux 只做下面的命令行部分：凭据在 `~/.claude/.credentials.json`，没有钥匙串这一步，也没有 macOS 桌面版目录。
2. 跑「探测」里的命令。输出留给清单。不要根据输出去猜没列出的路径。
3. 对每一个存在的路径，用「每一项是什么」里的话写明：完整路径、里面是什么、删了会怎样、哪一档才会删。同一类名字可能同时出现在标准版 `Claude`、Windows 上 `Claude-Data` 的第一层子目录、以及 `Claude-3p`。各算各的。
4. `Claude-Data` 如果存在，先看它的第一层。数据通常在里面的 `claude_pzs8sxrjxfjjc` 一类子目录。对那个子目录做桌面版检查。不要把 `%LOCALAPPDATA%` 本身写进删除清单。
5. `~/.claude` 里没出现在表上的名字，只列出名字，不要打开文件。这些在只清登录和连聊天记录一起清时都留下。彻底会把整个目录删掉，所以要在彻底那一档里写明它们会一起消失。
6. `~/Claude` 和「文档\Claude」（或 `~/Documents/Claude`）只列出下一级文件夹名字，不要读文件内容。三档都不会自动删这里。
7. 某个代码仓库里的 `.claude` 不要放进删除清单，除非用户后来点名那条路径。
8. 环境变量 `CLAUDE_CONFIG_DIR` 有值时，那个目录和默认的 `~/.claude` 是两套，都要探测。如果它等于用户主目录，或者路径里没有 `claude`，只报告路径，不要把它写进可删清单。
9. 统计会话文件个数可以，例如 `projects` 下的 `jsonl` 数量。不要把文件内容、提示词或回复抄进对话。
10. 清单末尾写一句：这次没有删除任何文件。请用户用下面三句之一回复。没回复就继续停着。
    - 只清登录，聊天记录留下
    - 连聊天记录一起清
    - 彻底。我知道聊天记录、自写配置和程序都会删

公司或学校发的电脑：清单照常给，并写明请先给管理员看。用户没有说可以继续之前，不要进入删除。

如果这次对话就跑在这台电脑的 Claude 里，要写明：连聊天记录一起清，或彻底，可能会让这次对话在本机上断开。清单先交完。

### 清单格式

用这四段，空的就写「没有」。不要把不存在的路径写成将要删除。

**只清登录就会删除。** 这里没有聊天正文。分成「登录和设备标识」和「缓存和日志」。必须单独写到这些容易被当成闲文件、其实会一起没的项：`~/.claude.json`（账户信息和个人 MCP 列表）、`backups`（旧账户信息还能从这里恢复）、`configLibrary`（在桌面版里亲手写的配置，密钥是加密保存的）、`plugin-mcp-auth`、`ccd-session-secrets`、`Preferences`。没有 `.credentials.json` 不代表没登录。macOS 上凭据通常在钥匙串里。

**连聊天记录一起清才会再删。** 开头写明：这是本机聊天、输入历史、附件和 Cowork 工作文件。云端记录不动。想留聊天记录，就不能选这一档，也不能选彻底。

**只有彻底才会再删。** 开头写明：程序会卸掉，探测到的整份数据目录会删掉，聊天记录一定在里面。`skills`、`plugins`、`CLAUDE.md`、`settings.json`、`claude_desktop_config.json`，以及 `.claude` 里没列入上表的文件，都会随整个目录消失。要留的，让用户现在点名。

**查到了，但不会自动删。** `~/Claude`、文档里的 Claude、仓库里的 `.claude`。只写文件夹名字。

## 同意之后才许改动

下面任何一句都不算同意：「可以」「清理一下」「你看着办」「行」。用户必须选中一档。

- 选了只清登录：只删那一档。聊天路径一个都不许删。
- 选了连聊天记录一起清：先删只清登录那一档，再删聊天那一档。
- 选了彻底：聊天记录、自写配置和程序都在范围内。用户同时又说要留聊天记录时，拒绝彻底，停下来。
- 用户还要在这台电脑上继续用现在这个账号：不要退出登录，停下来。

顺序固定：退出登录 → 退出进程 → 再删文件。顺序反了，程序会把刚删的登录写回去。

每批删除之前，把这一批的真实路径再念一遍。只删这一档、并且探测为存在的路径。

解析成真实路径之后，如果目标是下面任何一个本身，拒绝删除：用户主目录、`AppData`、`LocalAppData`、`/Applications`、`~/Library`、`~/Library/Application Support`。也不要删除它们的上一级。

不要打印、复制或总结凭据内容。禁止 `security` 带 `-w` 或 `-g`。禁止 `dump-keychain`。禁止用 `type`、`cat`、`Get-Content` 打开 `.credentials.json`、`config.json`、`Cookies`、钥匙串密码。回执里也不要出现这些内容。`claude auth status` 只向用户报告已登录或未登录，不粘贴原始输出。

不要运行从网上下载的清理脚本，不要安装第三方清理软件。

Windows 做完后问一句：WSL 里有没有装过 Claude。有的话，在 WSL 里再做一遍命令行那一节。

## 每一项是什么

本机不包括 claude.ai 上的云端记录。那些碰不到。

### 只清登录就会删

| 名字 | 里面是什么 | 删了会怎样 |
|---|---|---|
| `.claude` 里面的 `.credentials.json` | 命令行登录凭据。macOS 通常在钥匙串里，文件写不进去时才出现。Windows 默认就是这个文件 | 这台电脑要重新登录。这里没有登录密码。只删 `.claude` 目录里的这一份 |
| 钥匙串里服务名是 `Claude Code-credentials` 的条目 | 同一份登录凭据。官方只写了「钥匙串」，这个服务名来自使用报告 | 重新登录。不要把内容复制出来。后面带短横线和一串字符的同类条目也算。不要动别的服务名 |
| Windows 凭据管理器里名字带 Claude 或 Anthropic 的条目 | 只有本机真把登录放进凭据管理器时才有。官方默认不是这里 | 删之前把目标名给用户看。不要删其他软件的凭据 |
| 主目录下的 `.claude.json` | 账户信息，还有个人 MCP 列表和界面开关 | 旧账户信息没了，这份 MCP 列表也没了 |
| `.claude/backups` | `.claude.json` 的旧副本 | 旧账户信息不能从这里恢复 |
| `policy-limits.json` 和旁边的 `policy-limits.json.stamp.json` | 这台登录的额度缓存 | 下次登录会重新取 |
| `ant-did` | 这次安装随机生成的设备标识，不是硬件序列号 | 下次启动会生成一个新的 |
| `Cookies`、`Cookies-journal`、`Network/Cookies`、`Network/Cookies-journal` | 桌面版网站登录态。有的在数据目录根，有的在 `Network` 里 | 桌面版要重新登录。只删存在的那一处，不要为了找它去删整个 `Network` |
| `Local Storage`、`Session Storage`、`IndexedDB`、`Local State`、`blob_storage`、`Partitions`、`Service Worker`、`WebStorage`、`DIPS`、`Shared Dictionary`、`TransportSecurity`、`Network Persistent State`、`Trust Tokens`、`Trust Tokens-journal` | 桌面版记住的登录和站点状态 | 登录态和一部分界面状态没了 |
| `config.json`、`config.json.journal` | 桌面版保存的登录和设置 | 桌面版要重新登录 |
| `configLibrary` | 在桌面版里亲手写的配置，密钥是加密保存的 | 这些配置没了 |
| `plugin-mcp-auth`、`ccd-session-secrets` | 插件登录，以及一份会话密钥 | 插件要重新授权 |
| `Preferences` | 桌面版自己的偏好 | 窗口大小、缩放会重置 |
| `Extension State` | 扩展的本地状态 | 扩展要重新初始化 |
| `.claude` 里的 `cache`、`debug`、`logs`、`image-cache`、`stats-cache.json`、`usage-data`、`statsig` | 命令行缓存、调试日志、用量统计 | 统计和日志没了。聊天正文不在这里 |
| 桌面版数据目录里的 `Cache`、`Code Cache`、`GPUCache`、`GPUPersistentCache`、`DawnGraphiteCache`、`DawnWebGPUCache`、`GraphiteDawnCache`、`GrShaderCache`、`sentry`、`Crashpad`、`logs` | 加速缓存、崩溃报告、运行日志 | 下次打开会慢一会儿，然后自己重建。不是聊天正文 |
| `~/Library/Logs/Claude`、`~/Library/Logs/Claude-3p` | macOS 桌面版日志。不是每台机器都有 | 日志没了 |
| `%TEMP%\claude` | 仅 Windows。命令行临时草稿，可能有读过的文件内容 | 草稿没了。不要删整个 `%TEMP%` |

### 连聊天记录一起清才会再删

| 名字 | 里面是什么 | 删了会怎样 |
|---|---|---|
| `.claude/projects` | 命令行的完整会话：提示、回复、工具读过和写过的内容 | 这台电脑不能再恢复这些会话。云端还在 |
| `.claude/history.jsonl` | 命令行里输入过的提示词，没有完整回复 | 向上翻历史、搜索旧提示会空 |
| `.claude/file-history` | 会话改文件时留下的检查点 | 不能按检查点把旧文件恢复回来 |
| `.claude/paste-cache`、`uploads` | 贴进会话的大段文字、附过的文件 | 这些附件会丢 |
| `.claude/plans` | 会话里写过的计划 | 这些计划文本没了 |
| `.claude/session-env`、`shell-snapshots` | 某次会话当时的环境和命令现场 | 不含完整聊天，但可能夹着命令输出 |
| 桌面版的 `local-agent-mode-sessions` | 桌面版聊天和 Cowork 任务：提示、回复、附件副本、产出 | 这台电脑上的这些对话没了。云端还在 |
| `claude-code-sessions` | 桌面版代码会话的索引和摘要。正文在 `projects` | 桌面版会话列表会空。只删这里、留下 `projects`，正文还在 |
| `vm_bundles` | Cowork 的虚拟机磁盘，里面有会话的工作目录 | 那些工作文件没了。下次使用会重建空磁盘，体积常常很大 |

### 默认留下

| 名字 | 里面是什么 |
|---|---|
| `.claude/skills`、`plugins` | 用户安装或写的技能和插件 |
| `CLAUDE.md` | 写给 Claude 的长期说明 |
| `.claude/settings.json` | 偏好、权限规则、模型选择 |
| `claude_desktop_config.json` | 桌面版的 MCP 服务器配置 |
| 代码仓库里的 `.claude` | 这个项目共享的说明和设置，不是登录 |
| `.claude` 里没出现在上面两张表的文件 | 不在删除名单里。只清登录和连聊天记录一起清都不要动 |

彻底如果删掉整个目录，上表这些会一起没。用户点名要留的，先拷到一个新文件夹，再删目录。不要把 `.credentials.json` 拷进去。

### 只有彻底才会删

| 名字 | 里面是什么 | 删了会怎样 |
|---|---|---|
| `~/.local/share/claude`，Windows 上是 `%USERPROFILE%\.local\share\claude` | 命令行程序和下载过的旧版本 | 程序没了。不是聊天记录 |
| `~/.local/bin/claude` 或 `claude.exe` | 命令行的启动文件 | 命令会失效 |
| `/Applications/Claude.app` 或用户主目录里的同名 app；Windows 的 Claude 安装包 | 桌面版程序 | 程序没了 |
| 探测到的整份数据目录，末级名字只能是 `Claude`、`Claude-3p` 或 `Claude-Data` | 这一安装的全部桌面数据 | 聊天、登录、自写的桌面配置一起没 |
| `~/Library/Caches/com.anthropic.claudefordesktop`、同名的 `ShipIt`、`claude-cli-nodejs`、对应的 `HTTPStorages` 和那一个 `plist` | 桌面版缓存、待安装的更新包、命令行缓存、系统偏好 | 更新包和缓存没了。不是聊天记录 |
| 探测为存在的 `~/.claude-code`、`%LOCALAPPDATA%\Programs\claude-code`、`%LOCALAPPDATA%\Microsoft\WindowsApps\claude.exe` | 旧安装位置。当前官方卸载页没写这些 | 只删这一条，不删上级目录 |
| `%LOCALAPPDATA%\AnthropicClaude` | 多半是旧安装程序本身 | 只在彻底时卸。不要在只清登录时把它当数据目录删掉 |

`~/Claude` 和文档里的 Claude 不在这张表里。列出子文件夹名字，另外问。

## 探测

把 YES / NO 留给清单。不要打开文件。

### Windows（PowerShell）

```powershell
$ErrorActionPreference = 'Continue'
Write-Output "OS=$([Environment]::OSVersion.VersionString)"
Write-Output "CLAUDE_CONFIG_DIR=$env:CLAUDE_CONFIG_DIR"
Get-Command claude -ErrorAction SilentlyContinue | Select-Object Source
try { claude --version } catch { Write-Output "claude --version 失败" }
Get-AppxPackage -Name Claude -ErrorAction SilentlyContinue |
  Select-Object Name, Version, PackageFullName, InstallLocation | Format-List
Get-Process -ErrorAction SilentlyContinue |
  Where-Object { $_.ProcessName -match '^(?i)claude$' } |
  Select-Object Id, ProcessName, Path | Format-Table -AutoSize
cmdkey /list | Select-String -Pattern 'claude|anthropic'

$candidates = @(
  "$env:USERPROFILE\.claude",
  "$env:USERPROFILE\.claude.json",
  "$env:LOCALAPPDATA\Claude-Data",
  "$env:LOCALAPPDATA\Claude-3p",
  "$env:LOCALAPPDATA\AnthropicClaude",
  "$env:APPDATA\Claude",
  "$env:APPDATA\Claude-3p",
  "$env:LOCALAPPDATA\Packages\Claude_pzs8sxrjxfjjc\LocalCache\Roaming\Claude",
  "$env:LOCALAPPDATA\Packages\Claude_pzs8sxrjxfjjc\LocalCache\Roaming\Claude-3p",
  "$env:LOCALAPPDATA\Packages\Claude_pzs8sxrjxfjjc\LocalCache\Local\Claude-3p",
  "$env:USERPROFILE\Claude",
  "$env:USERPROFILE\Documents\Claude",
  "$env:TEMP\claude",
  "$env:LOCALAPPDATA\Programs\claude-code",
  "$env:LOCALAPPDATA\Microsoft\WindowsApps\claude.exe",
  "$env:USERPROFILE\.local\bin\claude.exe",
  "$env:USERPROFILE\.local\share\claude",
  "$env:APPDATA\Code\User\globalStorage\anthropic.claude-code"
)
if ($env:CLAUDE_CONFIG_DIR) { $candidates += $env:CLAUDE_CONFIG_DIR }
$candidates | ForEach-Object { "{0}  {1}" -f (Test-Path -LiteralPath $_), $_ }

$claudeData = "$env:LOCALAPPDATA\Claude-Data"
if (Test-Path -LiteralPath $claudeData) {
  Write-Output "---- Claude-Data 的第一层 ----"
  Get-ChildItem -LiteralPath $claudeData -Force | Select-Object Name, Mode
}
```

### macOS

```bash
echo "OS=$(sw_vers -productVersion) $(uname -m)"
echo "CLAUDE_CONFIG_DIR=${CLAUDE_CONFIG_DIR:-}"
command -v claude || echo "NO   claude-on-PATH"
claude --version || true
pgrep -l -i claude || echo "NO   claude-process"
for app in "/Applications/Claude.app" "$HOME/Applications/Claude.app"; do
  if [ -d "$app" ]; then
    echo "YES  $app"
    defaults read "$app/Contents/Info" CFBundleShortVersionString 2>/dev/null || true
  else
    echo "NO   $app"
  fi
done
echo "KEYCHAIN  不要运行 security find。请用户打开钥匙串访问，搜索 Claude Code-credentials，只回答有或没有、有几条。"

paths=(
  "$HOME/.claude"
  "$HOME/.claude.json"
  "$HOME/.local/bin/claude"
  "$HOME/.local/share/claude"
  "$HOME/.claude-code"
  "$HOME/Library/Caches/claude-cli-nodejs"
  "$HOME/Library/Application Support/Claude"
  "$HOME/Library/Application Support/Claude-3p"
  "$HOME/Library/Logs/Claude"
  "$HOME/Library/Logs/Claude-3p"
  "$HOME/Library/Caches/com.anthropic.claudefordesktop"
  "$HOME/Library/Caches/com.anthropic.claudefordesktop.ShipIt"
  "$HOME/Library/Preferences/com.anthropic.claudefordesktop.plist"
  "$HOME/Library/HTTPStorages/com.anthropic.claudefordesktop"
  "$HOME/Claude"
  "$HOME/Documents/Claude"
  "$HOME/Library/Application Support/Code/User/globalStorage/anthropic.claude-code"
)
if [ -n "${CLAUDE_CONFIG_DIR:-}" ]; then paths+=("$CLAUDE_CONFIG_DIR"); fi
for p in "${paths[@]}"; do
  if [ -e "$p" ]; then echo "YES  $p"; else echo "NO   $p"; fi
done
```

钥匙串只听用户口头说有或没有。不要运行 `security find`，不要改用 `-g`、`-w` 或 `dump-keychain`。用户没看钥匙串就写「钥匙串未能检查」。

桌面版数据目录（每一个探测为 YES 的 `Claude`、`Claude-3p`，以及 `Claude-Data` 的第一层子目录）里，再对「每一项是什么」的名字做存在检查。只把存在的写进对应档位。

`~/.claude` 如果存在，列出第一层名字。不要读文件内容。`projects` 如果存在，可以数 `jsonl` 的个数。

`~/Claude` 和文档里的 Claude 如果存在，只列出下一级名字。

彻底那一档还要只读看安装方式，有才写进清单：`brew list --cask` 里的 `claude-code` 或 `claude-code@latest`；`npm ls -g --depth=0` 里的 `@anthropic-ai/claude-code`；`winget list --id Anthropic.ClaudeCode`。没有这些命令就跳过。

## 退出登录

只在用户已经选了一档之后做。`claude` 不存在就跳过。

```bash
claude auth logout
claude auth status
```

Windows 用同一条命令。只报告已登录或未登录。

桌面版要用户自己点：打开 Claude → 左下角头像 → 退出登录 → 托盘图标右键 Quit。只关窗口不算退出。应用已经打不开时，用户说一声，记下「桌面版没能退出登录」，继续退出进程。

## 退出进程

### Windows

```powershell
Get-Process -ErrorAction SilentlyContinue |
  Where-Object { $_.ProcessName -match '^(?i)claude$' } |
  Select-Object Id, ProcessName, Path
Stop-Process -Name claude, Claude -Force -ErrorAction SilentlyContinue
```

再查一次。名字正好是 claude，或者路径在 `Claude.app`、`claude.exe`、`.local/share/claude` 下面的，如果还在，停下来给用户看。不要结束路径对不上的进程。

### macOS

```bash
osascript -e 'quit app "Claude"' 2>/dev/null || true
killall Claude 2>/dev/null || true
killall claude 2>/dev/null || true
pgrep -l -i claude || echo "NO   claude-process"
```

还有进程时，把名字和 PID 给用户看。用户同意后再结束那个 PID。不要对整个系统用 `killall -9`。

## 按档删除

先确认真实路径不是「同意之后才许改动」里拒绝的那些根目录。每一条都用完整路径。不存在就跳过。

`CLAUDE_CONFIG_DIR` 通过了前面的检查时，对那个目录再做一遍命令行删除，把根换成那个目录。主目录下的 `.claude.json` 只在默认位置删一次。

### 只清登录：命令行

下面两组只含登录档的名字。不要把 `projects` 或其他聊天名字加进这两组。

```powershell
if (-not $env:USERPROFILE) { throw "USERPROFILE empty" }
$root = Join-Path $env:USERPROFILE ".claude"
if ($root -eq $env:USERPROFILE -or $root -eq $env:LOCALAPPDATA -or $root -eq $env:APPDATA) { throw "refuse $root" }
$names = @(
  ".credentials.json","stats-cache.json","usage-data","debug","statsig","logs",
  "image-cache","cache","backups","policy-limits.json","policy-limits.json.stamp.json"
)
$names | ForEach-Object {
  $p = Join-Path $root $_
  if (Test-Path -LiteralPath $p) { Remove-Item -LiteralPath $p -Recurse -Force }
}
Remove-Item -LiteralPath (Join-Path $env:USERPROFILE ".claude.json") -Force -ErrorAction SilentlyContinue
$tempClaude = Join-Path $env:TEMP "claude"
if ($tempClaude -and $tempClaude -ne $env:TEMP -and (Test-Path -LiteralPath $tempClaude)) {
  Remove-Item -LiteralPath $tempClaude -Recurse -Force
}
```

```bash
if [ -z "$HOME" ]; then echo "REFUSE empty HOME"; exit 1; fi
root="$HOME/.claude"
case "$root" in
  "$HOME"|"$HOME/"|/|/Applications|"$HOME/Library"|"$HOME/Library/Application Support")
    echo "REFUSE $root"; exit 1 ;;
esac
for name in .credentials.json stats-cache.json usage-data debug statsig logs \
  image-cache cache backups policy-limits.json policy-limits.json.stamp.json
do
  if [ -n "$name" ] && [ -e "$root/$name" ]; then rm -rf "$root/$name"; fi
done
if [ -e "$HOME/.claude.json" ]; then rm -f "$HOME/.claude.json"; fi
```

### 只清登录：每一个桌面版数据目录

只删除目录里面的这些名字。不要删除数据目录本身。

`ant-did`、`Cookies`、`Cookies-journal`、`Network/Cookies`、`Network/Cookies-journal`、`Local Storage`、`Session Storage`、`IndexedDB`、`Local State`、`blob_storage`、`config.json`、`config.json.journal`、`Partitions`、`Service Worker`、`Cache`、`Code Cache`、`GPUCache`、`GPUPersistentCache`、`GrShaderCache`、`GraphiteDawnCache`、`DawnGraphiteCache`、`DawnWebGPUCache`、`DIPS`、`Shared Dictionary`、`TransportSecurity`、`Network Persistent State`、`WebStorage`、`Trust Tokens`、`Trust Tokens-journal`、`Preferences`、`sentry`、`Crashpad`、`Extension State`、`plugin-mcp-auth`、`ccd-session-secrets`、`configLibrary`、`logs`。

macOS 上如果存在，也删 `~/Library/Logs/Claude` 和 `~/Library/Logs/Claude-3p`。

留下 `claude_desktop_config.json`。

### 连聊天记录一起清才追加

只清登录不许跑这一组。这一组里不要加 `skills`、`plugins`、`settings.json`、`CLAUDE.md`。

命令行只追加这些名字：`projects`、`history.jsonl`、`paste-cache`、`uploads`、`file-history`、`session-env`、`shell-snapshots`、`plans`。

桌面版数据目录只追加：`vm_bundles`、`local-agent-mode-sessions`、`claude-code-sessions`。

用和上一节相同的写法，把 `$names` 或 `for name in` 换成这一组。根目录检查也照做。不要改成删除整个 `.claude`。

仍然留下 `skills`、`plugins`、`CLAUDE.md`、`settings.json`，以及没在表上的文件。

### 彻底

先卸桌面版、VS Code 扩展和 JetBrains 插件，再删数据目录。这三处只要还装着，下次打开就会把 `~/.claude` 再建出来。JetBrains 不要去猜目录，让用户在 IDE 里卸插件。

用户点名要留的 `skills`、`plugins`、`CLAUDE.md`、`settings.json`、`claude_desktop_config.json`，先拷到一个新文件夹。不要拷 `.credentials.json`。

然后只删除末级名字正好是 `.claude`、`Claude`、`Claude-3p` 或 `Claude-Data` 的那个目录，以及主目录下的 `.claude.json`。末级名字对不上就停。`Claude-Data` 只删这个目录本身，不删 `%LOCALAPPDATA%`。不要删除用户主目录。

```bash
if [ -z "$HOME" ]; then echo "REFUSE empty HOME"; exit 1; fi
target="$HOME/.claude"
if [ "$(basename "$target")" != ".claude" ]; then echo "REFUSE $target"; exit 1; fi
rm -rf "$target"
rm -f "$HOME/.claude.json"
```

Windows 用 `Remove-Item -LiteralPath`，删之前确认末级名字。数据目录同样先看末级名字，再删那一个目录。

VS Code 的 `anthropic.claude-code` globalStorage 如果存在，问过用户再删那一个目录。不要删整个 `Code\User`。

macOS 缓存只删名字完全对得上的：`~/Library/Caches/com.anthropic.claudefordesktop`、`~/Library/Caches/com.anthropic.claudefordesktop.ShipIt`、`~/Library/Caches/claude-cli-nodejs`、`~/Library/HTTPStorages/com.anthropic.claudefordesktop`、`~/Library/Preferences/com.anthropic.claudefordesktop.plist`。

## 钥匙串和凭据管理器

官方默认：macOS 是登录钥匙串，写不进去时才用 `~/.claude/.credentials.json`。Windows 默认就是那个文件。退出登录之后仍然做这一节，因为官方没有写退出登录会删掉钥匙串条目。

### macOS

```bash
security delete-generic-password -s "Claude Code-credentials"
```

再跑一次。提示找不到该项就停。不要加 `-w`，不要加 `-g`。

带短横线和一串字符的同类条目，让用户在「钥匙串访问」里搜索 `Claude Code-credentials` 后自行删除。不要用 `dump-keychain` 去找。不要删除其他服务名。

### Windows

探测时如果已经列出带 Claude 或 Anthropic 的目标，只对用户确认过的那一行执行 `cmdkey /delete:`，目标字符串原样复制。没有这类条目就跳过。

## 彻底才卸载程序

只清登录和连聊天记录一起清不做这一节。哪个安装方式探测到了，才卸哪一种。

### 桌面版

- macOS：确认路径是 `/Applications/Claude.app` 或 `~/Applications/Claude.app` 之后，进程已经退出，再删这个 `.app`。不要删 `/Applications` 里其他应用。
- Windows：让用户到「设置 → 应用」里卸载 Claude。包名是 `Claude`、包族是 `Claude_pzs8sxrjxfjjc` 时，用户确认过 `PackageFullName` 之后，可以对这一个包执行 `Remove-AppxPackage`。不要对整个 `Packages` 目录做删除。如果命令报 `0x80073D28`，这是没有管理员权限，不要反复重试，改走「设置 → 应用」。

卸掉程序之后，数据目录仍可能还在。回到上一节，把末级名字符合的数据目录整份删掉。

### 命令行

当前官方卸载页只写这两个位置。macOS、Linux、WSL：

```bash
rm -f "$HOME/.local/bin/claude"
rm -rf "$HOME/.local/share/claude"
```

Windows：

```powershell
Remove-Item -LiteralPath "$env:USERPROFILE\.local\bin\claude.exe" -Force -ErrorAction SilentlyContinue
Remove-Item -LiteralPath "$env:USERPROFILE\.local\share\claude" -Recurse -Force -ErrorAction SilentlyContinue
```

`~/.claude-code`、`%LOCALAPPDATA%\Programs\claude-code`、`%LOCALAPPDATA%\Microsoft\WindowsApps\claude.exe` 只有探测为 YES 才单独删那一条。

`brew list --cask` 里有 `claude-code` 或 `claude-code@latest` 时，才运行对应的 `brew uninstall --cask`。`npm ls -g --depth=0` 里有 `@anthropic-ai/claude-code` 时，才 `npm uninstall -g @anthropic-ai/claude-code`。`winget list --id Anthropic.ClaudeCode` 有结果时，才 `winget uninstall --id Anthropic.ClaudeCode`。

## 验证

再跑一遍探测。按用户选的那一档看，不要用别的档的标准。

只清登录做完：没有名为 claude 的进程；`.credentials.json`、`.claude.json`、`ant-did`、Cookies 为 NO。`projects` 和桌面版会话目录如果原来就有，应该还在。macOS 上 `security delete-generic-password -s "Claude Code-credentials"` 提示找不到。`claude` 如果还在，登录状态是未登录。

连聊天记录一起清做完：在上一档之外，`projects`、`history.jsonl`、`local-agent-mode-sessions`、`vm_bundles` 为 NO。`skills` 或 `plugins` 原来有的，应该还在。

彻底做完：再加上程序路径为 NO，数据目录为 NO。

第一次重新打开时，程序会自己再建目录，也会生成新的 `ant-did`。判断标准是里面没有旧账号的会话和凭据，不是文件夹必须永远不存在。

然后让用户看三处是否一致：登录页邮箱、桌面版头像里的账号、命令行的登录状态。不要替用户登录。

## 回执

删除做完后交给用户。只读检查那一步不要填这张。用户如果愿意把效果反馈回来，就发这张，不要附带任何文件。

```
系统：
档位：只清登录 / 连聊天记录一起清 / 彻底
命令行版本（claude --version，没有就写没有）：
桌面版版本（没有就写没有）：
CLAUDE_CONFIG_DIR：空 / 有（只写是否另有一套，不写里面的内容）
探测为 YES 的路径：
退出登录：命令行成功 / 桌面版用户已退出 / 应用打不开所以跳过
实际删除的路径：
保留的路径：
跳过的路径和原因：
报错原文：
验证再跑一遍之后仍为 YES 的路径：
```

这张回执只用来对照真实机器上的路径。它不是封号相关的证明。

## 做不到的

- 不能改变封号结果，也不能把账号洗白。封不封由服务端看：账号本身、支付方式、IP、用量、内容。这些不在这台电脑的文件夹里。本机清理只解决串号和隐私。
- 不能删除 claude.ai 上的云端记录。
- 不能退出其他电脑和手机上的登录。

## 位置从哪来

删之前以探测为准。官方页面：

- 凭据位置：https://code.claude.com/docs/en/authentication
- 卸载位置：https://code.claude.com/docs/en/setup
- `.claude` 里有什么：https://code.claude.com/docs/en/claude-directory
- 桌面版 3P 的数据目录：https://claude.com/docs/third-party/claude-desktop/data-storage
- 桌面版 `claude_desktop_config.json`：macOS 在 `~/Library/Application Support/Claude/`，Windows 在 `%APPDATA%\Claude\`。MSIX 常把 Roaming 改写到 `Packages\Claude_pzs8sxrjxfjjc\LocalCache\Roaming\Claude`，所以两处都探测。

`Claude-Data` 和钥匙串服务名不在上述官方页上。只有探测为 YES 才处理。
