# Git Bash 安装与使用指南

## 什么是 Git Bash
Git Bash 是 Git for Windows 自带的一个类 UNIX 终端环境，集成 Bash shell 和 MinTTY 终端模拟器。它让 Windows 用户能够使用常见的 Linux/Unix 命令（如 `ls`, `grep`, `ssh` 等）及 Git 命令。

## 安装步骤

1. **下载 Git for Windows**
   - 访问官网：https://gitforwindows.org
   - 点击「Download」按钮，获取最新的安装程序（`Git-<版本>-64-bit.exe`）。

2. **运行安装程序**
   1. 双击下载的安装包，点击 **Next**。
   2. **组件选择**：保持默认，确保勾选 “Git Bash Here” 以在资源管理器中添加右键菜单。
   3. **终端模拟器**：选择 “Use MinTTY” 以获得更佳的终端体验。
   4. **环境变量**：选 “Git from the command line and also from 3rd-party software”。
   5. 点击 **Install**，安装完成后点击 **Finish**。

3. **运行 Git Bash**
   - **开始菜单**：找到 “Git” 文件夹，点击 **Git Bash**。
   - **资源管理器右键**：在任意文件夹空白处，右键选择 **Git Bash Here**。

## 常用快捷键

### 复制与粘贴
- **Ctrl+Shift+C**：复制选中的文本  
- **Ctrl+Shift+V**：粘贴剪贴板内容  
- **鼠标右键**：复制/粘贴（选中后右键复制，右键粘贴）

### 全选文本
- **Ctrl+Shift+A** ：全选终端内容（需在 Options 中启用 Ctrl+Shift 快捷键）  
- **Alt+Space → E → S**：通过系统菜单全选

### 分屏与切换
- **Ctrl+Shift+T**：新建标签页  
- **Ctrl+Tab**：切换到下一个标签  
- **Ctrl+Shift+N**：新建一个 Git Bash 窗口

## 小提示
- 若未启用 `Ctrl+Shift` 快捷键：在 Git Bash 窗口标题栏右键 → **Options… → Keys**，勾选 “Ctrl+Shift+letter shortcuts” 后重启终端。
- 也可使用 **Shift+Insert** 粘贴、**Ctrl+Insert** 复制。

---

*文档来源于 Git for Windows 官方及实践经验整理*  
