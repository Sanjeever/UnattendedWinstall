# UnattendedWinstall Lab

这是一个面向纯内网实验机的 Windows 11 无人值守安装配置。

本仓库 fork 自 [memstechtips/UnattendedWinstall](https://github.com/memstechtips/UnattendedWinstall)。上游项目提供了基于 Winhance 的 Windows 安装后精简、优化和隐私配置；本 fork 在此基础上改成了更适合个人实验环境的“开箱即用”配置。

## 适用场景

这份配置适合：

- VMware / Hyper-V / Proxmox 等虚拟机测试环境。
- 纯内网实验机。
- 需要快速重装、快速进入桌面、快速远程连接的 Windows 11 Pro 环境。
- 需要保留 Edge、Microsoft Store、App Installer，并启用 WSL2 / Hyper-V 的开发实验环境。

这份配置不适合：

- 公网暴露机器。
- 办公网、生产环境、多人共享网络。
- 需要安全基线、账户密码策略、防火墙策略的正式环境。

## 当前配置会做什么

`autounattend.xml` 当前会自动执行以下操作。

### 安装与系统版本

- 使用 Windows Answer File 自动安装。
- 面向 x64 Windows 11 安装介质。
- 设置安装界面、系统语言、区域和输入法为简体中文 `zh-CN`。
- 设置时区为 `China Standard Time`。
- 使用 Windows Pro 通用安装密钥选择 Pro 版本：

```text
VK7JG-NPHTM-C97JM-9MPGT-3V66T
```

这个密钥只用于安装选版，不负责激活。

### 磁盘分区

- 自动清空 `Disk 0`。
- 自动创建 UEFI/GPT 启动布局：
  - EFI 分区
  - MSR 分区
  - Windows 主分区
- 自动安装到 `Disk 0 Partition 3`。

用户进入系统后，通常只会看到 C 盘。

> 注意：`Disk 0` 会被直接清空。真实机器上使用前必须确认目标磁盘。

### 账户与登录

- 启用内置 `Administrator` 用户。
- `Administrator` 使用空密码。
- 长期自动登录 `Administrator`。
- 跳过 OOBE 中的微软账户、联网、无线、EULA、OEM 注册等页面。

### 远程访问

- 开启 RDP。
- 允许空密码本地账户通过 RDP 登录：

```text
LimitBlankPasswordUse = 0
```

- 关闭 Domain / Private / Public 三类 Windows 防火墙配置。

这是为了纯内网实验环境下简单直接地远程连接。

### WSL2 与 Hyper-V

安装后会启用：

- `Microsoft-Windows-Subsystem-Linux`
- `VirtualMachinePlatform`
- `Microsoft-Hyper-V-All`

并设置：

```powershell
bcdedit /set hypervisorlaunchtype Auto
wsl --set-default-version 2
```

注意：这只保证 WSL2 / Hyper-V 能力启用，不会自动安装 Ubuntu 等 Linux 发行版。首次使用 WSL 发行版时仍需执行：

```powershell
wsl --install -d Ubuntu
```

### Winhance 精简与优化

保留上游 Winhance 的大部分精简和优化逻辑，包括：

- 移除大量预装应用。
- 移除 OneDrive。
- 禁用 Recall。
- 禁用 Copilot 相关组件。
- 调整隐私、性能、电源、Explorer、开始菜单、任务栏、Windows Update 等设置。
- 创建 `Install Winhance` 桌面快捷方式。

本 fork 明确保留：

- Microsoft Edge
- Microsoft Store
- App Installer / winget 能力

这样安装完成后仍可方便安装 Docker Desktop、WSL 发行版、开发工具和常用软件。

## 推荐 ISO

推荐使用 Microsoft 官方 Windows 11 x64 multi-edition ISO。

当前已测试：

```text
Win11_25H2_Chinese_Simplified_x64_v2.iso
```

建议使用 Windows 11 Pro。Home 版不适合本配置，因为 RDP Host、Hyper-V 等能力依赖 Pro / Enterprise / Education。

如果目标机器 BIOS / UEFI 内置 Home OEM key，Windows Setup 可能自动倾向 Home。当前配置通过 Pro 通用安装密钥选择 Pro；如仍遇到版本选择问题，可在安装介质 `sources` 目录加入 `ei.cfg`：

```ini
[EditionID]
Professional

[Channel]
Retail

[VL]
0
```

## 使用方法

### 方法一：写入 ISO

1. 下载官方 Windows 11 x64 简体中文 ISO。
2. 使用 AnyBurn 打开 ISO。
3. 将 `autounattend.xml` 添加到 ISO 根目录。
4. 保存生成新的 ISO。
5. 使用新 ISO 启动虚拟机或测试机。

文件必须满足：

- 文件名是 `autounattend.xml`。
- 位置是 ISO 根目录。

### 方法二：U 盘安装

1. 制作 Windows 安装 U 盘。
2. 将 `autounattend.xml` 放到 U 盘根目录。
3. 从 U 盘启动安装。

## VMware 测试建议

- 使用 UEFI 启动。
- 新建空虚拟磁盘。
- 开启虚拟化相关选项，便于 Hyper-V / WSL2 工作。
- 分配至少 8 GB 内存，Docker Desktop / WSL2 会更稳定。

当前已在 VMware 中验证：

- 自动安装成功。
- 自动进入 `Administrator` 桌面。
- Windows 11 Pro 25H2 安装成功。
- `Administrator` 空密码自动登录成功。
- RDP 已开启。
- 空密码远程限制已关闭。
- Windows 防火墙已关闭。
- WSL2 / VirtualMachinePlatform / Hyper-V Windows 功能已启用。
- Edge 已保留。

## 安装后验证

进入系统后，用 PowerShell 执行：

```powershell
whoami
winver
(Get-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Control\Lsa').LimitBlankPasswordUse
(Get-ItemProperty 'HKLM:\System\CurrentControlSet\Control\Terminal Server').fDenyTSConnections
Get-NetFirewallProfile | Select-Object Name, Enabled
'Microsoft-Windows-Subsystem-Linux','VirtualMachinePlatform','Microsoft-Hyper-V-All' | ForEach-Object {
    Get-WindowsOptionalFeature -Online -FeatureName $_ | Select-Object FeatureName, State
}
```

预期结果：

- 当前用户是 `administrator`。
- 系统版本是 Windows 11 Pro。
- `LimitBlankPasswordUse` 为 `0`。
- `fDenyTSConnections` 为 `0`。
- 防火墙 profile 的 `Enabled` 均为 `False`。
- WSL2 / VirtualMachinePlatform / Hyper-V 均为 `Enabled`。

## Docker Desktop

如果需要 Docker Desktop，可在安装完成后执行：

```powershell
winget install -e --id Docker.DockerDesktop --source winget --silent --accept-package-agreements --accept-source-agreements
```

安装完成后启动：

```powershell
Start-Process "$env:ProgramFiles\Docker\Docker\Docker Desktop.exe"
```

Docker Desktop 安装包较大。频繁重装实验机时，建议提前把安装器放到内网文件服务器、VMware 共享目录或自定义 ISO 中。

## 安全说明

这份配置刻意牺牲安全性来换取实验环境的便利性：

- 内置 `Administrator` 启用。
- `Administrator` 空密码。
- 长期自动登录。
- 允许空密码 RDP。
- Windows 防火墙关闭。

只建议在完全可信的纯内网实验环境使用。不要用于公网、办公网或生产环境。

## 上游项目

原始项目：

- [memstechtips/UnattendedWinstall](https://github.com/memstechtips/UnattendedWinstall)

相关项目：

- [memstechtips/Winhance](https://github.com/memstechtips/Winhance)

本 fork 主要保留上游的 Winhance 精简和优化能力，同时增加实验机无人值守安装、空密码 Administrator 自动登录、RDP、WSL2 / Hyper-V、保留 Edge / Store 等本地定制。
