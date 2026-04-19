---
title: Claude Code 在 cmd 下 Alt+V 粘贴图片失败的排查与解决
description: 新机 Windows 11 上，cmd 启动的 Claude Code 按 Alt+V 无法粘贴 Snipaste 图像（提示 No image found in clipboard），PowerShell 下却一切正常。本文记录完整排查思路、确认的表面原因与推测的深层原因，以及最终修复方法。
#多个标签请使用英文逗号分隔或使用数组语法
tags: Claude Code, cmd, PowerShell, Snipaste, 剪贴板, 环境变量, Windows, REG_EXPAND_SZ
#多个分类请使用英文逗号分隔或使用数组语法，暂不支持多级分类
category: 工具教程
---

## 导读

从旧的 Windows 10 电脑换到新买的 Windows 11 电脑（**真的就是昨天买的**），我的 Claude Code 工作流遇到了一个诡异的问题：

- **旧电脑（Win10）**：用 cmd 打开 Claude Code，Snipaste 截图后按 `Alt+V` 粘贴图像给 Claude，一切正常；
- **新电脑（Win11）**：同样的 Snipaste，同样的 Claude Code，从 **cmd** 启动时按 `Alt+V` 会报 `No image found in clipboard`；但从 **PowerShell** 启动时又一切正常。

这种"同一台电脑、同一个程序、不同父壳结果不同"的问题比较少见。本文记录完整的排查过程、**已确认的表面原因**、**推测但未完全证实的深层原因**，以及最终的修复方法。希望给遇到类似问题的朋友一个参考。

  <img src="resource\02-A-问题现象.png" alt="问题现象" width="700" style="display:block; margin:auto;">
  <div style="text-align: center;">问题现象：cmd 下 Alt+V 提示剪贴板没有图片</div>
  <br/>

## 1. 现象描述

- 从 Snipaste 截图后触发"复制到剪贴板"（F1 框选后按回车，或双击选区）
- 切到 Claude Code（cmd 启动的那个）按 `Alt+V`
- 提示：**`No image found in clipboard. Use alt+v to paste images.`**
- 但如果用 `Ctrl+V`（本来是粘贴文本的快捷键）**反而能把图片"贴进去"**——后面会解释为什么

而同样一张图，在 PowerShell 启动的 Claude Code 里按 `Alt+V`，**瞬间就能读到**。

## 2. 排查思路

### 猜想 1：多版本 claude 共存 → 排除

新电脑上我先用 PowerShell 的 `irm` 命令安装 Claude Code 失败了一次，后来改用 `winget` 才装成功。怀疑可能残留了旧版本，PATH 里同时有两个 `claude.exe`。

验证方法直接明了——**分别在两个壳里查询 `claude` 的解析位置**：

```cmd
:: cmd 下
where claude
```

```powershell
# PowerShell 下
Get-Command claude -All
```

结果两边返回的都是**同一个路径**：

```
C:\Users\Benso\AppData\Local\Microsoft\WinGet\Packages\
  Anthropic.ClaudeCode_Microsoft.Winget.Source_8wekyb3d8bbwe\claude.exe
```

  <img src="resource\02-B-两个壳下的claude路径.png" alt="两个壳下的 claude 路径" width="700" style="display:block; margin:auto;">
  <div style="text-align: center;">两个壳下解析到的 claude 完全是同一个可执行文件</div>
  <br/>

**结论：只有一个 claude.exe**，多版本共存的假设被否掉。

另外也检查了几个常见安装位置都无残留：

- `%APPDATA%\npm\`（npm 全局）：无 claude
- `%LOCALAPPDATA%\AnthropicClaude\`（irm 安装默认路径）：不存在
- `%LOCALAPPDATA%\Programs\`：无 anthropic 相关

### 猜想 2：管理员权限差异 / STA 套间问题 → 排除

Windows 剪贴板 API 要求线程处于 **STA（单线程套间）** 模式；另外，UIPI（用户界面特权隔离）会阻止非管理员进程读取管理员进程放到剪贴板里的内容。

但两个终端都是以普通用户身份运行，Snipaste 也没有以管理员身份运行，所以这两个方向也可以排除。

### 猜想 3：Claude Code 内部机制 → 进入深挖

既然是**同一个** `claude.exe`，差别只能出在"启动它的父进程环境"上。于是想到 Claude Code 有没有留下什么本地日志或环境快照。

## 3. 关键线索：Claude Code 的 shell 快照

查看 `%USERPROFILE%\.claude\` 目录，发现几个很有意思的子目录：

| 目录 | 用途 |
|------|------|
| `shell-snapshots/` | Claude Code 运行过程中保存的 shell 环境快照 |
| `image-cache/<UUID>/` | `Alt+V` 成功粘贴的图像缓存 |
| `paste-cache/` | `Ctrl+V` 文本粘贴的内容缓存 |
| `projects/`、`sessions/` 等 | 会话状态 |

  <img src="resource\02-C-claude目录结构.png" alt="claude 目录结构" width="600" style="display:block; margin:auto;">
  <div style="text-align: center;">~/.claude 目录结构</div>
  <br/>

**最关键的发现**：`shell-snapshots/` 下的所有快照文件名都是 `snapshot-bash-xxx.sh`，内容是一段 bash 脚本。打开看，里面有这样的判断：

```bash
elif [[ "$OSTYPE" == "msys" ]] || [[ "$OSTYPE" == "cygwin" ]] || [[ "$OSTYPE" == "win32" ]]; then
  ARGV0=rg "$_cc_bin" "$@"
