---
title: 动态组和 Azure Active Directory B2B 协作 | Microsoft 文档
description: 说明如何将 Azure AD 动态组与 Azure Active Directory B2B 协作一起使用
services: active-directory
documentationcenter: ''
author: twooley
manager: mtillman
editor: ''
tags: ''
ms.assetid: ''
ms.service: active-directory
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: identity
ms.date: 12/14/2017
ms.author: twooley
ms.reviewer: sasubram
ms.openlocfilehash: d96fefb859cba5db65382801fb1ac143df12b647
ms.sourcegitcommit: 6e43006c88d5e1b9461e65a73b8888340077e8a2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2018
---
# <a name="dynamic-groups-and-azure-active-directory-b2b-collaboration"></a>动态组和 Azure Active Directory B2B 协作

## <a name="what-are-dynamic-groups"></a>什么是动态组？
Azure Active Directory (Azure AD) 的安全组成员身份动态配置在 [Azure 门户](https://portal.azure.com)中提供。 管理员可以设置规则以填充在 Azure AD 中基于用户属性（如 userType、部门或国家/地区）创建的组。 可基于成员属性自动在安全组中添加或删除成员。 这些组可以为成员提供对应用程序或云资源（SharePoint 站点、文档）的访问权限并分配许可证。 在 [Azure Active Directory 中的专用组](active-directory-accessmanagement-dedicated-groups.md)中阅读有关动态组的详细信息。

创建和使用动态组需要相应的 [Azure AD Premium P1 或 P2 授权](https://azure.microsoft.com/pricing/details/active-directory/)。 有关详细信息，请参阅[在 Azure Active Directory 中为动态组成员身份创建基于属性的规则](active-directory-groups-dynamic-membership-azure-portal.md)一文。

## <a name="what-are-the-built-in-dynamic-groups"></a>什么是内置动态组？
“所有用户”动态组使租户管理员只需一次单击即可创建包含租户中所有用户的组。 默认情况下，“所有用户”组包含目录中的所有用户，其中包括成员和来宾。
在新的 Azure Active Directory 管理门户中，可以在“组设置”视图中选择启用“所有用户”组。

![显示将“启用‘所有用户’组”设置为“是”](media/active-directory-b2b-dynamic-groups/enable-all-users-group.png)

## <a name="hardening-the-all-users-dynamic-group"></a>强化“所有用户”动态组
默认情况下，“所有用户”组也包含 B2B 协作（来宾）用户。 可以使用规则删除来宾用户，进一步保护“所有用户”组。 下图显示修改后不包括来宾的“所有用户”组。

![显示用户类型不等于“来宾”的规则](media/active-directory-b2b-dynamic-groups/exclude-guest-users.png)

你可能还会发现创建仅包含来宾用户的新动态组很有用，这样就可以向来宾用户应用策略（例如 Azure AD 条件访问策略）。
此类组的外观可能如下所示：

![显示用户类型等于“来宾”的规则](media/active-directory-b2b-dynamic-groups/only-guest-users.png)

## <a name="next-steps"></a>后续步骤

在 Azure AD B2B 协作网站上浏览我们的其他文章：

* [什么是 Azure AD B2B 协作？](active-directory-b2b-what-is-azure-ad-b2b.md)
* [B2B 协作用户属性](active-directory-b2b-user-properties.md)
* [将 B2B 协作用户添加到角色](active-directory-b2b-add-guest-to-role.md)
* [委托 B2B 协作邀请](active-directory-b2b-delegate-invitations.md)
* [B2B 协作代码和 PowerShell 示例](active-directory-b2b-code-samples.md)
* [为 B2B 协作配置 SaaS 应用](active-directory-b2b-configure-saas-apps.md)

