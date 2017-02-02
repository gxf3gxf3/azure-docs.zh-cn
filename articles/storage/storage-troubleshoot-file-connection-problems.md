---
title: "排查 Azure 文件存储问题 | Microsoft Docs"
description: "排查 Azure 文件存储问题"
services: storage
documentationcenter: 
author: genlin
manager: felixwu
editor: na
tags: storage
ms.assetid: fbc5f600-131e-4b99-828a-42d0de85fff7
ms.service: storage
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/13/2016
ms.author: genli
translationtype: Human Translation
ms.sourcegitcommit: dcda8b30adde930ab373a087d6955b900365c4cc
ms.openlocfilehash: 71da2f8aaa994c8cfc48f968a5275f7f79604251


---
# <a name="troubleshooting-azure-file-storage-problems"></a>排查 Azure 文件存储问题
本文列出了从 Windows 和 Linux 客户端进行连接时，与 Microsoft Azure 文件存储相关的常见问题。 还提供了这些问题的可能原因和解决方法。

**Windows 和 Linux 客户端中的常见问题**

* [尝试打开文件时配额出错](#quotaerror)
* [从 Windows 或 Linux 访问 Azure 文件存储时性能不佳](#slowboth)

**Windows 客户端问题**

* [从 Windows 8.1 或 Windows Server 2012 R2 访问 Azure 文件存储时性能不佳](#windowsslow)
* [尝试装载 Azure 文件共享时出现错误 53](#error53)
* [Net use 成功，但Windows Explorer 中未显示已装载的 Azure 文件共享](#netuse)
* [我的存储帐户包含 "/" 且 net use 命令失败](#slashfails)
* [我的应用程序/服务无法访问装载的 Azure 文件驱动器。](#accessfiledrive)
* [优化性能的其他建议](#additional)

**Linux 客户端问题**

* [将文件上传/复制到 Azure 文件时，出现错误“要将该文件复制到的目标不支持加密”](#encryption)
* [在装载点上执行列表命令时，现有的文件共享上出现错误“主机已关闭”或外壳挂起](#errorhold)
* [尝试在 Linux VM 上装载 Azure 文件时，出现装载错误 115](#error15)
* [Linux VM 在类似 "ls" 的命令中出现随机延迟](#delayproblem)

<a id="quotaerror"></a>

## <a name="quota-error-when-trying-to-open-a-file"></a>尝试打开文件时配额出错
在 Windows 中，将收到下述类似错误消息：

**1816 ERROR_NOT_ENOUGH_QUOTA <--> 0xc0000044**

**STATUS_QUOTA_EXCEEDED**

**处理此命令的配额不够**

**无效句柄值 GetLastError：53**

在 Linux 上，将收到下述类似错误消息：

**<filename> [权限被拒绝]**

**超出磁盘配额**

### <a name="cause"></a>原因
由于已达到文件允许的并发打开句柄数上限，因此出现问题。

### <a name="solution"></a>解决方案
关闭一些句柄，减少并发打开句柄的数量，然后重试。 有关详细信息，请参阅 [Microsoft Azure 存储性能和可伸缩性清单](storage-performance-checklist.md)。

<a id="slowboth"></a>

## <a name="slow-performance-when-accessing-file-storage-from-windows-or-linux"></a>从 Windows 或 Linux 访问文件存储时性能不佳
* 如果没有特定的最低 I/O 大小要求，建议 I/O 大小为 1 MB 以实现最佳性能。
* 如果知道通过写入扩展的最终文件大小，并且文件上尚未写入的结尾包含零时软件不存在兼容性问题，请提前设置文件大小，而不是使每次写入都成为扩展写入。

<a id="windowsslow"></a>

## <a name="slow-performance-when-accessing-the-file-storage-from-windows-81-or-windows-server-2012-r2"></a>从 Windows 8.1 或 Windows Server 2012 R2 访问文件存储时性能不佳
对于正在运行 Windows 8.1 或 Windows Server 2012 R2 的客户端，请确保安装了修补程序 [KB3114025](https://support.microsoft.com/kb/3114025)。 此修补程序可改善创建和关闭句柄的性能。

可运行以下脚本，检查是否已安装修补程序：

`reg query HKLM\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters\Policies`

如果安装了修补程序，将显示以下输出：

**HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters\Policies**

**{96c345ef-3cac-477b-8fcd-bea1a564241c}    REG_DWORD    0x1**

> [!NOTE]
> 从 2015 年 12 月起，Azure 应用商店中的 Windows server 2012 R2 映像将默认安装修补程序 KB3114025。
>
>

<a id="additional"></a>

## <a name="additional-recommendations-to-optimize-performance"></a>优化性能的其他建议
绝不要为请求写入权限而未请求读取权限的缓存 I/O 创建或打开文件。 也就是说，调用 **CreateFile()** 时，不要只指定 **GENERIC_WRITE**，而始终要指定 **GENERIC_READ | GENERIC_WRITE**。 只写句柄无法本地缓存小型写入，即使它是文件中唯一打开句柄。 这将严重影响小型写入的性能。 请注意，CRT **fopen()** 的 "a" 模式仅打开只写句柄。

<a id="error53"></a>

## <a name="error-53-or-error-67-when-you-try-to-mount-or-unmount-an-azure-file-share"></a>尝试装载或卸载 Azure 文件共享时发生“错误 53”或“错误 67”
以下情况可能引起此问题：

### <a name="cause-1"></a>原因 1
“发生系统错误 53。 访问被拒绝。” 出于安全原因，如果通信通道未加密，并且连接尝试并非来自 Azure 文件共享驻留的同一数据中心，则将阻止连接到 Azure 文件共享。 如果用户的客户端 OS 不支持 SMB 加密，则不提供通信通道加密。 如果用户尝试从本地或不同的数据中心装载文件共享， 这将表示为“发生系统错误 53。访问被拒绝”错误消息。 Windows 8、Windows Server 2012 及更高版本的每个协商请求包括支持加密的 SMB 3.0。

### <a name="solution-for-cause-1"></a>原因 1 的解决方案
使用符合 Windows 8、Windows Server 2012 或更高版本要求的客户端进行连接，或者连接使用的虚拟机要位于 Azure 文件共享时所用的 Azure 存储帐户所在的同一数据中心。

### <a name="cause-2"></a>原因 2
如果端口 445 到 Azure 文件数据中心的出站通信受阻，则装载 Azure 文件共享时可能出现“系统错误 53”或“系统错误 67”。 单击[此处](http://social.technet.microsoft.com/wiki/contents/articles/32346.azure-summary-of-isps-that-allow-disallow-access-from-port-445.aspx)，概括了解允许或禁止从端口 445 访问的 ISP。

Comcast 和某些 IT 组织阻止此端口。 若要了解“系统错误 53”是否由此造成，可使用 Portqry 查询 TCP:445 终结点。 如果 TCP:445 终结点显示为“已筛选”，则 TCP 端口被阻止。 下面是一个示例查询：

`g:\DataDump\Tools\Portqry>PortQry.exe -n [storage account name].file.core.windows.net -p TCP -e 445`

如果 TCP 445 遭到网络路径上的某条规则阻止，将显示以下输出：

**TCP port 445 (microsoft-ds service): FILTERED**

若要深入了解如何使用 Portqry，请参阅 [Portqry.exe 命令行实用工具说明](https://support.microsoft.com/kb/310099)。

### <a name="solution-for-cause-2"></a>原因 2 的解决方案
配合 IT 组织打开到 [Azure IP 范围](https://www.microsoft.com/download/details.aspx?id=41653)的端口 445 出站通信。

### <a name="cause-3"></a>原因 3
如果客户端上启用了 NTLMv1 通信，也可能收到“系统错误 53”。 启用 NTLMv1 将创建安全级别较低的客户端。 使通信受到 Azure 文件阻止。 若要验证这是否是错误原因，可验证以下注册表子项的值是否设置为 3：

HKLM\SYSTEM\CurrentControlSet\Control\Lsa > LmCompatibilityLevel.

有关详细信息，请参阅 TechNet 上的 [LmCompatibilityLevel](https://technet.microsoft.com/library/cc960646.aspx) 主题。

### <a name="solution-for-cause-3"></a>原因 3 的解决方案
若要解决此问题，可将 HKLM\SYSTEM\CurrentControlSet\Control\Lsa 注册表项的 LmCompatibilityLevel 值恢复为默认值 3。

Azure 文件仅支持 NTLMv2 身份验证。 请确保客户端应用了组策略。 这将防止发生此错误。 这也被视为最佳安全方案。 有关详细信息，请参阅[如何使用组策略配置客户端以使用 NTLMv2](https://technet.microsoft.com/library/jj852207\(v=ws.11\).aspx)

建议的策略设置为“仅发送 NTLMv2 响应”。 这与注册表值 3 对应。 客户端仅使用 NTLMv2 身份验证，如果服务器支持，则使用 NTLMv2 会话安全。 域控制器接受 LM、NTLM 和 NTLMv2 身份验证。

<a id="netuse"></a>

## <a name="net-use-was-successful-but-dont-see-the-azure-file-share-mounted-in-windows-explorer"></a>Net use 成功，但 Windows Explorer 中未显示已装载的 Azure 文件共享
### <a name="cause"></a>原因
默认情况下，Windows Explorer 不以管理员身份运行。 如果从管理员命令提示符中运行“net use”，可“以管理员身份”映射网络驱动器。 由于映射的驱动器以用户为中心，如果不同用户帐户下已装载这些驱动器，则已登录的用户帐户将不显示它们。

### <a name="solution"></a>解决方案
从非管理员命令行中装载共享。 此外，可按照 [TechNet 主题](https://technet.microsoft.com/library/ee844140.aspx)配置 **EnableLinkedConnections** 注册表值。

<a id="slashfails"></a>

## <a name="my-storage-account-contains--and-the-net-use-command-fails"></a>我的存储帐户包含“/”且 net use 命令失败
### <a name="cause"></a>原因
如果 **net use** 命令在命令提示符 (cmd.exe) 下运行，则通过添加“/”作为命令行选项进行分析。 这会导致驱动器映射失败。

### <a name="solution"></a>解决方案
可使用下述某个步骤来解决该问题：

•    使用以下 PowerShell 命令：

`New-SmbMapping -LocalPath y: -RemotePath \\server\share  -UserName acountName -Password "password can contain / and \ etc"`

可在批处理文件中按以下方式解决问题

`Echo new-smbMapping ... | powershell -command –`

•    用双引号将密钥括起来 - 除非“/”是第一个字符。 如果是，可使用交互模式并单独输入密码，或者生成密钥来获取不以正斜杠字符 (/) 开头的密钥。

<a id="accessfiledrive"></a>

## <a name="my-applicationservice-cannot-access-mounted-azure-files-drive"></a>我的应用程序/服务无法访问装载的 Azure 文件驱动器
### <a name="cause"></a>原因
每个用户都装载了驱动器。 如果应用程序或服务正在不同的用户帐户下运行，则用户无法看到驱动器。

### <a name="solution"></a>解决方案
从应用程序所在的同一用户帐户下装载驱动器。 可使用类似 psexec 的工具来完成操作。

或者，可创建与网络服务或系统帐户特权相同的新用户，然后在该帐户下运行 **cmdkey** 和 **net use**。 用户名应为存储帐户名，密码应为存储帐户密钥。 **net use** 还可用于传递 **net use** 命令的用户名和密码参数中的存储帐户名和密钥。

按照这些说明操作后， 如果为系统/网络服务帐户运行 **net use**可能会收到以下错误消息：“发生系统错误 1312。指定的登录会话不存在。可能已终止。” 若发生此情况，请确保传递到 **net use** 的用户名包括域信息，例如“[storage account name].file.core.windows.net”。

<a id="encryption"></a>

## <a name="error-you-are-copying-a-file-to-a-destination-that-does-not-support-encryption"></a>出现错误“要将该文件复制到的目标不支持加密”
### <a name="cause"></a>原因
可将 Bitlocker 加密的文件复制到 Azure 文件。 但是，文件存储不支持 NTFS EFS。 因此，此时很可能要使用 EFS。 如果文件已通过 EFS 加密，文件存储的复制操作可能会失败，除非复制命令对复制的文件进行解密。

### <a name="workaround"></a>解决方法
必须先将文件解密，才能将其复制到文件存储。 可使用下述某个方法来执行此操作：

•    使用 **copy /d**。

•    设置以下注册表项：

* 路径 = HKLM\Software\Policies\Microsoft\Windows\System
* 值类型 = DWORD
* 名称 = CopyFileAllowDecryptedRemoteDestination
* 值 = 1

但请注意，设置注册表项会影响网络共享的所有复制操作。

<a id="errorhold"></a>

## <a name="host-is-down-error-on-existing-file-shares-or-the-shell-hangs-when-you-run-list-commands-on-the-mount-point"></a>在装载点上运行列表命令时，现有的文件共享上出现错误“主机已关闭”或外壳挂起
### <a name="cause"></a>原因
如果客户端长时间处于空闲状态，Linux 客户端上将发生此错误。 此时，客户端将断开连接，客户端连接超时。

### <a name="solution"></a>解决方案
Linux 内核中已将该问题作为[变更集](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/fs/cifs?id=4fcd1813e6404dd4420c7d12fb483f9320f0bf93)的一部分进行了修复，该变更集正在等待向后移植到 Linux 分发中。

若要解决此问题，可在要定期写入的 Azure 文件共享中放置一个文件，保持连接并避免客户端处于空闲状态。 这必须是一个写入操作，例如在文件上重写创建/修改的日期。 否则，结果可能会延迟，操作可能不会触发连接。

<a id="error15"></a>

## <a name="mount-error-115-when-you-try-to-mount-azure-files-on-the-linux-vm"></a>尝试在 Linux VM 上装载 Azure 文件时，出现“装载错误 115”
### <a name="cause"></a>原因
Linux 分发尚不支持 SMB 3.0 中的加密功能。 在某些分发中，如果用户尝试使用 SMB 3.0，可能会由于缺少此功能而收到“115”错误信息。

### <a name="solution"></a>解决方案
如果使用的 Linux SMB 客户端不支持加密，则使用文件存储帐户所在的同一数据中心上 Linux VM 的 SMB 2.1 装载文件。

<a id="delayproblem"></a>

## <a name="linux-vm-experiencing-random-delays-in-commands-like-ls"></a>Linux VM 在类似“ls”的命令中出现随机延迟
### <a name="cause"></a>原因
如果装载命令不包括“serverino”选项，则可能发生此错误。 没有“serverino”，Is 命令将在每个文件上运行 **stat**。

### <a name="solution"></a>解决方案
在“/etc/fstab”项中检查“serverino”：

//azureuser.file.core.windows.net/wms/comer on /home/sampledir type cifs (rw,nodev,relatime,vers=2.1,sec=ntlmssp,cache=strict,username=xxx,domain=X, file_mode=0755,dir_mode=0755,serverino,rsize=65536,wsize=65536,actimeo=1)

如果“serverino”选项不存在，可通过选中“serverino”选项卸载并再次装载 Azure 文件。

## <a name="learn-more"></a>了解详细信息
* [在 Windows 上开始使用 Azure 文件存储](storage-dotnet-how-to-use-files.md)
* [在 Linux 上实现 Azure 文件存储入门](storage-how-to-use-files-linux.md)



<!--HONumber=Dec16_HO2-->

