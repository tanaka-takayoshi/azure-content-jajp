<properties
	pageTitle="Azure Data Factory を使用してデータをオンプレミスの SQL Server から SQL Azure に移動する | Azure"
	description="オンプレミスとクラウド内のデータベース間で毎日同時にデータを移動する 2 つのデータ移行アクティビティを構成する ADF パイプラインを設定します。"
	services="machine-learning"
	documentationCenter=""
	authors="bradsev"
	manager="paulettm"
	editor="cgronlun" />

<tags
	ms.service="machine-learning"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="05/10/2016"
	ms.author="fashah;bradsev" />


# Azure Data Factory を使用してオンプレミスの SQL Server から SQL Azure にデータを移動する

このトピックでは、Azure Data Factory (ADF) を使用して、オンプレミスの SQL Server データベースから Azure BLOB ストレージを経由して SQL Azure データベースにデータを移動する方法を説明します。

次の**メニュー**は、Cortana Analytics Process (CAP) でデータを保存および処理できる他のターゲット環境にデータを取り込む方法について説明するトピックにリンクしています。

[AZURE.INCLUDE [cap-ingest-data-selector](../../includes/cap-ingest-data-selector.md)]


## <a name="intro"></a>概要: ADF の説明とデータの移行に ADF を使用するべきタイミング

Azure Data Factory は、データの移動や変換を調整し自動化する、完全に管理されたクラウドベースのデータ統合サービスです。ADF モデルにおける主要な概念は、パイプラインです。パイプラインはアクティビティの論理グループで、それぞれがデータセットに含まれているデータに対して実行するアクションを定義します。リンクされたサービスは、Data Factory がデータ リソースに接続するために必要な情報を定義するために使用されます。

ADF を使用すると、既存のデータ処理サービスを、可用性が高く、クラウドで管理されるデータ パイプラインに組み込むことができます。これらのデータ パイプラインは、データの取り込み、準備、変換、分析、および発行のためのスケジュールが設定できます。ADF が複雑なデータと処理の依存関係をすべて管理して調整します。増加するオンプレミスのデータ ソースとクラウドのデータ ソースを接続するソリューションをクラウド内で迅速に構築してデプロイすることができます。

