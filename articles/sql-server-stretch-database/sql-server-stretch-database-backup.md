<properties
	pageTitle="Stretch 対応データベースをバックアップし、復元する | Microsoft Azure"
	description="Stretch が有効なデータベースをバックアップし、復元する方法について説明します。"
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


# Stretch 対応データベースをバックアップし、復元する

Stretch 対応データベースをバックアップし、復元するには、現在使用している方法を引き続き使用できます。SQL Server のバックアップと復元の詳細については、「[SQL Server データベースのバックアップと復元](https://msdn.microsoft.com/library/ms187048.aspx)」を参照してください。

Stretch 対応データベースのバックアップは、リモート サーバーに移行されるデータを含まない浅い (shallow) バックアップとなります。

Stretch Database は、ポイントインタイム リストアに完全対応しています。ある時点に SQL Server データベースを復元し、Azure への接続を再認証すると、Stretch Database はその同じ時点とリモート データを一致させます。SQL Server のポイントインタイム リストアの詳細については、「[SQL Server データベースを特定の時点に復元する方法 (完全復旧モデル)](https://msdn.microsoft.com/library/ms179451.aspx)」を参照してください。Azure への接続を再認証するために復元後に実行するストアド プロシージャについては、「[sys.sp\_rda\_reauthorize\_db (Transact-SQL)](https://msdn.microsoft.com/library/mt131016.aspx)」を参照してください。

## <a name="Reconnect"></a>Stretch 対応データベースをバックアップから復元する

1.  バックアップからデータベースを復元します。

2.  ストアド プロシージャ [sys.sp\_rda\_reauthorize\_db (Transact-SQL)](https://msdn.microsoft.com/library/mt131016.aspx) を実行し、Stretch を有効にしたローカル データベースを Azure に再接続します。

    -   既存のデータベース スコープ資格情報を sysname または varchar(128) 値として指定します。(varchar(max) は使用しないでください。) 資格情報の名前は、**sys.database\_scoped\_credentials** ビューで調べることができます。

	-   リモート データのコピーを作成して、そのコピーに接続するかどうかを指定します。

    ```tsql
    Declare @credentialName nvarchar(128);
    SET @credentialName = N'<database_scoped_credential_name_created_previously>';
    EXEC sp_rda_reauthorize_db @credential = @credentialName, @with_copy = 0;
    ```

## <a name="MoreInfo"></a>バックアップとリストアの詳細
Stretch データベースが有効になっているデータベースのバックアップには、バックアップ実行時のローカル データと有資格データのみが含まれています。これらのバックアップには、データベースのリモート データが置かれているリモート エンドポイントに関する情報も含まれています。これは "shallow backup (浅いバックアップ)" と呼ばれています。ローカル データベースとリモート データベースの両方の全データが含まれる deep backup (深いバックアップ) には対応していません。

Stretch Database が有効になっているデータベースのバックアップを復元すると、ローカル データと有資格データがデータベースに予想どおりに復元されます。(有資格データとは、まだ移されていないが、テーブルの Stretch Database 構成に基づき、Azure に移されるデータのことです。) 復元を実行すると、バックアップの実行時点からのローカル データと有資格データがデータベースに含まれますが、リモート エンドポイントに接続するために必要な資格情報とアーティファクトは含まれていません。

ローカル データベースとそのリモート エンドポイントの間の接続を再確立するには、ストアド プロシージャ **sys.sp\_rda\_reauthorize\_db** を実行する必要があります。db\_owner だけがこの操作を実行できます。このストアド プロシージャには、ターゲット Azure サーバーの管理者のログインとパスワードも必要です。

接続を再確立すると、Stretch Database はリモート エンドポイントのリモート データをコピーし、それをローカル データベースとリンクさせることでローカル データベースの有資格データとリモート データの一致を試行します。このプロセスは自動であり、ユーザーの介入を必要としません。この照合の実行後、ローカル データベースとリモート エンドポイントが一致している状態になります。

## 関連項目

[Stretch Database を管理し、問題を解決します。](sql-server-stretch-database-manage.md)

[sys.sp\_rda\_reauthorize\_db (Transact-SQL)](https://msdn.microsoft.com/library/mt131016.aspx)

[SQL Server データベースのバックアップと復元](https://msdn.microsoft.com/library/ms187048.aspx)

<!---HONumber=AcomDC_0518_2016-->