```

这说明 **Claude Code 在 Windows 上统一使用 Git Bash（MSYS）作为内部 shell 运行时**，不管你的父壳是 cmd 还是 PowerShell，它内部都会走 bash。

快照文件里还能看到继承过来的 `PATH`：

```bash
export PATH='/c/Users/Benso/bin:/mingw64/bin:/usr/local/bin:/usr/bin:/bin:
  ...
  %SystemRoot%/system32:%SystemRoot%:%SystemRoot%/System32/Wbem:
  %SYSTEMROOT%/System32/WindowsPowerShell/v1.0:%SYSTEMROOT%/System32/OpenSSH:
  /c/Windows/system32:...'
```

注意到关键问题了：**`%SystemRoot%` 和 `%SYSTEMROOT%` 出现在 bash 的 PATH 里，且没被展开**！

  <img src="resource\02-D-shell快照PATH问题.png" alt="shell 快照里的 PATH 问题" width="800" style="display:block; margin:auto;">
  <div style="text-align: center;">shell 快照里 PATH 包含未展开的 %SystemRoot% / %SYSTEMROOT%</div>
  <br/>

> **补充**：快照不是每次启动 Claude Code 就生成一个，而是在 Claude **首次真正需要用 Bash 工具**（比如 `ls`、`grep`、`rg`）时才生成。如果你那次 Claude 只是"粘贴图片→得到文字回复"，不会生成快照。

## 4. 表面原因（已确认）

### bash 不认识 Windows 风格的 `%VAR%`

这是一个 shell 语法常识：

| Shell | 变量展开语法 |
|-------|-------------|
| cmd.exe | `%VAR%`（不区分大小写） |
| PowerShell | `$env:VAR` |
| **bash / msys** | `$VAR` 或 `${VAR}`（**根本没有 `%VAR%` 语法**） |

所以 bash 把 `%SYSTEMROOT%/System32/WindowsPowerShell/v1.0` 当成**字面目录名**——它真的会去找一个名字叫 `%SYSTEMROOT%` 的文件夹，找不到就跳过这条 PATH 条目。

### 为什么 Windows 下正常情况 `%SYSTEMROOT%` 不会传到 bash？

因为 **Windows 在创建进程环境块时，本应把 REG_EXPAND_SZ 类型的环境变量里的 `%VAR%` 展开**。在正常系统上：

```
注册表存的字面值：%SystemRoot%\System32\WindowsPowerShell\v1.0\
           ↓（Windows 在进程创建时自动展开）
子进程拿到的 PATH：C:\Windows\System32\WindowsPowerShell\v1.0\
```

这样即使 bash 不认识 `%VAR%`，它拿到的也早就是 `C:\Windows\...` 了。

**但在我的故障状态下，这个展开没发生**——这是最诡异的地方。下一节会给出直接证据。

### 症状链条

```
[Windows 没展开 %SystemRoot%]
     ↓
[cmd 启动 claude 时，传给 claude 的 PATH 里带字面 %SYSTEMROOT%]
     ↓
[claude spawn 的 Git Bash 继承这份 PATH，把 %SYSTEMROOT% 当成字面字符串]
     ↓
[bash 沿 PATH 搜索 powershell.exe 时跳过了唯一一条指向它的条目]
     ↓
[Claude Code 读剪贴板图像需要 powershell.exe（推测 via Get-Clipboard -Format Image）]
     ↓
