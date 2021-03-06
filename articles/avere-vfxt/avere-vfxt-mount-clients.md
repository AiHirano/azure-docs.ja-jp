---
title: Avere vFXT をマウントする - Azure
description: Avere vFXT for Azure でクライアントをマウントする方法
author: ekpgh
ms.service: avere-vfxt
ms.topic: conceptual
ms.date: 10/31/2018
ms.author: v-erkell
ms.openlocfilehash: 5d55879a5a8487636e2252abd359accf07e60ce6
ms.sourcegitcommit: 6135cd9a0dae9755c5ec33b8201ba3e0d5f7b5a1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/31/2018
ms.locfileid: "50669479"
---
# <a name="mount-the-avere-vfxt-cluster"></a>Avere vFXT クラスターをマウントする  

クライアント マシンを vFXT クラスターに接続するには以下の手順のようにします。

1. クラスター ノード間にクライアント トラフィックの負荷を分散させる方法を決定します。 詳細については、後の「[クライアントの負荷を分散させる](#balance-client-load)」をご覧ください。 
1. マウントする [IP アドレスとジャンクション パス](#identify-ip-addresses-and-paths-to-mount)を識別します。
1. 適切な引数を指定して [mount コマンド](#mount-command-arguments)を発行します。

## <a name="balance-client-load"></a>クライアントの負荷を分散させる

クライアントの要求をクラスター内のすべてのノード間に分散させるには、クライアント側の IP アドレスの全範囲にクライアントをマウントする必要があります。 このタスクを自動化するには、いくつかの簡単な方法があります。

> [!TIP] 
> 他の負荷分散方法は、さらに大規模なシステムまたは複雑なシステムに適しています (ヘルプが必要な場合は、[サポート チケットを開いて](avere-vfxt-open-ticket.md#open-a-support-ticket-for-your-avere-vfxt)ください)。
> 
> サーバー側の自動負荷分散に DNS サーバーを使用したい場合は、Azure 内で独自の DNS サーバーを設定して管理する必要があります。 その場合は、「[Avere cluster DNS configuration](avere-vfxt-configure-dns.md)」(Avere クラスターの DNS 構成) に従って、vFXT クラスター用にラウンドロビン DNS を構成することができます。

### <a name="sample-balanced-client-mounting-script"></a>負荷分散されたクライアントのマウント スクリプトの例

このコード例では、vFXT クラスターで使用可能なすべての IP アドレスにクライアントを分散させるためのランダム化要素として、クライアントの IP アドレスを使用しています。

```bash
function mount_round_robin() {
    # to ensure the nodes are spread out somewhat evenly the default 
    # mount point is based on this node's IP octet4 % vFXT node count.
    declare -a AVEREVFXT_NODES="($(echo ${NFS_IP_CSV} | sed "s/,/ /g"))"
    OCTET4=$((`hostname -i | sed -e 's/^.*\.\([0-9]*\)/\1/'`))
    DEFAULT_MOUNT_INDEX=$((${OCTET4} % ${#AVEREVFXT_NODES[@]}))
    ROUND_ROBIN_IP=${AVEREVFXT_NODES[${DEFAULT_MOUNT_INDEX}]}

    DEFAULT_MOUNT_POINT="${BASE_DIR}/default"

    # no need to write again if it is already there
    if ! grep --quiet "${DEFAULT_MOUNT_POINT}" /etc/fstab; then
        echo "${ROUND_ROBIN_IP}:${NFS_PATH}    ${DEFAULT_MOUNT_POINT}    nfs hard,nointr,proto=tcp,mountproto=tcp,retry=30 0 0" >> /etc/fstab
        mkdir -p "${DEFAULT_MOUNT_POINT}"
        chown nfsnobody:nfsnobody "${DEFAULT_MOUNT_POINT}"
    fi
    if ! grep -qs "${DEFAULT_MOUNT_POINT} " /proc/mounts; then
        retrycmd_if_failure 12 20 mount "${DEFAULT_MOUNT_POINT}" || exit 1
    fi   
} 
```

上記の関数は、[Avere vFXT の例](https://github.com/Azure/Avere#tutorials)のサイトから入手できる Batch の例の一部です。

## <a name="create-the-mount-command"></a>mount コマンドを作成する 

> [!NOTE]
> Avere vFXT クラスターを作成するときに新しい BLOB コンテナーを作成しなかった場合は、クライアントをマウントする前に、「[Configure storage](avere-vfxt-add-storage.md)」(ストレージの構成) の手順に従います。

クライアントで ``mount`` コマンドを実行すると、vFXT クラスター上の仮想サーバー (vserver) が、ローカル ファイル システムのパスにマップされます。 形式は ``mount <vFXT path> <local path> {options}`` です

mount コマンドには 3 つの要素があります。 

* vFXT パス - ([後](#junction-and-ip-address)で説明する、IP アドレスと名前空間ジャンクション パスの組み合わせ)
* ローカル パス - クライアント上のパス 
* mount コマンドのオプション - (「[Mount command arguments](#mount-command-arguments)」(mount コマンドの引数) を参照)

### <a name="junction-and-ip"></a>ジャンクションと IP

vserver のパスは、その "*IP アドレス*" と、"*名前空間ジャンクション*" へのパスの組み合わせです。 名前空間ジャンクションは、ストレージ システムが追加されるときに定義された仮想パスです。

クラスターが BLOB ストレージで作成された場合、名前空間パスは `/msazure` です

例: ``mount 10.0.0.12:/msazure /mnt/vfxt``

クラスターを作成した後でストレージを追加した場合は、名前空間ジャンクション パスは、ジャンクションを作成するときに **[Namespace path]\(名前空間のパス\)** で設定した値に対応します。 たとえば、名前空間のパスとして ``/avere/files`` を使用した場合、クライアントは <*IP アドレス*>:/avere/files をローカルなマウント ポイントにマウントします。

![[Namespace path]\(名前空間のパス\) フィールドに "/avere/files" が設定された [Add new junction]\(新しいジャンクションの追加\) ダイアログ](media/avere-vfxt-create-junction-example.png)


IP アドレスは、vserver に対して定義されたクライアント側の IP アドレスの 1 つです。 クライアント側の IP アドレスの範囲は、Avere Control Panel の 2 つの場所で確認できます。

* **[VServers]** テーブル ([Dashboard]\(ダッシュボード\) タブ) - 

  ![Avere Control Panel の [Dashboard]\(ダッシュボード\) タブで、グラフの下のデータ テーブルの [VServer] タブが選択され、IP アドレス セクションが囲まれている](media/avere-vfxt-ip-addresses-dashboard.png)

* **[Client Facing Network]\(クライアント側ネットワーク\)** 設定ページ - 

  ![[Settings]\(設定\) > [VServer] > [Client Facing Network]\(クライアント側ネットワーク\) 構成ページで、特定の vserver のテーブルの [Address Range]\(アドレス範囲\) セクションが囲まれている](media/avere-vfxt-ip-addresses-settings.png)

各クライアントをマウントするときは、パスに加えて、以下で説明する [mount コマンド引数](#mount-command-arguments)を指定します。

### <a name="mount-command-arguments"></a>mount コマンドの引数

クライアントのマウントがシームレスに行われるように、mount コマンドで以下の設定と引数を渡します。 

``mount -o hard,nointr,proto=tcp,mountproto=tcp,retry=30 ${VSERVER_IP_ADDRESS}:/${NAMESPACE_PATH} ${LOCAL_FILESYSTEM_MOUNT_POINT}``


| 必須の設定 | |
--- | --- 
``hard`` | vFXT クラスターに対するソフト マウントは、アプリケーション エラーおよび可能性のあるデータ損失と関連付けられます。 
``proto=netid`` | このオプションは、NFS ネットワーク エラーの適切な処理をサポートします。
``mountproto=netid`` | このオプションは、マウント操作に対するネットワーク エラーの適切な処理をサポートします。
``retry=n`` | 一時的なマウントの障害を回避するため、``retry=30`` を設定します。 (フォアグラウンド マウントの場合は、別の値が推奨されます。)

| 推奨される設定  | |
--- | --- 
``nointr``            | このオプションをサポートする従来のカーネル (2008 年 4 月より前) を使用するクライアントの場合は、オプション "nointr" が推奨されます。 オプション "intr" が既定値であることに注意してください。


## <a name="next-steps"></a>次の手順

クライアントをマウントした後は、クライアントを使用してバックエンド データ ストレージ (コア ファイラー) を設定できます。 他のセットアップ タスクの詳細については、以下のドキュメントをご覧ください。

* [クラスターのコア ファイラーにデータを移動する](avere-vfxt-data-ingest.md) - 複数のクライアントとスレッドを使用して、データを効率的にアップロードする方法
* [クラスターのチューニングをカスタマイズする](avere-vfxt-tuning.md) - ワークロードに合わせてクラスターの設定を調整します
* [クラスターを管理する](avere-vfxt-manage-cluster.md) - クラスターの開始または停止およびノードの管理の方法