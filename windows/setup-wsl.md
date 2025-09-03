# Windows 安装与排错：WSL2

本文档中文描述，命令与路径保持英文。

## 常见错误：HCS_E_SERVICE_NOT_AVAILABLE
含义：Host Compute Service（HCS）相关服务不可用，通常是虚拟化/Hyper-V/WSL2 相关特性未启用或服务未启动。

## 一、前置检查（BIOS/虚拟化）
- 任务管理器 → 性能 → Virtualization=Enabled；若 Disabled，请进 BIOS 启用 Intel VT-x/AMD SVM，重启。

## 二、启用所需 Windows 功能（管理员 PowerShell）
```powershell
dism /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
dism /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
:: 可选（Pro/Enterprise）：Hyper-V 全量特性
dism /online /enable-feature /featurename:Microsoft-Hyper-V-All /all /norestart
```
- 执行后请重启一次。

## 三、检查并启动相关服务（管理员 PowerShell）
```powershell
Get-Service vmcompute, LxssManager, hns | Select-Object Name, Status
Start-Service vmcompute
Start-Service LxssManager
Start-Service hns
```
- 若服务不存在或启动失败，回到第二步确认功能已启用并重启。

## 四、更新并设定 WSL2
```powershell
wsl --update
wsl --set-default-version 2
wsl --install -d Ubuntu
```
- 如仍报错，可先完全关闭/重装 WSL 特性：
  - 关闭：在“打开或关闭 Windows 功能”中，取消勾选 “Windows Subsystem for Linux”、“Virtual Machine Platform”，重启
  - 再次勾选启用上述两项，重启后重复本节命令

## 五、系统修复（若系统组件异常）
```powershell
sfc /scannow
DISM /Online /Cleanup-Image /RestoreHealth
```
- 执行后重启。

## 六、Docker Desktop 备用方案
- 若短期内无法修复 WSL2，且你是 Windows Pro/Enterprise，可尝试 Docker Desktop Hyper-V 后端（Settings → General → 取消勾选 Use the WSL 2 based engine）。
- Windows Home 通常需要 WSL2 支持。

## 七、完成后验证
```powershell
wsl --status
wsl -l -v
```

## 补充：快速排错命令（管理员 PowerShell）
```powershell
:: 查看相关功能是否启用
dism /online /get-features /format:table ^| findstr /i VirtualMachinePlatform Microsoft-Windows-Subsystem-Linux

:: 启用必要功能（执行后重启）
dism /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
dism /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

:: （可选，Pro/Enterprise）启用 Hyper-V 全量特性
dism /online /enable-feature /featurename:Microsoft-Hyper-V-All /all /norestart

:: 确保 Hypervisor 启动
bcdedit /set hypervisorlaunchtype Auto

:: 检查并启动相关服务
Get-Service vmcompute, LxssManager, hns ^| ft Name,Status,StartType
Set-Service vmcompute -StartupType Automatic; Start-Service vmcompute
Set-Service LxssManager -StartupType Automatic; Start-Service LxssManager
Set-Service hns -StartupType Automatic; Start-Service hns

:: 更新并设定 WSL2
wsl --update
wsl --set-default-version 2
wsl --install -d Ubuntu

:: 若仍异常，系统修复后重试
sfc /scannow
DISM /Online /Cleanup-Image /RestoreHealth
```

## 备用方案：Docker Desktop Hyper-V 后端（无 WSL2 情况）
- 前提：Windows Pro/Enterprise；启用 Hyper-V（见上）。
- Docker Desktop → Settings → General：取消勾选 Use the WSL 2 based engine。
- 正常启动后，继续使用 compose 起 Postgres+pgvector。
