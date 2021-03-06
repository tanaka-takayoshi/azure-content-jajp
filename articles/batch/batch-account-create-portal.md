<properties
	pageTitle="Azure Batch アカウントの作成 | Microsoft Azure"
	description="Azure ポータルで、クラウド内で大規模な並列ワークロードを実行する Azure Batch アカウントを作成する方法について説明します"
	services="batch"
	documentationCenter=""
	authors="mmacy"
	manager="timlt"
	editor=""/>

<tags
	ms.service="batch"
	ms.workload="big-compute"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="05/26/2016"
	ms.author="marsma"/>

# Azure ポータルで Azure Batch アカウントを作成して管理する

> [AZURE.SELECTOR]
- [Azure ポータル](batch-account-create-portal.md)
- [Batch Management .NET](batch-management-dotnet.md)

[Azure ポータル][azure_portal]には、Azure Batch アカウントの作成と管理に必要なツールが用意されています。これらのツールを使用して、大規模な並列ワークロードを処理することができます。この記事では、ポータルを使用して Batch アカウントを作成する手順について説明し、さらに Batch アカウントの重要な設定とプロパティについて取り上げます。たとえば、Batch を使用して開発するアプリケーションやサービスには、Batch サービスと通信するためにアカウントの URL とアクセス キーが必要です。どちらも Azure ポータル内にあります。

>[AZURE.NOTE] Azure ポータルでは、現時点でアカウントの作成、Batch アカウントの設定とプロパティの管理、プールとジョブの作成および監視など、Batch サービスで利用できる機能の一部をサポートしています。開発時は、Batch API を介して Batch の全機能を利用できます。

## Batch アカウントを作成する

1. [Azure ポータル][azure_portal]にサインインします。

2. **[新規]**、**[Virtual Machines]**、**[Batch サービス]** の順にクリックします。

	![Marketplace での Batch][marketplace_portal]

3. **[Batch サービス]** ブレードの内容を確認し、**[作成]** をクリックします。Batch では、Resource Manager デプロイ モデルのみが使用される点に注意してください。

	![Batch Service create blade in Azure portal][3]

