---
title: Azure Active Directory B2B コラボレーションの招待の委任 | Microsoft Docs
description: Azure Active Directory B2B コラボレーション ユーザーのプロパティは構成できます
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: conceptual
ms.date: 05/23/2017
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: sasubram
ms.openlocfilehash: fd00fb8da3cf36518f9e28e827e59fd7ff45d687
ms.sourcegitcommit: 1f9e1c563245f2a6dcc40ff398d20510dd88fd92
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2018
ms.locfileid: "51622212"
---
# <a name="delegate-invitations-for-azure-active-directory-b2b-collaboration"></a>Azure Active Directory B2B コラボレーションの招待の委任

Azure Active Directory (Azure AD) 企業間 (B2B) コラボレーションでは、グローバル管理者でなくても招待を送信できます。 それには、ポリシーを使用して、招待の送信が許可されているロールを持つユーザーに招待を委任します。 ゲスト ユーザーの招待を委任するための重要な新しい方法は、ゲストの招待元ロールを通じた方法です。

## <a name="guest-inviter-role"></a>ゲストの招待元ロール
ゲストの招待元ロールにユーザーを割り当てて、招待を送信することができます。 全体管理者ロールのメンバーである必要はありません。 既定では、全体管理者が通常のユーザーの招待を無効にしていない限り、通常のユーザーも招待 API を呼び出すことができます。 ユーザーは、Azure ポータルまたは PowerShell を使用して API を呼び出すこともできます。

次の例は、PowerShell を使って、ゲストの招待元ロールにユーザーを追加する方法を示しています。

```
Add-MsolRoleMember -RoleObjectId 95e79109-95c0-4d8e-aee3-d01accf2d47b -RoleMemberEmailAddress <RoleMemberEmailAddress>
```

## <a name="control-who-can-invite"></a>招待できるユーザーの制御

[Azure Active Directory] > [ユーザー設定] > [外部コラボレーションの設定を管理します] の順に移動します。

![外部ユーザー](https://user-images.githubusercontent.com/13383753/45905128-2c47f680-bda4-11e8-955d-6219c67935e0.PNG)

Azure AD B2B コラボレーションでは、テナント管理者が次の招待ポリシーを設定できます。

- 招待を無効にする
- 管理者と、ゲストの招待元ロールのユーザーのみが招待できる
- 管理者、ゲストの招待元ロール、およびメンバーが招待できる
- ゲストを含むすべてのユーザーが招待できる

既定では、テナントは #4  (ゲストを含むすべてのユーザーが B2B ユーザーを招待できる) に設定されます。

## <a name="next-steps"></a>次の手順

Azure AD B2B コラボレーションに関する以下の記事を参照してください。

- [Azure AD B2B コラボレーションとは](what-is-b2b.md)
- [招待を使用せずに B2B コラボレーション ゲスト ユーザーを追加する](add-user-without-invite.md)
- [B2B コラボレーション ユーザーのロールへの追加](add-guest-to-role.md)


