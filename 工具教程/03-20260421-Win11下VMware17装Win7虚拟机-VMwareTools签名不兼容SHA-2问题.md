---
title: Win11 下 VMware 17 装 Win7 虚拟机：VMware Tools 签名不兼容（SHA-2）的解决方案
description: 记录在 Win11 宿主机上通过 Broadcom 官网安装 VMware 17 Pro，并创建 Win7 虚拟机时遇到的 VMware Tools 安装失败问题。根因是 Win7 原生只认 SHA-1 签名，而新版 Tools 驱动使用 SHA-2 签名，Win7 "看不懂" 而拒装。通过微软补丁 KB4490628 + KB4474419 可彻底解决。
#多个标签请使用英文逗号分隔或使用数组语法
tags: 虚拟机, VMWare, VMWare Tools, Win7, SHA-2 签名, KB4474419, KB4490628
#多个分类请使用英文逗号分隔或使用数组语法，暂不支持多级分类
category: 工具教程
---

## 导读

换了一台新笔记本（ThinkBook 16 225H，预装 Win11），准备重新安装 VMware + Win7 虚拟机。时隔一年半再次走这个流程，才发现环境已经很不一样了：

- **VMware 已被 Broadcom 收购**，Workstation Pro 变成了**个人免费**，可以直接从官网下载，不再需要破解版
- **新版 VMware Tools 的驱动**全部采用 **SHA-2 签名**，而 Win7 原生只认 SHA-1，结果在装 Tools 时各种驱动报错、安装回滚
- 顺带还遇到了 Win7 不认 USB 3.0 U 盘 的小坑

