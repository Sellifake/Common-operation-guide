# 在两台 Windows 11 机器之间通过 SSH 互联的完整流程

下面汇总了在两台 Windows 11 服务器（以下简称 “服务器” 与 “笔记本”）之间，通过原生或手动安装的 OpenSSH 完成互联的全过程。本文并未涉及 VS Code 远程开发，仅针对纯 SSH 互联做详细说明。

---

## 一、前提与总体思路

- **两台机器都运行 Windows 11**（且系统已打上近期更新补丁；本文示例系统版本号约为 10.0.26100.xxxx）。  
- 机器 A：称为“服务器”（Server），需作为 SSH Server 供远程登录。  
- 机器 B：称为“笔记本”（Client），用来 SSH 登录服务器并执行命令。  
- 两台机器都已安装并登录同一个 Tailscale 帐号，Tailscale IP 分别假定为：  
  - 服务器：`<SERVER_IP>`  
  - 笔记本：`<CLIENT_IP>`  
- 目标：在服务器上安装、启动、配置 OpenSSH Server → 在服务器本机验证 SSH 服务可用 → 再在笔记本上测试 SSH 端口可达并登录。

---

## 二、服务器端（目标机）准备

### 1. 检查系统内置 OpenSSH 功能状态

在服务器本机以管理员身份打开 PowerShell，执行：
```powershell
Get-WindowsCapability -Online | Where-Object Name -Like 'OpenSSH.Server*'
```
- 如果输出显示 `State : Installed`，说明系统自带 OpenSSH Server 功能已经存在，可跳到第 3 节“启动并配置 sshd 服务”。  
- 如果输出显示 `State : NotPresent`，则需要手动安装 OpenSSH Server（可二选一）：

> 方法 1：使用 DISM/PowerShell 安装（依赖 Windows Update）  
> 方法 2：手动从 GitHub 下载并安装 Win32-OpenSSH

---

### 2. 手动安装 Win32-OpenSSH （针对 Rare Case：内置功能不可用）

以下步骤摘自<https://www.cnblogs.com/lqqgis/p/17365809.html>，适用于“Get-WindowsCapability”中显示 `NotPresent` 或因策略限制无法通过 Add-WindowsCapability 安装的情况。

1. **下载最新 Win32-OpenSSH 发布版（Win64 版本）**  
   - 打开浏览器访问：  
     ```
     https://github.com/PowerShell/Win32-OpenSSH/releases
     ```  
   - 在页面中找到最新的 `OpenSSH-Win64.zip`（或 `OpenSSH-Win32.zip`，视系统位数而定）并点击下载。

2. **解压并复制到系统目录**  
   - 将下载得到的 `OpenSSH-Win64.zip` 解压到本地临时目录（例如 `D:\\temp\\OpenSSH-Win64`）。  
   - 复制整个解压文件夹下的文件到一个目标目录，比如：  
     ```
     C:\\Program Files\\OpenSSH
     ```  
   - 确保该目录下可以看到 `sshd.exe`、`ssh.exe`、`ssh-keygen.exe`、`scp.exe` 等可执行文件。

3. **以管理员身份打开 PowerShell，切换到 OpenSSH 目录**  
   ```powershell
   cd 'C:\\Program Files\\OpenSSH'
   ```

4. **创建并配置服务：将 sshd 与 ssh-agent 安装为 Windows 服务**  
   ```powershell
   # 设置执行策略以允许运行安装脚本
   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope LocalMachine

   # 安装 sshd 服务
   .\\install-sshd.ps1

   # （可选）安装 ssh-agent 服务，以支持基于 ssh-agent 的密钥登录
   .\\install-ssh-agent.ps1
   ```

5. **生成或配置 host 密钥**  
   OpenSSH 需要 host key 才能安全通信,如果在客户端已经生成过密钥，请跳转到`五、注意事项与常见问题`中的`补充操作`里查看。如果目录下还没有 key，需要执行：
   ```powershell
   cd 'C:\\Program Files\\OpenSSH'
   # 以下命令会在该目录下生成 ed25519 和 rsa host key
   .\\ssh-keygen.exe -A
   ```
   这会在 `C:\\ProgramData\\ssh\\` 下生成 `ssh_host_ed25519_key`、`ssh_host_rsa_key` 等文件。

6. **调整权限与配置**  
   - 确保 `C:\\ProgramData\\ssh\\` 及其子文件（host key、`sshd_config` 等）只有 `Administrators` 或 `SYSTEM` 用户拥有写权限。  
   - 如果需要修改配置，比如允许密码登录、修改端口等，可编辑：  
     ```
     C:\\ProgramData\\ssh\\sshd_config
     ```  
   - 常见配置项（示例）：
     ```text
     # 在文件中查找并修改或确认：
     Port 22
     PasswordAuthentication yes
     PubkeyAuthentication yes
     # 其他默认项无需改动
     ```

