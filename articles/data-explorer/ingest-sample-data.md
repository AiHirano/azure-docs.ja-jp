---
title: Azure のデータ エクスプローラーにサンプル データを取り込む
description: Azure のデータ エクスプローラーに気象関連のサンプル データ (負荷) を取り込む方法を説明します。
services: data-explorer
author: orspod
ms.author: v-orspod
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 7fdd32f9263b4d1694a0516a98b681ba8744ab6b
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/17/2018
ms.locfileid: "49394580"
---
# <a name="ingest-sample-data-into-azure-data-explorer"></a>Azure のデータ エクスプローラーにサンプル データを取り込む

この記事では、Azure のデータ エクスプローラー データベースにサンプル データ (負荷) を取り込む方法を示します。 [データを取り込む方法はいくつかあります](ingest-data-overview.md)｡ 今回は、テスト目的に適した基本的なアプローチに注目します。

> [!NOTE]
> [クイック スタート: Azure のデータ エクスプローラーの Python ライブラリを使用してデータを取り込む](python-ingest-data.md) を終えている場合､サンプル データはすでにあります｡

## <a name="prerequisites"></a>前提条件

[テスト用のクラスターとデータベース](create-cluster-database-portal.md)

## <a name="ingest-data"></a>データの取り込み

**StormEvents**サンプル データ セットには､ [National Centers for Environmental Information から入手した気象関連データが含まれています](https://www.ncdc.noaa.gov/stormevents/)｡

1. [https://dataexplorer.azure.com](https://dataexplorer.azure.com) にサインインします。

1. アプリケーションの左上にある **[Add cluster]\(クラスターの追加\)** を選択します。

1. **[Add cluster]** ダイアログ ボックスで `https://<ClusterName>.<Region>.kusto.windows.net/` の形式でラスターの URL を入力して､**[追加]** を選択します。

1. 次のコマンドに貼り付けて、**[実行]** を選択します。

    ```Kusto
    .create table StormEvents (StartTime: datetime, EndTime: datetime, EpisodeId: int, EventId: int, State: string, EventType: string, InjuriesDirect: int, InjuriesIndirect: int, DeathsDirect: int, DeathsIndirect: int, DamageProperty: int, DamageCrops: int, Source: string, BeginLocation: string, EndLocation: string, BeginLat: real, BeginLon: real, EndLat: real, EndLon: real, EpisodeNarrative: string, EventNarrative: string, StormSummary: dynamic)

    .ingest into table StormEvents h'https://kustosamplefiles.blob.core.windows.net/samplefiles/StormEvents.csv?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D' with (ignoreFirstRecord=true)
    ```

1. 取り込みが完了したら、次のクエリに貼り付けて､ウィンドウでクエリを選択し､**[実行]** を選択します。

    ```Kusto
    StormEvents
    | sort by StartTime desc
    | take 10
    ```
    取り込まれたサンプル データから次のクエリ結果が返されます。

    ![Query results](media/ingest-sample-data/query-results.png)

## <a name="next-steps"></a>次の手順

[クエリを作成する](write-queries.md)

[Azure のデータ エクスプローラーへのデータの取り込み](ingest-data-overview.md)