---
title: インクルード ファイル
description: インクルード ファイル
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 02/02/2018
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: 026540290710d039dbc06c394ab538ebe2d7c12f
ms.sourcegitcommit: edacc2024b78d9c7450aaf7c50095807acf25fb6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/13/2018
ms.locfileid: "53344671"
---
"_ローカル ターミナル ウィンドウ_" で、ローカル Git リポジトリに Azure リモートを追加します。 _&lt;deploymentLocalGitUrl-from-create-step>_ を、[Web アプリの作成](#create-a-web-app)に関するセクションで保存した Git リモートの URL に置き換えます。

```bash
git remote add azure <deploymentLocalGitUrl-from-create-step>
```

アプリをデプロイするために、次のコマンドで Azure リモートにプッシュします。 Git Credential Manager によって資格情報の入力を求めるメッセージが表示されたら、Azure portal へのサインインに使用する資格情報ではなく、[デプロイ ユーザーの構成](#configure-a-deployment-user)に関するセクションで作成した資格情報を入力してください。

```bash
git push azure master
```

このコマンドの実行には、数分かかる場合があります。 実行中、次の例のような情報が表示されます。
