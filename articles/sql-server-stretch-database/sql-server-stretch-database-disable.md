<properties
	pageTitle="Stretch Database を無効にしてリモート データを戻す | Microsoft Azure"
	description="テーブルの Stretch Database を無効にし、必要に応じてリモート データを戻す方法について説明します。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglaslMS"
	manager=""
	editor=""/>

<tags
	ms.service="sql-server-stretch-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="05/17/2016"
	ms.author="douglasl"/>

# Stretch Database を無効にしてリモート データを戻す

テーブルの Stretch Database を無効にするには、SQL Server Management Studio でテーブルに **[Stretch]** を選択します。次のいずれかを選択します。

-   **無効にする | Azure からデータを戻す**。Azure から SQL Server にテーブルのリモート データをコピーし、テーブルの Stretch Database を無効にします。この操作にはデータ転送コストが発生し、キャンセルできません。

-   **無効にする | Azure にデータを残す**。テーブルの Stretch Database を無効にします。Azure のテーブルのリモート データを破棄します。

テーブルの Stretch Database を無効にすると、データ移行が停止し、クエリ結果にリモート テーブルからの結果が含まれなくなります。

Transact-SQL を利用し、テーブルまたはデータベースの Stretch Database を無効にすることもできます。

データ移行を一時停止する場合、「[Pause and resume Stretch Database (Stretch Database を一時停止し、再開する)](sql-server-stretch-database-pause.md)」を参照してください。

>   [AZURE.NOTE] テーブルまたはデータベースの Stretch Database を無効にしても、リモート オブジェクトは削除されません。リモート テーブルまたはリモート データベースを削除する場合は、Microsoft Azure 管理ポータルを使用して削除する必要があります。リモート オブジェクトを削除するまで、Azure Storage のコストが引き続き発生します。詳細については、「[SQL Server Stretch Database の価格](https://azure.microsoft.com/pricing/details/sql-server-stretch-database/)」をご覧ください。

## テーブルの Stretch Database を無効にする

### SQL Server Management Studio を使用し、テーブルの Stretch Database を無効にします。

1.  SQL Server Management Studio のオブジェクト エクスプローラーで、Stretch Database を無効にするテーブルを選択します。

2.  右クリックして **[Stretch]** を選択し、次のいずれかを選択します。

    -   **無効にする | Azure からデータを戻す**。Azure から SQL Server にテーブルのリモート データをコピーし、テーブルの Stretch Database を無効にします。このコマンドはキャンセルできません。

        >   [AZURE.NOTE] Azure から SQL Server にテーブルのリモート データをコピーすると、データ転送コストが発生します。詳細については、「[Data Transfers (データ転送) の料金詳細](https://azure.microsoft.com/pricing/details/data-transfers/)」をご覧ください。

        Azure から SQL Server にすべてのリモート データがコピーされると、テーブルの Stretch が無効になります。

    -   **無効にする | Azure にデータを残す**。テーブルの Stretch Database を無効にします。Azure のテーブルのリモート データを破棄します。

    >   [AZURE.NOTE] テーブルの Stretch Database を無効にしても、リモート データおよびリモート テーブルは削除されません。リモート テーブルを削除する場合は、Microsoft Azure 管理ポータルを使用して削除する必要があります。リモート テーブルを削除するまで、Azure Storage のコストが引き続き発生します。詳細については、「[SQL Server Stretch Database の価格](https://azure.microsoft.com/pricing/details/sql-server-stretch-database/)」をご覧ください。

### Transact-SQL を利用し、テーブルの Stretch Database を無効にする

-   テーブルの Stretch を無効にして Azure から SQL Server にテーブルのリモート データをコピーするには、次のコマンドを実行します。このコマンドはキャンセルできません。

    ```tsql
    ALTER TABLE <table name>
       SET ( REMOTE_DATA_ARCHIVE ( MIGRATION_STATE = INBOUND ) ) ;
    ```
    >   [AZURE.NOTE] Azure から SQL Server にテーブルのリモート データをコピーすると、データ転送コストが発生します。詳細については、「[Data Transfers (データ転送) の料金詳細](https://azure.microsoft.com/pricing/details/data-transfers/)」をご覧ください。

    Azure から SQL Server にすべてのリモート データがコピーされると、テーブルの Stretch が無効になります。

-   テーブルの Stretch を無効にしてリモート データを破棄するには、次のコマンドを実行します。

    ```tsql
    ALTER TABLE <table_name>
       SET ( REMOTE_DATA_ARCHIVE = OFF_WITHOUT_DATA_RECOVERY ( MIGRATION_STATE = PAUSED ) ) ;
    ```

>   [AZURE.NOTE] テーブルの Stretch Database を無効にしても、リモート データおよびリモート テーブルは削除されません。リモート テーブルを削除する場合は、Microsoft Azure 管理ポータルを使用して削除する必要があります。リモート テーブルを削除するまで、Azure Storage のコストが引き続き発生します。詳細については、「[SQL Server Stretch Database の価格](https://azure.microsoft.com/pricing/details/sql-server-stretch-database/)」をご覧ください。

## データベースの Stretch Database を無効にする
データベースの Stretch Database を無効にする前に、データベースの個々の Stretch 対応テーブルで Stretch Database を無効にする必要があります。

### SQL Server Management Studio を使用し、データベースの Stretch Database を無効にする

1.  SQL Server Management Studio のオブジェクト エクスプローラーで、Stretch Database を無効にするデータベースを選択します。

2.  右クリックして **[タスク]** を選択します。次に、**[Stretch]** を選択し、**[無効化]** を選択します。

>   [AZURE.NOTE] データベースの Stretch Database を無効にしても、リモート データベースは削除されません。リモート データベースを削除する場合は、Microsoft Azure 管理ポータルを使用して削除する必要があります。リモート データベースを削除するまで、Azure Storage のコストが引き続き発生します。詳細については、「[SQL Server Stretch Database の価格](https://azure.microsoft.com/pricing/details/sql-server-stretch-database/)」をご覧ください。

### Transact-SQL を利用し、データベースの Stretch Database を無効にする
次のコマンドを実行します。

```tsql
ALTER DATABASE <database name>
    SET REMOTE_DATA_ARCHIVE = OFF ;
```

>   [AZURE.NOTE] データベースの Stretch Database を無効にしても、リモート データベースは削除されません。リモート データベースを削除する場合は、Microsoft Azure 管理ポータルを使用して削除する必要があります。リモート データベースを削除するまで、Azure Storage のコストが引き続き発生します。詳細については、「[SQL Server Stretch Database の価格](https://azure.microsoft.com/pricing/details/sql-server-stretch-database/)」をご覧ください。

## 関連項目

[ALTER DATABASE SET のオプション (Transact-SQL)](https://msdn.microsoft.com/library/bb522682.aspx)

[Stretch Database を一時停止し、再開します。](sql-server-stretch-database-pause.md)

<!---HONumber=AcomDC_0518_2016-->