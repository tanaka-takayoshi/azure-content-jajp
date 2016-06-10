<properties
	pageTitle="Oracle WebLogic Server および Database VM | Microsoft Azure"
	description="リソース マネージャーのデプロイ モデルを使用して、Windows Server 2012 で実行される Oracle WebLogic Server 12c および Oracle Database 12c Azure イメージを作成します。"
	services="virtual-machines-windows"
	authors="bbenz"
	documentationCenter=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="vm-windows"
	ms.workload="infrastructure-services"
	ms.date="06/22/2015"
	ms.author="bbenz" />

#Azure での Oracle WebLogic Server 12c および Oracle Database 12c の仮想マシンの作成

この記事では、Windows Server 2012 で実行しているマイクロソフト提供の Oracle WebLogic Server 12c および Oracle Database 12c のイメージに基づいて、Azure で仮想マシンを作成する方法を示しています。

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)]クラシック デプロイ モデル。


##Azure で Oracle WebLogic Server 12c および Oracle Database 12c の仮想マシンを作成するには

1. [Azure ポータル](https://portal.azure.com/)にサインインします。

2.	**[Marketplace]**、**[Compute]** をクリックし、続いて検索ボックスに**「Oracle」**と入力します。

3.	**[Windows Server 2012 での Oracle Database 12c および WebLogic Server 12c Standard Edition]** または **[Windows Server 2012 での Oracle Database 12c および WebLogic Server 12c Enterprise Edition]** のイメージを選択します。このイメージに関する情報 (最小の推奨サイズなど) を確認し、**[次へ]** をクリックします。

4.	仮想マシンの**ホスト名**を指定します。

5.	仮想マシンの**ユーザー名**を指定します。なお、このユーザー名は仮想マシンにリモートログインするためのもので、Oracle データベースのユーザー名ではありませんのでご注意ください。

6.	仮想マシンのパスワードを指定し確認するか、または Secure Shell (SSH) 公開キーを入力します。

7.	**[価格レベル]** を選択。既定では推奨される価格レベルが表示されます。すべての構成オプションを表示するには、右上の **[すべて表示]** をクリックします。

8. 必要に応じて、オプションの構成を設定します。次の考慮事項に従います。

	a.仮想マシン名で新しいストレージ アカウントを作成するには、**[ストレージ アカウント]** をそのままにします。

	b.**[可用性セット]** を「**未構成**」のままにします。

	c.この時点ではエンドポイントを追加しないでください。

9.	リソース グループを選択または作成します。詳細については、「[Azure ポータルを使用した Azure リソースの管理](../azure-portal/resource-group-portal.md)」を参照してください。

10. **[サブスクリプション]** を選択します。

11. **[場所]** を選択します。


##この仮想マシンでホストされているデータベースを作成するには

「[Azure での Oracle Database 12c 仮想マシンの作成](virtual-machines-windows-classic-create-oracle-database.md)」の「**Azure で Oracle Database 12c VM を使ってデータベースを作成するには**」セクションの手順に従ってください。

##この仮想マシンでホストされている Oracle WebLogic Server 12c を構成するには
「[Azure での Oracle WebLogic Server 12c 仮想マシンの作成](virtual-machines-windows-create-oracle-weblogic-server-12c.md)」の「**Azure で Oracle WebLogic Server 12c 仮想マシンを構成するには**」セクションの手順に従ってください。

##その他のリソース
[Oracle 仮想マシンのイメージに関するその他の考慮事項](virtual-machines-windows-classic-oracle-considerations.md)

[Oracle 仮想マシン イメージの一覧](virtual-machines-linux-classic-oracle-images.md)

[Java アプリケーションから Oracle Database に接続する](http://docs.oracle.com/cd/E11882_01/appdev.112/e12137/getconn.htm#TDPJD136)

[Microsoft Azure で Linux を使用する Oracle WebLogic Server 12c](http://www.oracle.com/technetwork/middleware/weblogic/learnmore/oracle-weblogic-on-azure-wp-2020930.pdf)

[Oracle Database 2 日間 DBA 12c リリース 1](http://docs.oracle.com/cd/E16655_01/server.121/e17643/toc.htm)

<!---HONumber=AcomDC_0413_2016-->