7. **启动并设置 sshd 服务开机自启**  
   ```powershell
   # 启动服务
   Start-Service sshd

   # 设置为自动启动
   Set-Service -Name sshd -StartupType Automatic

   # 确认状态
   Get-Service -Name sshd
   ```
   - 如果 `Status` 为 `Running`，表示 SSHD 已正常启动。  
   - 如果没有出现 “sshd” 服务，请检查上一步安装脚本是否报错，或重新打开 PowerShell 以管理员身份重试 `install-sshd.ps1`。

8. **放行 Windows 防火墙 TCP 22 端口**  
   ```powershell
   New-NetFirewallRule -Name "Allow SSH (TCP 22)" `
       -DisplayName "Allow SSH (TCP 22)" `
       -Enabled True `
       -Direction Inbound `
       -Protocol TCP `
       -Action Allow `
       -LocalPort 22
   ```
   - 该规则将允许来自任意网络（域/专用/公用）的外部机器通过 TCP 22 连接服务器。  
   - 运行完毕后可验证：
     ```powershell
     Get-NetFirewallRule -DisplayName "Allow SSH (TCP 22)" |
       Select-Object DisplayName, Enabled, Direction, Action, LocalPort
     ```

---

### 3. 启用系统内置 OpenSSH Server（如果可用）

如果“Get-WindowsCapability”显示 `OpenSSH.Server~~~~0.0.1.0 State : Installed`，则无需手动下载，只需按以下步骤操作：

1. **以管理员 PowerShell 运行**：
   ```powershell
   # 启动 sshd 服务
   Start-Service sshd

   # 设置为自动启动
   Set-Service -Name sshd -StartupType Automatic
   ```

2. **放行 Windows 防火墙 TCP 22 端口**（与上述第 2.8 节相同）：
   ```powershell
   New-NetFirewallRule -Name "Allow SSH (TCP 22)" `
       -DisplayName "Allow SSH (TCP 22)" `
       -Enabled True -Direction Inbound `
       -Protocol TCP -Action Allow -LocalPort 22
   ```

3. **验证 sshd 服务是否在监听**：
   ```powershell
   netstat -ano | findstr ":22"
   ```
   - 如果能看到 `TCP 0.0.0.0:22  LISTENING  <PID>`，则说明 SSHD 已经在所有接口上监听端口 22。

---

### 4. 服务器本机测试 SSH

在服务器本机打开一个普通用户 PowerShell（无需管理员权限），执行：
```powershell
ssh <SERVER_USER>@localhost
```
- 如果第一次连接，会提示：
  ```
  The authenticity of host 'localhost (127.0.0.1)' can't be established.
  ECDSA key fingerprint is SHA256:...
  Are you sure you want to continue connecting (yes/no)? 
  ```
  输入 `yes` 并回车。  
- 系统会提示输入密码，此处填写服务器本机用户 `<SERVER_USER>` 的 Windows 登录密码（即你的微软账户密码）。  
- 如果登录成功，提示符会变为类似：
  ```
  PS C:\\Users\\<SERVER_USER>>
  ```
  说明服务器上的 SSHD 服务已正常可用。

---

## 三、笔记本端（客户端）测试连通性与登录

在确认服务器端 SSHD 服务已正常启动并监听后，下面在笔记本上验证连通性并登录服务器。

### 1. 验证 Tailscale 网络层连通

在笔记本 PowerShell （普通权限）中执行：
```powershell
tailscale status
```
- 确认能看到服务器的条目（IP <SERVER_IP>）且状态非 `offline`。  
- 例如可能输出：
  ```
  <SERVER_IP>   gqy-computer   sellifake1538409@  windows   active; direct
  <CLIENT_IP> <CLIENT_HOSTNAME> sellifake1538409@ windows idle, tx 532 rx 124
  ```

### 2. Ping 测试基础网络可达性

在笔记本 PowerShell 中执行：
```powershell
ping <SERVER_IP>
```
- 如果能收到类似：
  ```
  Reply from <SERVER_IP>: bytes=32 time=40ms TTL=128
  ```
  的回复，说明 ICMP（Ping）通畅，网络层没问题。

### 3. 测试 TCP 22 端口是否可达

在笔记本 PowerShell 中执行：
```powershell
Test-NetConnection -ComputerName <SERVER_IP> -Port 22
```
- 如果输出中 `TcpTestSucceeded : True`，说明笔记本能够连通到服务器的 SSH 端口 22。  
- 如果 `TcpTestSucceeded : False`，说明 22 端口被阻塞或 SSH 服务未启动，需要回到服务器端检查防火墙与 sshd 状态。

### 4. 直接尝试 SSH 登录

在笔记本 PowerShell 中执行：
```powershell
ssh <SERVER_USER>@<SERVER_IP>
```
- 第一次提示接受指纹（Fingerprint），输入 `yes` 回车，系统会将该指纹写入本地 `~/.ssh/known_hosts`。  
- 随后提示输入密码，如果服务器尚未配置公钥登录，请输入 `<SERVER_USER>` 用户在服务器上的登录密码（微软账户密码）；如果已部署公钥，则直接登录成功。  
- 成功登录后，你会看到远程提示符：
  ```
  Microsoft Windows [版本 10.0.26100.4061]
  (c) Microsoft Corporation。保留所有权利。

  <SERVER_USER>@<SERVER_HOSTNAME> C:\\Users\\<SERVER_USER>>
  ```
  这就表明笔记本已经能够通过 SSH 使用 Tailscale IP 成功登录到服务器。

---

## 四、完整流程概要（结合前面所述各步骤）

1. **检查并安装 SSH 服务（服务器）**  
   - 在服务器 PowerShell 执行  
     ```powershell
     Get-WindowsCapability -Online | Where Name -Like 'OpenSSH.Server*'
     ```  
     - 如果 `State : Installed`，跳到第 3 步；  
     - 否则，通过 DISM 或手动下载 Win32-OpenSSH 进行安装。  
   - 安装完成后，部署 host key（`ssh-keygen -A`）、运行 `install-sshd.ps1`（如果手动安装）。  

2. **启动、配置与验证 SSHD（服务器）**  
   1. 启动 `sshd`：  
      ```powershell
      Start-Service sshd
      Set-Service -Name sshd -StartupType Automatic
      ```  
   2. 放行防火墙 TCP 22：  
      ```powershell
      New-NetFirewallRule -Name "Allow SSH (TCP 22)" `
          -DisplayName "Allow SSH (TCP 22)" `
          -Enabled True -Direction Inbound `
          -Protocol TCP -Action Allow -LocalPort 22
      ```  
   3. 验证本机 SSH：  
      ```powershell
      ssh <SERVER_USER>@localhost
      ```  
      - 首次接受指纹 → 输入 Windows 登录密码 → 出现 `PS C:\\Users\\<SERVER_USER>>` 提示。  

