---
title: 快速入门 - 在 Azure 门户中创建 Windows VM | Microsoft Docs
description: 本快速入门介绍了如何使用 Azure 门户创建 Windows 虚拟机
services: virtual-machines-windows
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 05/09/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: c28686c3b6494a0cf8938d39ab9b8338de7aa0c1
ms.sourcegitcommit: 909469bf17211be40ea24a981c3e0331ea182996
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/10/2018
---
# <a name="quickstart-create-a-windows-virtual-machine-in-the-azure-portal"></a>快速入门：在 Azure 门户中创建 Windows 虚拟机

可以通过 Azure 门户创建 Azure 虚拟机 (VM)。 此方法提供基于浏览器的用户界面来创建 VM 及其相关资源。 本快速入门展示了如何使用 Azure 门户在 Azure 中部署运行 Windows Server 2016 的虚拟机 (VM)。 若要查看运行中的 VM，可以通过 RDP 登录到该 VM 并安装 IIS Web 服务器。

如果你还没有 Azure 订阅，可以在开始前创建一个 [免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="log-in-to-azure"></a>登录 Azure

通过 https://portal.azure.com 登录到 Azure 门户。

## <a name="create-virtual-machine"></a>创建虚拟机

1. 在 Azure 门户的左上角选择“创建资源”。

2. 在 Azure Marketplace 资源列表上方的搜索框中，搜索并选择“Windows Server 2016 Datacenter”，然后选择“创建”。

3. 提供 VM 名称，例如 *myVM*，将磁盘类型保留为 *SSD*，然后提供用户名，例如 *azureuser*。 密码必须至少 12 个字符长，且符合[定义的复杂性要求](faq.md#what-are-the-password-requirements-when-creating-a-vm)。

    ![在门户边栏选项卡中输入 VM 的基本信息](./media/quick-create-portal/create-windows-vm-portal-basic-blade.png)

5. 选择“新建”资源组，然后提供一个名称，例如 *myResourceGroup*。 选择所需**位置**，然后选择“确定”。

4. 为 VM 选择大小。 例如，可以按*计算类型*或*磁盘类型*进行筛选。 建议的 VM 大小是 *D2s_v3*。

    ![显示 VM 大小的屏幕截图](./media/quick-create-portal/create-windows-vm-portal-sizes.png)

5. 在“设置”下，保留所有默认设置，然后选择“确定”。

6. 在“摘要”页上，选择“创建”以启动 VM 部署。

7. VM 将固定到 Azure 门户仪表板。 完成部署后，会自动打开 VM 摘要。

## <a name="connect-to-virtual-machine"></a>连接到虚拟机

创建到虚拟机的远程桌面连接。 这些说明指明了如何从 Windows 计算机连接到 VM。 在 Mac 上，需要一个 RDP 客户端，例如 Mac 应用商店提供的这个[远程桌面客户端](https://itunes.apple.com/us/app/microsoft-remote-desktop/id715768417?mt=12)。

1. 单击虚拟机属性页上的“连接”按钮。 

    ![从门户连接到 Azure VM](./media/quick-create-portal/quick-create-portal/portal-quick-start-9.png)
    
2. 在“连接到虚拟机”页面中，保留默认选项，以使用 DNS 名称通过端口 3389 进行连接，然后单击“下载 RDP 文件”。

2. 打开下载的 RDP 文件，然后在出现提示时单击“连接”。 

3. 在“Windows 安全性”窗口中，依次选择“更多选择”、“使用其他帐户”。 键入 *vmname*\*username* 作为用户名，输入为虚拟机创建的密码，然后单击“确定”。

4. 你可能会在登录过程中收到证书警告。 单击“是”或“继续”继续进行连接。

## <a name="install-web-server"></a>安装 Web 服务器

若要查看运行中的 VM，请安装 IIS Web 服务器。 在 VM 中打开 PowerShell 提示符并运行以下命令：

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

完成后，关闭到 VM 的 RDP 连接。

## <a name="open-port-80-for-web-traffic"></a>为 Web 流量打开端口 80

网络安全组 (NSG) 保护入站和出站流量的安全。 从 Azure 门户创建 VM 后，会在进行 RDP 连接的端口 3389 上创建入站规则。 由于此 VM 托管着 Web 服务器，因此需要为端口 80 创建 NSG 规则。

1. 在 VM 概述页中，选择“网络”。
2. 此时将显示现有的入站和出站规则的列表。 选择“添加入站端口规则”。
3. 选择顶部的“基本”选项，然后从可用服务列表中选择“HTTP”。 将为你提供端口 80、优先级和名称。
4. 若要创建规则，请选择“添加”。

## <a name="view-the-iis-welcome-page"></a>查看 IIS 欢迎页

IIS 已安装，并且现在已从 Internet 打开 VM 上的端口 80 - 可以使用所选的 Web 浏览器查看默认的 IIS 欢迎页。 使用上一步中获取的 VM 的公用 IP 地址。 以下示例展示了默认 IIS 网站：

![IIS 默认站点](./media/quick-create-powershell/default-iis-website.png)

## <a name="clean-up-resources"></a>清理资源

不再需要资源组、虚拟机和所有相关的资源时，可将其删除。 为此，请选择虚拟机的资源组，选择“删除”，然后确认要删除的资源组的名称。

## <a name="next-steps"></a>后续步骤

在本快速入门中，你部署了简单的虚拟机，打开了 Web 流量的网络端口，并安装了一个基本 Web 服务器。 若要详细了解 Azure 虚拟机，请继续学习 Windows VM 的教程。

> [!div class="nextstepaction"]
> [Azure Windows 虚拟机教程](./tutorial-manage-vm.md)
