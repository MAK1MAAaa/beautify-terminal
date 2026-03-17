# **beautify-terminal**

本指南将带你从零配置一个兼具超高颜值与极致效率的 Windows 终端环境。核心工具链：**Windows Terminal \+ Oh My Posh \+ PSReadLine \+ Zoxide \+ eza \+ gsudo**。

## **零、 前置准备：安装 Nerd Font 字体**

为了让终端正确显示各类开发图标（如 Git 状态、Python Logo、文件夹图标等），必须安装包含这些图标的补丁字体。

1. 访问 [Nerd Fonts 官网](https://www.nerdfonts.com/font-downloads)。
2. 推荐下载 **Meslo Nerd Font** 或 **CaskaydiaCove Nerd Font**。
3. 解压下载的压缩包，选中里面所有的 .ttf 字体文件，右键选择“安装”。
4. 打开 Windows Terminal 设置界面 \-\> 左侧选择你的配置文件（或“默认值”） \-\> 外观 \-\> 将“字体”修改为你刚刚安装的 Nerd Font 并保存。

## **一、 Oh My Posh：终端颜值与状态提示**

提供精美的命令提示符，能够直观展示当前路径、代码执行时间、Git 分支状态等信息。

### **1\. 安装**

在终端中运行以下命令（使用 Windows 自带包管理器）：

```shell
winget install JanDeDobbeleer.OhMyPosh \-s winget
```

*(安装后建议彻底关闭并重新打开终端)*

### **2\. 下载并存放主题**

1. 前往 [Oh My Posh 官方主题画廊](https://ohmyposh.dev/docs/themes) 挑选喜欢的主题。
2. 将主题对应的 JSON 文件（例如 `1\_shell.omp.json`）下载或保存到你的个人目录中，推荐存放在：`\~\\.poshthemes\\1\_shell.omp.json`。

## **二、 PSReadLine：历史命令智能补全**

提供类似 Fish Shell 的基于历史记录的灰色预测补全，大幅提升敲击长命令的效率。

### **1\. 安装/更新**

```shell
Install-Module \-Name PSReadLine \-AllowClobber \-Force \-Scope CurrentUser
```

*(如果提示需要安装 NuGet 或不受信任的存储库，输入 Y 并回车)*

## **三、 Zoxide：智能目录跳转神器**

记住你的目录访问习惯，实现跨层级瞬间跳转，告别繁琐的 cd 和 Tab 补全。

### **1\. 安装**

```shell
winget install zoxide
```

*(注意：安装后必须彻底重启终端，让系统加载新的环境变量)*

## **四、 eza：现代版的 ls 命令（炫酷文件列表）**

替代枯燥的传统 ls，提供带图标、色彩高亮以及 Git 状态集成的文件列表展示。

### **1\. 安装**

```shell
winget install eza
```

*(配置代码已集成在下方的最终 $PROFILE 模板中，我们会用它替换掉 PowerShell 原本难看的 ls)*

## **五、 gsudo：Windows 提权神器**

无需重新打开管理员窗口，在当前普通权限的终端内无缝执行高权限命令。

### **1\. 安装**

```shell
winget install gerardog.gsudo
```

## **六、 🎯 终极 $PROFILE 配置文件**

这是串联以上所有工具的核心步骤。

1. 在终端中输入以下命令打开配置文件：  

```shell
notepad $PROFILE
```

   *(如果提示找不到文件，先运行 `New-Item \-Type File \-Path $PROFILE \-Force` 创建它)*
2. 将以下完整代码粘贴进去（**请确保 Oh My Posh 的 JSON 路径与你实际存放的路径一致**）：  

```text
# ==========================================
# 1. Oh My Posh (主题与提示符)
# ==========================================
oh-my-posh init pwsh --config "~\.poshthemes\illusi0n.omp.json" | Invoke-Expression

# ==========================================
# 2. PSReadLine (智能补全与快捷键)
# ==========================================
# 开启基于历史记录的预测性补全
Set-PSReadLineOption -PredictionSource History
# 设置预测文本为灰色内联显示
Set-PSReadLineOption -PredictionViewStyle InlineView
# 增强上下方向键：根据当前输入过滤历史记录
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward

# ==========================================
# 3. Zoxide (目录跳转)
# ==========================================
Invoke-Expression (& { (zoxide init powershell | Out-String) })

# ==========================================
# 4. eza (现代版 ls，带图标和颜色)
# ==========================================
# 移除 PowerShell 自带的 ls 别名
Remove-Item Alias:ls -ErrorAction Ignore
# 设置 eza 函数替代原本的 ls
function ls { eza --icons=always --color=always $args }
# 常用缩写：ll (显示详细信息), la (显示隐藏文件), lt (显示树状目录)
function ll { eza -l --icons=always --color=always $args }
function la { eza -la --icons=always --color=always $args }
function lt { eza --tree --level=2 --icons=always --color=always $args }

# ==========================================
# 5. gsudo (提权别名)
# ==========================================
# 将 sudo 映射为 gsudo，贴合 Linux 使用习惯
Set-Alias sudo gsudo
```

3. 保存并关闭记事本。然后在终端运行 . $PROFILE，或者直接重启终端即可生效！

## **七、 🚨 常见问题排查 (Troubleshooting)**

### **问题：运行 zoxide 或其他命令时提示“无法识别为 cmdlet...”**

**原因**：这通常是因为通过 winget 安装新软件后，当前运行的终端进程或 IDE 尚未获取最新的系统环境变量 (PATH)。

**解决方案**：

1. **Windows Terminal**：彻底关闭所有 Terminal 窗口，重新打开即可。
2. **JetBrains IDE (如 CLion, PyCharm)**：IDE 的内嵌终端继承的是 IDE 启动时的环境变量。
    * **必须彻底退出 IDE**。
    * **注意 Toolbox**：如果你通过 JetBrains Toolbox 管理软件，请在系统右下角托盘找到 Toolbox 图标，右键选择 **“退出 (Quit)”**。
    * 重新打开 Toolbox 和 IDE，内嵌终端即可正常识别新命令。
3. **终极方案**：重启电脑，强制刷新全局环境变量。