オンプレミスとクラウドの両方のリソースにアクセスするハイブリッド シナリオで、継続的にデータを移行する必要がある場合、および移行の過程で、データを処理する場合や、データに変更を加えたりビジネス ロジックを付加したりする必要がある場合には、ADF の使用を検討してください。ADF では、定期的にデータの移動を管理するシンプルな JSON スクリプトを使用して、ジョブのスケジュールと監視ができます。ADF には他にも、複雑な操作のサポートなどの機能があります。詳細については、[Azure Data Factory (ADF)](https://azure.microsoft.com/services/data-factory/) にあるドキュメントを参照してください。


## <a name="scenario"></a>シナリオ

オンプレミスの SQL Database とクラウド内の Azure SQL Database 間で毎日同時にデータを移動する 2 つのデータ移行アクティビティを構成する ADF パイプラインを設定します。2 つのアクティビティは次のとおりです。

* オンプレミスの SQL Server データベースから Azure BLOB ストレージ アカウントにデータをコピーする
* Azure BLOB ストレージ アカウントから Azure SQL Database にデータをコピーする

**参照**: 以下に示した手順は、ADF チームが提供しているより詳細なチュートリアル「[Data Management Gateway を使用してオンプレミスのソースとクラウドの間でデータを移動する](../data-factory/data-factory-move-data-between-onprem-and-cloud.md)」からの抜粋です。また、トピックの関連セクションへの参照が適宜提供されています。


## <a name="prereqs"></a>前提条件
このチュートリアルでは、以下があることを前提としています。

* **Azure サブスクリプション**。サブスクリプションがない場合は、[無料試用版](https://azure.microsoft.com/pricing/free-trial/)にサインアップできます。
* **Azure ストレージ アカウント**。このチュートリアルのデータを格納するには、Azure ストレージ アカウントを使用します。Azure ストレージ アカウントがない場合は、「[ストレージ アカウントの作成](storage-create-storage-account.md#create-a-storage-account)」を参照してください。ストレージ アカウントを作成した後は、ストレージにアクセスするために使用するアカウント キーを取得する必要があります。「[ストレージ アクセス キーの表示、コピーおよび再生成](storage-create-storage-account.md#view-copy-and-regenerate-storage-access-keys)」をご覧ください。
* **Azure SQL Database** へのアクセス権。Azure SQL Database をセットアップする必要がある場合、Azure SQL Database の新しいインスタンスをプロビジョニングする方法については、「[最初の Azure SQL データベースを作成する](../sql-database/sql-database-get-started.md)」をご覧ください。
* **Azure PowerShell** がローカルにインストールされ構成されていること。手順については、「[Azure PowerShell のインストールおよび構成方法](../powershell-install-configure.md)」を参照してください。

> [AZURE.NOTE] この手順では、[Azure ポータル](https://portal.azure.com/)を使用します。


##<a name="upload-data"></a>オンプレミスの SQL Server にデータをアップロードする

[NYC タクシー データセット](http://chriswhong.com/open-data/foil_nyc_taxi/)を使用して、移行プロセスを説明します。NYC タクシー データセットは、記事に記載されているように、Azure BLOB ストレージの [NYC タクシー データ](http://www.andresmh.com/nyctaxitrips/)から入手できます。データには、乗車の詳細を含む trip\_data.csv ファイルと、乗車ごとの料金の詳細を含む trip\_far.csv ファイルの 2 つのファイルがあります。これらのファイルのサンプルと説明は、「[NYC タクシー乗車データセットの説明](machine-learning-data-science-process-sql-walkthrough.md#dataset)」にあります。


ここに示されている手順は、自身のデータに適用することも、NYC タクシー データセットを使用してこの手順に従って行うこともできます。NYC タクシー データセットを自身のオンプレミスの SQL Server データベースにアップロードするには、「[SQL Server データベースにデータを一括インポートする](machine-learning-data-science-process-sql-walkthrough.md#dbload)」に記載されている手順に従います。これらは Azure Virtual Machine 上の SQL Server にアップロードする手順ですが、オンプレミスの SQL Server へのアップロード手順も同じです。


##<a name="create-adf"></a>Azure Data Factory を作成する

[Azure ポータル](https://portal.azure.com/)に新しい Azure Data Factory とリソース グループを作成するための手順については、「[Azure Data Factory を作成する](../data-factory/data-factory-build-your-first-pipeline-using-editor.md#step-1-creating-the-data-factory)」を参照してください。新しい ADF インスタンスに *adfdsp* という名前を付け、作成されたリソース グループに *adfdsprg* という名前を付けます。


## Data Management Gateway をインストールして構成する

Azure Data Factory 内でパイプラインがオンプレミスの SQL Server を使用できるようにするには、リンクされたサービスとしてその SQL Server を ADF に追加する必要があります。オンプレミスの SQL Server にリンクされたサービスを作成するには、まず Microsoft Data Management Gateway をダウンロードしてオンプレミスのコンピューターにインストールし、オンプレミスのデータ ソースがゲートウェイを使用できるようにリンクされたサービスを構成する必要があります。Data Management Gateway では、データのシリアル化と逆シリアル化、データがホストされているコンピューター上のデータの同期が行われます。

Data Management Gateway のセットアップ手順と詳細については、「[Data Management Gateway を使用してオンプレミスのソースとクラウドの間でデータを移動する](../data-factory/data-factory-move-data-between-onprem-and-cloud.md)」をご覧ください。


## <a name="adflinkedservices"></a>データ リソースに接続するためにリンクされたサービスを作成する

リンクされたサービスは、Azure Data Factory がデータ リソースに接続するために必要な情報を定義します。リンクされたサービスを作成するための手順は、「[リンクされたサービスを作成する](../data-factory/data-factory-move-data-between-onprem-and-cloud.md#step-2-create-linked-services)」を参照してください。

このシナリオには、リンクされたサービスを必要とする 3 つのリソースがあります。

1. [オンプレミスの SQL Server 用のリンクされたサービス](#adf-linked-service-onprem-sql)
2. [Azure BLOB ストレージ用のリンクされたサービス](#adf-linked-service-blob-store)
3. [Azure SQL データベース用のリンクされたサービス](#adf-linked-service-azure-sql)


###<a name="adf-linked-service-onprem-sql"></a>オンプレミスの SQL Server データベース用のリンクされたサービス
オンプレミスの SQL Server用にリンクされたサービスを作成するには、Azure クラシック ポータルで ADF ランディング ページ内の **[データ ストア]** をクリックし、*SQL* を選択してオンプレミスの SQL Server の*ユーザー名*と*パスワード*に対して資格情報を入力します。サーバー名は**完全修飾サーバー名、バックスラッシュ、インスタンス名 (servername\\instancename)** で入力する必要があります。リンクされたサービスに *adfonpremsql* という名前を付けます。

###<a name="adf-linked-service-blob-store"></a>BLOB 用のリンクされたサービス
Azure BLOB ストレージ アカウント用にリンクされたサービスを作成するには、Azure クラシック ポータルで ADF ランディング ページ内の **[データ ストア]** をクリックし、*ADF ストレージアカウント*を選択して Azure BLOB ストレージ アカウント キーとコンテナー名を入力します。リンク サービスに *adfds* という名前を付けます。

###<a name="adf-linked-service-azure-sql"></a>Azure SQL データベース用のリンクされたサービス
Azure SQL Database 用にリンクされたサービスを作成するには、Azure クラシック ポータルで ADF ランディング ページ内の **[データ ストア]** をクリックし、*Azure SQL* を選択して Azure SQL Database の*ユーザー名*と*パスワード*に対して資格情報を入力します。*ユーザー名*は **user@servername* で指定する必要があります。


##<a name="adf-tables"></a>データセットへのアクセス方法を指定するためのテーブルを定義して作成する

以下のスクリプトベースの手順に従って、データセットの構造、場所、可用性を指定するテーブルを作成します。テーブルを定義するには、JSON ファイルを使用します。これらのファイルの構造の詳細については、「[データセット](../data-factory/data-factory-create-datasets.md)」を参照してください。

> [AZURE.NOTE]  `Add-AzureAccount` コマンドレットを実行してから [New-AzureDataFactoryTable](https://msdn.microsoft.com/library/azure/dn835096.aspx) コマンドレットを実行して、コマンドの実行に正しい Azure サブスクリプションが選択されていることを確認する必要があります。このコマンドレットの説明については、「[Add-AzureAccount](https://msdn.microsoft.com/library/azure/dn790372.aspx)」を参照してください。

テーブル内の JSON ベースの定義では、次の名前が使用されます。

* オンプレミスの SQL サーバーでは、**テーブル名**は *nyctaxi\_data* です。
* Azure BLOB ストレージ アカウントでは、**コンテナー名**は *containername* です。  

この ADF パイプラインには、次の 3 つのテーブル定義が必要です。

1. [オンプレミスの SQL テーブル](#adf-table-onprem-sql)
2. [BLOB テーブル](#adf-table-blob-store)
3. [SQL Azure テーブル](#adf-table-azure-sql)

> [AZURE.NOTE]  次の手順では、Azure PowerShell を使用して ADF アクティビティの定義と作成を行います。これらのタスクは、Azure ポータルを使用しても行えます。詳細については、「[入力データセットと出力データセットを作成する](../data-factory/data-factory-move-data-between-onprem-and-cloud.md#step-3-create-input-and-output-datasets)」を参照してください。

###<a name="adf-table-onprem-sql"></a>オンプレミスの SQL テーブル

オンプレミスの SQL Server のテーブル定義は、次の JSON ファイルで指定されます。

    	{
	    	"name": "OnPremSQLTable",
	    	"properties":
	    	{
		    	"location":
		    	{
		    	"type": "OnPremisesSqlServerTableLocation",
		    	"tableName": "nyctaxi_data",
		    	"linkedServiceName": "adfonpremsql"
		    	},
		    	"availability":
		    	{
		    	"frequency": "Day",
		    	"interval": 1,   
		    	"waitOnExternal":
		    	{
		    	"retryInterval": "00:01:00",
		    	"retryTimeout": "00:10:00",
		    	"maximumRetry": 3
		    	}

		    	}
	    	}
    	}
ここでは列名が含まれていないことに注意してください。ここに列名を含めることで、列名を副選択できます (詳細については、[ADF のドキュメント](../data-factory/data-factory-data-movement-activities.md)を参照)。

テーブルの JSON 定義を *onpremtabledef.json* というファイルにコピーし、それを既知の場所に保存します (ここでは、*C:\\temp\\onpremtabledef.json*)。次の Azure PowerShell コマンドレッドを使用して、ADF 内にテーブルを作成します。

	New-AzureDataFactoryTable -ResourceGroupName ADFdsprg -DataFactoryName ADFdsp –File C:\temp\onpremtabledef.json


###<a name="adf-table-blob-store"></a>BLOB テーブル
以下は、出力 BLOB の場所用のテーブル定義です (これはオンプレミスから取り込まれたデータを Azure BLOB にマップします)。

	    {
		    "name": "OutputBlobTable",
		    "properties":
		    {
			    "location":
			    {
			    "type": "AzureBlobLocation",
			    "folderPath": "containername",
			    "format":
			    {
			    "type": "TextFormat",
			    "columnDelimiter": "\t"
			    },
			    "linkedServiceName": "adfds"
			    },
			    "availability":
			    {
			    "frequency": "Day",
			    "interval": 1
			    }
		    }
	    }

テーブルの JSON 定義を *bloboutputtabledef.json* というファイルにコピーし、それを既知の場所に保存します (ここでは、*C:\\temp\\bloboutputtabledef.json*)。次の Azure PowerShell コマンドレッドを使用して、ADF 内にテーブルを作成します。

	New-AzureDataFactoryTable -ResourceGroupName adfdsprg -DataFactoryName adfdsp -File C:\temp\bloboutputtabledef.json  

###<a name="adf-table-azure-sq"></a>SQL Azure テーブル
以下は、SQL Azure 出力の場所用のテーブル定義です (このスキーマは BLOB からのデータをマップします)。

	{
	    "name": "OutputSQLAzureTable",
	    "properties":
	    {
	        "structure":
	        [
				{ "name": "column1", type": "String"},
				{ "name": "column2", type": "String"}                
	        ],
	        "location":
	        {
	            "type": "AzureSqlTableLocation",
	            "tableName": "your_db_name",
	            "linkedServiceName": "adfdssqlazure_linked_servicename"
	        },
	        "availability":
	        {
	            "frequency": "Day",
	            "interval": 1            
	        }
	    }
	}

テーブルの JSON 定義を *AzureSqlTable.json* というファイルにコピーし、それを既知の場所に保存します (ここでは、*C:\\temp\\AzureSqlTable.json*)。次の Azure PowerShell コマンドレッドを使用して、ADF 内にテーブルを作成します。

	New-AzureDataFactoryTable -ResourceGroupName adfdsprg -DataFactoryName adfdsp -File C:\temp\AzureSqlTable.json  


##<a name="adf-pipeline"></a>パイプラインを作成して定義する

次のスクリプトベースの手順に従って、パイプラインに属するアクティビティを指定し、パイプラインを作成します。パイプラインのプロパティを定義するため、JSON ファイルを使用します。

* このスクリプトでは、**パイプライン名**を *AMLDSProcessPipeline* としています。
* また、既定の実行時間 (12 am UTC) を使用して、ジョブが毎日実行されるようにパイプラインの周期性を設定していることにも注目してください。

> [AZURE.NOTE]  次の手順では、Azure PowerShell を使用して ADF パイプラインの定義と作成を行います。このタスクは、Azure ポータルを使用しても行えます。詳細については、「[パイプラインを作成して実行する](../data-factory/data-factory-move-data-between-onprem-and-cloud.md#step-4-create-and-run-a-pipeline)」を参照してください。

上記のテーブル定義を使用して、ADF のパイプライン定義を次のように指定します。

		{
		    "name": "AMLDSProcessPipeline",
		    "properties":
		    {
		        "description" : "This pipeline has one Copy activity that copies data from an on-premise SQL to Azure blob",
		         "activities":
		        [
		            {
		                "name": "CopyFromSQLtoBlob",
		                "description": "Copy data from on-premise SQL server to blob",     
		                "type": "CopyActivity",
		                "inputs": [ {"name": "OnPremSQLTable"} ],
		                "outputs": [ {"name": "OutputBlobTable"} ],
		                "transformation":
		                {
		                    "source":
		                    {                               
		                        "type": "SqlSource",
		                        "sqlReaderQuery": "select * from nyctaxi_data"
		                    },
		                    "sink":
		                    {
		                        "type": "BlobSink"
		                    }   
		                },
		                "Policy":
		                {
		                    "concurrency": 3,
		                    "executionPriorityOrder": "NewestFirst",
		                    "style": "StartOfInterval",
		                    "retry": 0,
		                    "timeout": "01:00:00"
		                }       

		             },

					{
						"name": "CopyFromBlobtoSQLAzure",
						"description": "Push data to Sql Azure",		
						"type": "CopyActivity",
						"inputs": [ {"name": "OutputBlobTable"} ],
						"outputs": [ {"name": "OutputSQLAzureTable"} ],
						"transformation":
						{
							"source":
							{                               
								"type": "BlobSource"
							},
							"sink":
							{
								"type": "SqlSink",
								"WriteBatchTimeout": "00:5:00",				
							}			
						},
						"Policy":
						{
							"concurrency": 3,
							"executionPriorityOrder": "NewestFirst",
							"style": "StartOfInterval",
							"retry": 2,
							"timeout": "02:00:00"
						}
				     }
		        ]
		    }
		}

パイプラインのこの JSON 定義を *pipelinedef.json* というファイルにコピーし、それを既知の場所に保存します (ここでは、*C:\\temp\\pipelinedef.json*)。次の Azure PowerShell コマンドレッドを使用して、ADF 内にパイプラインを作成します。

	New-AzureDataFactoryPipeline  -ResourceGroupName adfdsprg -DataFactoryName adfdsp -File C:\temp\pipelinedef.json

Azure クラシック ポータルで (図をクリックすると) ADF 上にパイプラインが次のように表示されることを確認します。

![](media/machine-learning-data-science-move-sql-azure-adf/DJP1kji.png)


##<a name="adf-pipeline-start"></a>パイプラインを開始する
これで、次のコマンドを使用してパイプラインを実行できます。

	Set-AzureDataFactoryPipelineActivePeriod -ResourceGroupName ADFdsprg -DataFactoryName ADFdsp -StartDateTime startdateZ –EndDateTime enddateZ –Name AMLDSProcessPipeline

*startdate* と *enddate* のパラメーター値を、パイプラインを実行する実際の開始日と終了日に置き換える必要があります。

パイプラインを実行すると、BLOB に選択したコンテナー内に表示されるデータを確認することができます (1 日につき 1 ファイル)。

ADF が提供するデータを段階的にパイプ処理する機能をまだ活用していないことに注意してください。これを行う方法と ADF が提供するその他の機能の詳細については、[ADF のドキュメント](https://azure.microsoft.com/services/data-factory/)を参照してください。

<!---HONumber=AcomDC_0518_2016-->