3. **配置客户端 SSH 简写（笔记本）**  
   - 在笔记本 PowerShell 中执行：  
     ```powershell
     New-Item -ItemType Directory -Path $HOME\\.ssh -Force
     notepad $HOME\\.ssh\\config
     ```  
   - 在打开的 `config` 文件里粘贴（并保存）：  
     ```text
     Host gqy-server
         HostName <SERVER_IP>
         User <SERVER_USER>
         Port 22
         IdentityFile ~/.ssh/id_rsa
     ```  
   - 保存后测试：  
     ```powershell
     ssh gqy-server
     ```  
     - 首次接受指纹 → 输入密码 → 出现 `<SERVER_USER>@<SERVER_HOSTNAME> C:\\Users\\<SERVER_USER>>`。  

4. **网络连通性检查（笔记本）**  
   1. `tailscale status` → 确认两端在线并显示对方 IP。  
   2. `ping <SERVER_IP>` → 确认 ICMP 可达。  
   3. `Test-NetConnection -ComputerName <SERVER_IP> -Port 22` → 确认 TCP 22 可达。  
   4. `ssh <SERVER_USER>@<SERVER_IP>`（或 `ssh gqy-server`）→ 最终登录确认。  

---

## 五、注意事项与常见问题

1. **Windows 11 Home 版默认未启用 OpenSSH Server 功能**  
   - 如果 `Get-WindowsCapability` 中找不到 `OpenSSH.Server`，就必须采用“手动安装 Win32-OpenSSH”的方法（第 2 节）。  
   - Home 版也可以通过“添加可选功能”来安装，但在某些公司策略下可能被禁用回显。  

2. **防火墙和网络策略**  
   - Windows Defender 防火墙应当明确放行 TCP 22（“Allow SSH (TCP 22)” 规则）。  
   - 如果服务器有第三方安全软件，务必确认其也已放行 22 端口并允许 `sshd.exe` 启动。  
   - 确认 Tailscale 允许两端直连，当网络质量不佳时，可在 Tailscale 控制面板中手动切换中继节点（DERP），以提升连通性。

3. **公钥认证 vs 密码认证**  
   - 建议在两台机器上都生成或复用一对 SSH 增加相应字符...

> **补充操作：** 如果客户端已经生成过 SSH 密钥对（例如 `~/.ssh/id_rsa` 与 `~/.ssh/id_rsa.pub`），可以直接将公钥复制到服务器上，无需重新生成。具体操作如下：
> 1. 在客户端机器上打开 PowerShell（或命令行），显示公钥内容：
>    ```powershell
>    type $HOME\.ssh\id_rsa.pub
>    ```
> 2. 复制输出的一整行公钥字符串（以 `ssh-rsa` 或 `ssh-ed25519` 开头）。
> 3. 在服务器上以管理员身份打开 PowerShell，执行：
>    ```powershell
>    cd $env:USERPROFILE
>    if (-not (Test-Path .ssh)) {
>        New-Item -ItemType Directory -Path .ssh
>    }
>    notepad .ssh\authorized_keys
>    ```
> 4. 在打开的记事本窗口中将公钥粘贴为新一行，保存并关闭。  
> 5. 确保 `C:\Users\<SERVER_USER>\.ssh` 文件夹和 `authorized_keys` 文件的权限仅限该用户可写。  
> 6. 之后即可在客户端直接使用私钥登录服务器，无需再次输入密码：
>    ```powershell
>    ssh <SERVER_USER>@<SERVER_IP>
>    ```