4. **[新しい Batch アカウント]** ブレードが表示されます。ブレードの各要素の説明については、下記の *a* ～ *e* の項目を参照してください。

    ![Batch アカウントを作成する][account_portal]

	a.**アカウント名** -- Batch アカウントの一意の名前。この名前は、アカウントが作成されている Azure リージョン内で一意である必要があります (下の「*場所*」を参照してください)。アカウント名に含めることができるのは、英小文字と数字のみで、文字数は 3 ～ 24 文字にする必要があります。

	b.**サブスクリプション** -- Batch アカウントを作成するサブスクリプション。サブスクリプションが 1 つのみの場合は、既定でそのサブスクリプションが選択されます。

	c.**リソース グループ** -- 新しい Batch アカウント用のリソース グループ。必要に応じて、新しく作成することもできます。

	d.**場所** -- Batch アカウントを作成する Azure リージョン。サブスクリプションとリソース グループでサポートされているリージョンのみがオプションとして表示されます。

    e.**ストレージ アカウント** (省略可能) -- 新しい Batch アカウントに関連付ける (リンクする) **汎用**のストレージ アカウント。Batch の[アプリケーション パッケージ](batch-application-packages.md)機能では、リンクされたストレージ アカウントを使用してアプリケーション パッケージの保存と取得を行います。この機能の詳細については、「[Azure Batch アプリケーション パッケージを使用したアプリケーションのデプロイ](batch-application-packages.md)」を参照してください。

     > [AZURE.TIP] リンクされたストレージ アカウント内のキーを再生成する際は、特に注意が必要な点があります。詳細については、下の「[Batch アカウントに関する注意点](#considerations-for-batch-accounts)」を参照してください。

5. **[作成]** をクリックしてアカウントを作成します。

  アカウントを**デプロイ中**であることがポータルに表示され、完了すると [Batch アカウント] ブレードが表示されます。

## Batch アカウントのプロパティを表示する

[Batch アカウント] ブレードにはアカウントのプロパティがいくつか表示されているほか、アクセス キー、クォータ、ユーザー、ストレージ アカウントの関連付けなどの詳細設定も利用できます。

* **Batch アカウント URL** -- [Batch REST][api_rest] API や [Batch .NET][api_net] クライアント ライブラリなどの API を使用している場合は、この URL で Batch アカウントにアクセスできます。URL の形式は次のとおりです。

    `https://<account_name>.<region>.batch.azure.com`

* **アクセス キー** -- Batch アカウントのアクセス キーの表示や管理を行うには、鍵アイコンをクリックして **[キーの管理]** ブレードを開くか、**[すべての設定]**、**[キー]** の順にクリックします。アクセス キーは、[Batch REST][api_rest] や [Batch .NET][api_net] クライアント ライブラリなどの Batch サービス API と通信する際に必要になります。

    ![Batch アカウント キー][account_keys]

* **すべての設定** -- Batch アカウントのすべての設定を管理したり、そのプロパティを表示したりするには、**[すべての設定]** をクリックして **[設定]** ブレードを開きます。このブレードから、アカウント クォータの表示、Batch アカウントにリンクする Azure ストレージ アカウントの選択、ユーザーの管理など、アカウントのすべての設定とプロパティを利用できます。

    ![Batch account settings and properties blades][5]

## Batch アカウントに関する注意点

* Batch アカウントの作成と管理には、[Batch PowerShell コマンドレット](batch-powershell-cmdlets-get-started.md)や [Batch Management .NET ライブラリ](batch-management-dotnet.md)も使用できます。

* Batch アカウント自体には課金されません。課金は、Batch ソリューションが使用する Azure コンピューティング リソースと、ワークロードの実行時に他のサービスが使用するリソースが対象になります。たとえば、プール内のコンピューティング ノードは課金の対象です。[アプリケーション パッケージ](batch-application-packages.md)機能を使用している場合は、アプリケーション パッケージのバージョンを格納するために使用している Azure Storage リソースに課金されます。詳細については、「[Batch の価格][batch_pricing]」を参照してください。

* 1 つの Batch アカウントで複数の Batch ワークロードを実行したり、異なる Azure リージョンの複数の Batch アカウントにワークロードを分散したりすることができます。

* いくつかの大規模な Batch ワークロードを実行する場合は、Azure サブスクリプションと各 Batch アカウントに適用する [Batch サービス クォータと制限](batch-quota-limit.md)について気を付けてください。Batch アカウントの現在のクォータは、アカウント プロパティのポータルに表示されます。

* Batch アカウントにストレージ アカウントを関連付けている (リンクしている) 場合は、ストレージ アカウントのアクセス キーを再生成する際に注意が必要です。ストレージ アカウント キーは 1 つだけ再生成します。リンクされたストレージ アカウントのブレードで **[キーの同期]** をクリックし、キーがプール内のコンピューティング ノードに反映されるまで 5 分待ちます。その後、必要に応じて他のキーの再生成と同期を行います。両方のキーを同時に再生成すると、コンピューティング ノードはどちらのキーも同期できず、ストレージ アカウントにアクセスできなくなります。

  ![Regenerating storage account keys][4]

> [AZURE.IMPORTANT] 「[Azure ストレージ アカウントについて](../storage/storage-create-storage-account.md)」の手順 5.「[ストレージ アカウントの作成](../storage/storage-create-storage-account.md#create-a-storage-account)」で説明されているように、Batch では、現時点で**汎用**のストレージ アカウントの種類*のみ*がサポートされています。Azure Storage アカウントを Batch アカウントにリンクする場合は、**汎用**のストレージ アカウント*のみ*をリンクしてください。

## 次のステップ

* Batch サービスの概念と機能の詳細については、[Azure Batch 機能の概要](batch-api-basics.md)に関するページを参照してください。この記事では、プール、コンピューティング ノード、ジョブ、タスクなど、主要な Batch リソースについて説明すると共に、大規模なコンピューティング ワークロードを実行する機能の概要について説明しています。

* [Batch .NET クライアント ライブラリ](batch-dotnet-get-started.md)を使用した Batch 対応アプリケーションの開発に関する基本事項を確認してください。この[入門記事](batch-dotnet-get-started.md)では、Batch サービスを使用して複数のコンピューティング ノードでワークロードを実行する実用アプリケーションの開発手順を説明しています。また、Azure Storage を使用してワークロード ファイルのステージングと取得を行う方法についても取り上げています。

[api_net]: https://msdn.microsoft.com/library/azure/mt348682.aspx
[api_rest]: https://msdn.microsoft.com/library/azure/Dn820158.aspx

[azure_portal]: https://portal.azure.com
[batch_pricing]: https://azure.microsoft.com/pricing/details/batch/

[3]: ./media/batch-account-create-portal/batch_acct_03.png "Batch Service create blade in Azure portal"
[4]: ./media/batch-account-create-portal/batch_acct_04.png "Regenerating storage account keys"
[5]: ./media/batch-account-create-portal/batch_acct_05.png "Batch account settings and properties blades"
[marketplace_portal]: ./media/batch-account-create-portal/marketplace_batch.PNG
[account_portal]: ./media/batch-account-create-portal/batch_acct_portal.png
[account_keys]: ./media/batch-account-create-portal/account_keys.PNG

<!---HONumber=AcomDC_0601_2016-->