本文是旧文 [《VMWare 安装与拖动文件到 Win7 虚拟机》](https://www.cnblogs.com/BensonLaur/p/18472405) 的续集，**只记录这次踩到的新坑**。VMware 的基础安装和 Win7 虚拟机的创建这里不再重复，请参见旧文。

## 1. 官网下载 VMware 17 Pro（免费但繁琐）

Broadcom 收购 VMware 后，Workstation Pro 对个人用户免费，但下载流程不像过去那样点一下就行，需要：

- 注册 Broadcom 账号
- 登录 Broadcom Support Portal
- 同意一长串协议（据说协议不点开就不能勾选）
- 才能拿到下载链接

整个过程相当折腾，参考这篇知乎文章按步骤操作即可：[官网直达！VMware 17 Pro 下载安装避坑手册 - 知乎](https://zhuanlan.zhihu.com/p/1974155096729331109)

文章评论区吐槽很真实，可以感受一下官方下载流程有多逆天：

  <img src="resource\03-A-知乎评论区吐槽.png" alt="知乎评论区对官网下载流程的吐槽" width="500" style="display:block; margin:auto;">
  <div style="text-align: center;">知乎评论区对官网下载流程的吐槽</div>
  <br/>

装完后确认一下版本，可以看到已经是 **VMware Workstation 17 Pro 17.6.4**，版权方是 **Broadcom**：

  <img src="resource\03-A-VMware17About版本信息.png" alt="VMware 17 Pro 17.6.4 版本信息" width="500" style="display:block; margin:auto;">
  <div style="text-align: center;">VMware 17 Pro 17.6.4 版本信息（Broadcom）</div>
  <br/>

## 2. 创建 Win7 虚拟机

这步和旧文基本一致，使用的是同一个 ISO 镜像：`cn_windows_7_ultimate_with_sp1_x86_dvd_u_677486.iso`（Win7 旗舰版 32 位 SP1）。

激活码这次换了个参考来源：[Win7激活码大全 - jb51.net](https://www.jb51.net/os/windows/922470.html)，挑一个能用的即可。

具体的新建虚拟机步骤见旧文，不再赘述。

## 3. 踩坑：VMware Tools 安装失败

装完 Win7 之后依然是熟悉的问题——**没法从主机拖拽文件进虚拟机**，需要装 VMware Tools。但这一次和旧文不同的是：

### 3.1 `Install VMware Tools` 菜单灰色

在新版 VMware 17.6.4 中，`VM → Install VMware Tools` 菜单是灰色的，**点不了**：

  <img src="resource\03-B-InstallVMwareTools菜单灰色.png" alt="Install VMware Tools 菜单灰色" width="500" style="display:block; margin:auto;">
  <div style="text-align: center;">Install VMware Tools 菜单灰色</div>
  <br/>

解决办法是**手动挂载 VMware 自带的 Tools ISO**。VMware 安装目录下其实已经自带了 ISO，不需要额外下载。

32 位 Win7 要挂载的 ISO 路径：

```
C:\Program Files (x86)\VMware\VMware Workstation\windows-x86.iso
```

> 💡 **为什么是 `windows-x86.iso` 而不是 `windows.iso`？**
>
> 从 **VMware Tools 12.5.0（2024 年 10 月发布）** 开始，官方把 32 位支持从 `windows.iso` 中剥离出去：
>
> | ISO | 支持系统 | 对应 Tools 版本 |
> |---|---|---|
> | `windows-x86.iso` | **32 位** Windows（含 Win7 32-bit） | 12.4.5（32 位最后一版） |
> | `windows.iso` | **64 位** Windows | 12.5.0 及之后（仅 64 位） |
>
> 所以 32 位 Win7 只能挂 `windows-x86.iso`，对应 Tools 版本也固定在 12.4.5 不再更新。

挂载方式：虚拟机 → 设置 → CD/DVD → 使用 ISO 映像文件 → 浏览到上面那个路径。

### 3.2 运行 setup.exe 后驱动连环报错

挂载 ISO 后进 Win7 虚拟机，打开光驱里的 `setup.exe`（32 位系统用这个，不是 `setup64.exe`），"典型安装"一路下一步。结果安装到一半弹出一连串错误：

**① VSock 驱动装不上：**

  <img src="resource\03-C-VSock驱动安装失败.png" alt="VSock 驱动安装失败" width="500" style="display:block; margin:auto;">
  <div style="text-align: center;">VSock 驱动安装失败</div>
  <br/>

**② 弹窗"Windows 无法验证此驱动程序软件的发布者"，即使选"始终安装此驱动程序软件"也救不回来：**

  <img src="resource\03-C-Windows无法验证驱动发布者.png" alt="Windows 无法验证驱动发布者" width="500" style="display:block; margin:auto;">
  <div style="text-align: center;">Windows 无法验证驱动程序软件的发布者</div>
  <br/>

**③ 内存控制器驱动也装不上：**

  <img src="resource\03-C-内存控制器驱动安装失败.png" alt="内存控制器驱动安装失败" width="500" style="display:block; margin:auto;">
  <div style="text-align: center;">内存控制器驱动安装失败</div>
  <br/>

**④ 最终安装回滚：**

  <img src="resource\03-C-正在回滚.png" alt="正在回滚操作" width="500" style="display:block; margin:auto;">
  <div style="text-align: center;">正在回滚操作</div>
  <br/>

**⑤ 提示"VMware Tools 安装向导提前结束"：**

  <img src="resource\03-C-安装向导提前结束.png" alt="VMware Tools 安装向导提前结束" width="500" style="display:block; margin:auto;">
  <div style="text-align: center;">VMware Tools 安装向导提前结束</div>
  <br/>

## 4. 根因分析：SHA-2 vs SHA-1 签名

报错的本质**不是驱动坏了，而是 Win7 看不懂新签名**。

### 签名机制背景

Windows 安装驱动时会校验**数字签名**，确认驱动可信且未被篡改。签名依赖**哈希算法**：

- 过去使用 **SHA-1**
- SHA-1 在 2017 年 2 月被 Google 公开演示过碰撞攻击（SHAttered），确认存在安全漏洞
- 从 2019 年 8 月 13 日起，**Windows Updates 本身**开始改用 **SHA-2** 签名；与此同时，整个 Windows 驱动签名生态也逐步迁移到 SHA-2 证书，新发布的驱动包普遍使用 SHA-2 签名

### 为什么会冲突

| 组件 | 发布时间 | 签名算法 |
|---|---|---|
| Windows 7 | 2009 年 | 原生只认 **SHA-1** |
| VMware 17 Pro（及其自带 Tools） | 2024+ | 驱动全部 **SHA-2** 签名 |

Win7 没有 SHA-2 验证能力 → 认为 VSock、内存控制器等驱动"签名不可信" → 拒绝安装 → 整个 VMware Tools 安装失败回滚。

**一句话总结：** Win7 是 2009 年的老系统，不认识 2019 年之后的新签名算法。需要给 Win7 打上微软官方补丁，让它"学会"验证 SHA-2 签名。

## 5. 解决方案：打两个补丁

需要按顺序装两个微软官方补丁：

| 补丁 | 作用 |
|---|---|
| **KB4490628** | 服务堆栈更新（SSU）。服务堆栈是 Windows 中负责安装其他更新的核心组件，相当于更新系统的"发动机"。必须先升级它，才能正确安装后续补丁。 |
| **KB4474419** | SHA-2 代码签名支持更新。给 Win7 的代码完整性验证模块（CI.dll）添加 SHA-2 算法支持。装完后，Win7 就"学会"了如何验证 SHA-2 签名的驱动。 |

### 5.1 下载补丁（在 Win11 主机进行）

打开微软官方更新目录：

```
https://www.catalog.update.microsoft.com/
```

**下载 KB4490628**（服务堆栈更新）：

- 搜索 `KB4490628`
- 32 位 Win7 选 `Windows 7 + x86`（约 4 MB）
- 64 位 Win7 选 `Windows 7 + x64`（约 9 MB）
- ⚠️ 不要选 `Windows Embedded Standard 7`、`Server 2008` 或 `Itanium` 版本

  <img src="resource\03-E-KB4490628搜索结果.png" alt="KB4490628 搜索结果" width="500" style="display:block; margin:auto;">
  <div style="text-align: center;">KB4490628 搜索结果（选 Windows 7 x86）</div>
  <br/>

**下载 KB4474419**（SHA-2 签名支持）：

- 搜索 `KB4474419`
- 选**最新日期**的版本（推荐 2019-09 或之后）
- 32 位 Win7 选 `Windows 7 + x86`（约 36 MB）
- 64 位 Win7 选 `Windows 7 + x64`（约 53 MB）

  <img src="resource\03-E-KB4474419搜索结果.png" alt="KB4474419 搜索结果" width="500" style="display:block; margin:auto;">
  <div style="text-align: center;">KB4474419 搜索结果（选 Windows 7 x86）</div>
  <br/>

### 5.2 把补丁传进虚拟机：USB 3.0 的小坑

一开始我想直接在 Win7 虚拟机里用 IE 访问微软更新目录下载补丁，结果 **IE 8 访问 `https` 的微软目录站点失败**（和旧文里下载浏览器失败是同一个问题）。

于是退而求其次：**主机下载后用 U 盘拷进虚拟机**。但是插上 U 盘后，Win7 提示"无法识别的 USB 设备"：

  <img src="resource\03-E-无法识别USB设备.png" alt="Win7 无法识别 USB 设备" width="500" style="display:block; margin:auto;">
  <div style="text-align: center;">Win7 提示"无法识别的 USB 设备"</div>
  <br/>

原因：**Win7 原生不支持 USB 3.0**。

解决方法很简单——**换一个 USB 2.0 的 U 盘即可**，虚拟机里就能正常识别了（不需要改虚拟机的 USB 控制器设置）。

连接方式：插上 U 盘后，在 VMware 菜单里选 `VM → Removable Devices → 你的 U 盘 → Connect (Disconnect from Host)`，U 盘就会从主机断开，转接到虚拟机里，这时 Win7 的"计算机"里就会出现 U 盘盘符。

### 5.3 严格按顺序安装（顺序很重要）

把两个 `.msu` 文件拷进 Win7 桌面后，**严格按如下顺序**操作，每一步都要重启：

1. 双击 **KB4490628**（小的那个，4 MB）→ 确认安装 → **重启 Win7** ⚠️
2. 双击 **KB4474419**（大的那个，36 MB）→ 确认安装 → **重启 Win7** ⚠️
3. 重新运行光驱里的 `setup.exe`（32 位用这个，**不是 `setup64.exe`**），安装 VMware Tools → **重启 Win7** ⚠️

这次 VSock 和内存控制器驱动都能正常通过签名验证了，VMware Tools 安装成功：

  <img src="resource\03-E-VMwareTools安装成功.png" alt="VMware Tools 安装成功" width="500" style="display:block; margin:auto;">
  <div style="text-align: center;">VMware Tools 安装成功</div>
  <br/>

> ⚠️ 顺序不能颠倒：必须先装 KB4490628（服务堆栈）再装 KB4474419，颠倒顺序可能导致 KB4474419 装不上或生效不完整。每装完一个补丁都要重启，不要省略。

## 6. 验证：可以拖文件了

重启后，从 Win11 主机直接拖文件到 Win7 虚拟机，成功：

  <img src="resource\03-F-成功拖动文件.png" alt="从主机成功拖动文件到 Win7 虚拟机" width="600" style="display:block; margin:auto;">
  <div style="text-align: center;">从 Win11 主机成功拖动文件到 Win7 虚拟机</div>
  <br/>

主机和虚拟机之间复制粘贴文本、虚拟机分辨率自适应窗口，这些功能也一并恢复。

## 7. 踩坑总结

| 坑 | 现象 | 解决方法 |
|---|---|---|
| Broadcom 官网下载繁琐 | 注册、协议一堆步骤 | 参考知乎文章按步骤操作 |
| `Install VMware Tools` 菜单灰色 | 点不了 | 手动挂载 `windows-x86.iso`（32 位）或 `windows.iso`（64 位） |
| VSock / 内存控制器驱动装不上 | 弹窗报错，安装回滚 | 打上 **KB4490628 + KB4474419** 两个补丁 |
| IE 8 访问微软更新目录失败 | `https` 握手问题 | 改用主机下载 + U 盘传输 |
| U 盘无法识别 | Win7 提示"无法识别的 USB 设备" | 换 USB 2.0 U 盘 |
| 直接装 KB4474419 没效果 | 装不上或生效不完整 | 必须先装 KB4490628 再装 KB4474419 |

## 8. 参考资料

- 旧文（基础安装流程）：[VMWare 安装与拖动文件到 Win7 虚拟机](https://www.cnblogs.com/BensonLaur/p/18472405)
- [官网直达！VMware 17 Pro 下载安装避坑手册 - 知乎](https://zhuanlan.zhihu.com/p/1974155096729331109)
- [Win7激活码大全 - jb51.net](https://www.jb51.net/os/windows/922470.html)
- [Microsoft Update Catalog](https://www.catalog.update.microsoft.com/)
- [VMware Tools 12.5.0 is ready for the latest Windows operating systems - VMware Blog](https://blogs.vmware.com/cloud-foundation/2024/11/07/vmware-tools-12-5-0-is-ready-for-the-latest-windows-operating-systems/)

-------------------------

*本文源地址待同步后补充*