[调用失败，回显 No image found in clipboard]
```

### 为什么 Ctrl+V 反而能"粘贴图片"？

这里有一个有趣的副作用。当你从微信之类应用复制聊天里的图片时，剪贴板里放的**并不是原始位图数据**，而是**图片文件路径的文本表示**（CF_HDROP 格式 + CF_UNICODETEXT）。

- `Alt+V`：Claude Code 读剪贴板**图像二进制** → 没有 CF_DIB → 失败
- `Ctrl+V`：Windows Terminal 把剪贴板**文本**（也就是图片文件路径）送给 Claude Code → Claude Code 识别到这像一个本地图片路径 → 自动读盘加载 → 成功

所以 `Ctrl+V` 能"粘贴图片"并不是魔法，而是**粘贴了图片的本地路径文本**，让 Claude Code 绕过剪贴板图像 API 直接从磁盘读图。

## 5. 解决方案

从表面现象看，只要让 `echo %path%` 的输出里**不再出现字面 `%SystemRoot%`**，问题就解决了。操作非常简单：

### 打开"环境变量"对话框 → 编辑 → 保存

1. `Win + R` → 输入 `sysdm.cpl` → 回车
2. 切到 "**高级**" 选项卡 → 点 "**环境变量**"
3. 在 "**系统变量**" 里找到 `Path`，双击打开
4. 把含 `%SystemRoot%` / `%SYSTEMROOT%` 的条目改成绝对路径 `C:\Windows\...`，例如：
   - `%SYSTEMROOT%\System32\WindowsPowerShell\v1.0\` → `C:\Windows\System32\WindowsPowerShell\v1.0\`
   - `%SYSTEMROOT%\System32\OpenSSH\` → `C:\Windows\System32\OpenSSH\`
5. 点确定，**关闭所有已打开的终端**，从开始菜单重开一个新 cmd 再试

  <img src="resource\02-Final-Fix-最终调整操作.png" alt="最终调整操作" width="800" style="display:block; margin:auto;">
  <div style="text-align: center;">修改前后对比：把 %SYSTEMROOT% 改成绝对路径 C:\Windows</div>
  <br/>

### 效果立竿见影

这是修复前后在同一个 cmd 里跑 `echo %path%` 的对比——**修复前字面 `%SystemRoot%` 还在，修复后变成了展开的 `C:\Windows\...`**：

  <img src="resource\02-H-cmd下PATH变化对比.png" alt="cmd 下 PATH 变化对比" width="800" style="display:block; margin:auto;">
  <div style="text-align: center;">上半：修复前 echo %path% 仍含字面 %SystemRoot%；下半：修复后全部展开为 C:\Windows</div>
  <br/>

之后再用 cmd 启动 Claude Code，Alt+V 立刻恢复正常。

### 也可以直接用 PowerShell

如果你平时就用 Windows Terminal，把默认配置文件改成 **PowerShell**（甚至 PowerShell 7），就不会遇到这个问题。cmd 在 Windows 11 上已经是"兼容层"性质，遇到 Electron/Node 新工具时体验普遍不如 PowerShell。

## 6. 深层原因：推测与反向验证

表面的修复动作（GUI 编辑保存 PATH）**立刻生效**，这引出一个耐人寻味的问题——

> **为什么只是打开"环境变量"对话框改个内容再保存，Windows 就重新开始展开 `%VAR%` 了？**

按理说 `%SystemRoot%` 一直在注册表里，Windows 应该从一开始就会展开它。它之前为什么不展开？

### 推测：PATH 的注册表类型可能被改成了 REG_SZ

Windows 注册表里 PATH 的类型**本应是 `REG_EXPAND_SZ`**（可扩展字符串）——这个类型告诉 Windows：读这个值时请自动展开其中的 `%VAR%` 引用。

如果它被某种操作误写成了 `REG_SZ`（普通字符串），Windows 就会把 `%SystemRoot%` 当成**字面字符串**，不做展开，一路传给所有子进程。

### 反向验证（间接但相对有力）

修复后我又做了一个实验，把 PATH 改**回来**包含 `%SystemRoot%` 形式再保存：

```powershell
# 查看注册表里 Path 的字面值（不让 PowerShell 帮我展开）
(Get-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Environment').GetValue('Path','',[Microsoft.Win32.RegistryValueOptions]::DoNotExpandEnvironmentNames)
```

输出（注册表里实实在在存着字面 `%VAR%`）：
```
D:\Projects\WSN\soui-ws-2631\bin;%SystemRoot%\system32;%SystemRoot%;%SystemRoot%\System32\Wbem;%SYSTEMROOT%\System32\WindowsPowerShell\v1.0\;%SYSTEMROOT%\System32\OpenSSH\;C:\Windows\system32;...
```

而同一个 PowerShell 进程里看到的 `$env:Path`：
```
D:\Projects\WSN\soui-ws-2631\bin;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\;...
```

**注册表里是 `%SystemRoot%`，进程里已经展开成 `C:\Windows`**——说明现在 Windows 的展开机制在正常工作，bug 没复现。

同时查类型：
```powershell
(Get-Item 'HKLM:\...\Environment').GetValueKind('Path')
# 输出：ExpandString（即 REG_EXPAND_SZ）
```

这说明：
- **现在的 PATH 类型是正确的 REG_EXPAND_SZ**，展开工作正常
- 同样的内容（含 `%SystemRoot%`），现在已经不能复现 bug
- 所以故障时的类型**不太可能**也是 REG_EXPAND_SZ——只能是别的（最可能的嫌疑就是 REG_SZ）

一个关键的辅助事实：**Windows 的 GUI 环境变量对话框保存时，总是用 REG_EXPAND_SZ 类型写入**。所以你第一次点"确定"的那一刻，其实**同时做了两件事**（而你可能没意识到第二件）：

1. **可见**：改了内容
2. **不可见**：把注册表类型从可能的 REG_SZ 纠正成了 REG_EXPAND_SZ

### 但我无法完全证实 REG_SZ 就是原始原因

诚实地说，**我没法直接确认故障发生时的注册表类型到底是什么**——系统不保留历史，Windows 的 `regedit` 图形工具也**不提供"修改值类型"的功能**（只能删除再重建，操作风险大我没做）。要完全证实，只能用 PowerShell 脚本强制把 PATH 类型降级为 REG_SZ 复现故障——这需要管理员权限且有一定风险，我最后没有做这一步。

所以深层原因这里我保持一个**诚实的"强假设"态度**：

> REG_SZ 是最符合所有观察事实的最简解释，但**不是 100% 证实的结论**。也不能排除有其他机制（比如某个启动项用某种方式缓存/冻结了未展开的 env 块）。

### 新机背景可能是诱因

这台机器**是我拿到的第二天就开始装开发环境**，装过 Node.js、Git、Python、.NET SDK、Windows Kits、OpenSSH 等。

Windows 有一个著名的坑：**`setx` 命令写环境变量时，默认类型是 `REG_SZ`，而不是 PATH 本应有的 `REG_EXPAND_SZ`**。很多安装脚本和自动化工具用 `setx PATH "%PATH%;新路径"` 的方式追加 PATH，一次就能把整个 PATH 的类型降级。新机在开发环境搭建阶段是 PATH 被频繁修改的窗口期，也是这类问题的高发期——这或许就是为什么旧 Win10 上我一直没碰到，到了新机第二天就撞上了。

## 7. 总结与反思

这个问题的有意思之处在于：

1. **表层现象和根因之间隔了好几层抽象**：`Alt+V 失败` → `剪贴板 API` → `powershell.exe 不可见` → `bash PATH 里 %VAR% 字面化` → `Windows 没展开 %VAR%` → `PATH 注册表类型可能是 REG_SZ`。
2. "**多版本共存**"是一个很合理的直觉猜想，但实际并不成立。一条 `where claude` / `Get-Command claude` 就能 5 秒钟排除，这种最廉价的验证要优先做。
3. `.claude/shell-snapshots/` 这个目录里的 bash 快照，是定位问题的关键线索——**Claude Code 在 Windows 上内部统一走 Git Bash**，不看快照基本看不出来。
4. **GUI 保存"顺手"纠正了注册表类型**——这是一个典型的"看不见的副作用"，也是为什么一个简单的编辑动作能瞬间修好问题的真正原因。
5. PATH 里**尽量用绝对路径**（而不是 `%SystemRoot%`）在跨 shell、跨进程场景下更稳——即便类型正确，少一层展开依赖永远没坏处。

### 排查猜想对照表

| 猜想 | 结论 | 验证方法 |
|------|------|---------|
| 多个 claude 版本共存 | ❌ 排除 | `where claude` / `Get-Command claude` |
| 管理员权限差异 | ❌ 排除 | 查窗口标题是否有 "管理员" |
| COM 套间（STA/MTA）问题 | ❌ 排除 | `powershell -Command "Get-Clipboard -Format Image"` 能读则排除 |
| PATH 里 `%VAR%` 没被展开（表面原因） | ✅ 确认 | `echo %path%` 和 bash 快照都能看到字面 `%VAR%` |
| 注册表 PATH 类型曾是 REG_SZ（深层原因） | ⚠️ **高度推测** | 通过"改回 %VAR% 形式保存后不复现"间接佐证，但未做直接复现实验 |

### 关于"修复动作"的本质

最终我发现，GUI 对话框里的那次保存动作，看起来是"把内容改成绝对路径"，但**真正起作用的可能是"把类型纠正为 REG_EXPAND_SZ"**——这两件事恰好一起发生，很难单独验证。对读者的实用建议是：

> 遇到类似"`%SystemRoot%` 没被展开"的 Windows 环境变量怪事时，**随便用 GUI 编辑一下 PATH 再保存**（加个空格再删掉即可），很大概率就能修好——因为这个动作会隐式把类型纠正为 REG_EXPAND_SZ。

---

*本文源地址：https://www.cnblogs.com/BensonLaur/p/19891984*
