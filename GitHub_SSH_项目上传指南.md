# GitHub SSH 密钥及项目上传指南

Generated with the assistance of ChatGPT

## 1. 检查本地是否已有 SSH 密钥及新密钥生成

### 1.1 查看现有 SSH 密钥

**Git Bash**

```bash
ls -al ~/.ssh
```

**PowerShell**

```powershell
dir $HOME\.ssh
```

### 1.2 如果本地没有 SSH 密钥，创建新的密钥对

在 **Git Bash** 或 **PowerShell** 中运行：

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

- **确认密钥保存路径**系统会提示：

  ```
  Enter file in which to save the key (/c/Users/你的用户名/.ssh/id_ed25519):
  ```

  - **按回车**：使用默认路径 `~/.ssh/id_ed25519`。
  - **输入其他路径或文件名**：例如 `~/.ssh/id_ed25519_github`。
- **设置密码短语（passphrase）**系统会提示：

  ```
  Enter passphrase (empty for no passphrase):
  ```

  - **留空回车**：生成无短语私钥。
  - **输入短语并回车**：为私钥增加保护，然后再输入一次确认。
- **完成后输出结果**

  ```
  Your identification has been saved in /c/Users/你的用户名/.ssh/id_ed25519
  Your public key has been saved in /c/Users/你的用户名/.ssh/id_ed25519.pub
  ```

## 2. 将现有公钥添加到 GitHub

### 2.1 复制公钥到剪贴板

**Git Bash**

```bash
cat ~/.ssh/id_ed25519.pub | clip
```

或

```bash
cat ~/.ssh/id_rsa.pub | clip
```

**PowerShell**

```powershell
Get-Content $HOME\.ssh\id_ed25519.pub | clip
```

或

```powershell
Get-Content $HOME\.ssh\id_rsa.pub | clip
```

### 2.2 在 GitHub 上添加 SSH 密钥

1. 登录 GitHub，进入 **Settings** → **SSH and GPG keys**。
2. 点击 **New SSH key**，填写 **Title**（如“我的 Windows 电脑”）。
3. 在 **Key** 文本框中粘贴公钥内容。
4. 点击 **Add SSH key** 完成绑定。

### 2.3 检查本地与 GitHub 连接性

**Git Bash**

```bash
ssh -T git@github.com
```

**PowerShell**

```powershell
ssh -T git@github.com
```

- 首次连接会提示是否继续连接，输入 `yes` 确认。
- 成功时显示：
  ```
  Hi <YourUsername>! You've successfully authenticated, but GitHub does not provide shell access.
  ```

## 3. 初始化本地仓库并推送项目到 GitHub

在 **Git Bash** 或 **PowerShell** 中执行：

```bash
cd /c/Users/jidegaosuwo/你的项目路径
git init
git branch -M main
git remote add origin git@github.com:Sellifake/Hyperspectral_Image_Datasets_Collection.git
git add -A
git commit -m "Initial commit"
git push -u origin main


完成以上操作后，你的本地项目就已通过 SSH 安全地上传到 GitHub 仓库。

## 4. Clone Github现有库到本地
```bash
git clone git@github.com:Sellifake/Graduation-Design-Code.git
```
