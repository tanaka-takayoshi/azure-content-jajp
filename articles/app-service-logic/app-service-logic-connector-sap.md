<properties
   pageTitle="Logic Apps での SAP コネクタの使用 | Microsoft Azure App Service"
   description="SAP コネクタまたは API アプリを作成、構成して、Azure App Service のロジック アプリで使用する方法"
   services="app-service\logic"
   documentationCenter=".net,nodejs,java"
   authors="harishkragarwal"
   manager="erikre"
   editor=""/>

<tags
   ms.service="app-service-logic"
   ms.devlang="multiple"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="integration"
   ms.date="02/18/2016"
   ms.author="sameerch"/>


# SAP コネクタの使用開始とロジック アプリへの追加

>[AZURE.NOTE] このコネクタは、お客様のフィードバックに基づいてアップグレードしています。このページは準備ができしだい更新します。新しいバージョンが提供される前に SAP コネクタを使用することはお勧めしません。本記事は、ロジック アプリの 2014-12-01-preview スキーマ バージョンを対象としています。

オンプレミスの SAP に接続して、RFC の呼び出し、メタデータの取得を実行します。ファイアウォールの内側にあるオンプレミスにインストールされた SAP の操作が必要になる場合があるシナリオがあります。SAP コネクタをフローで使用すると、さまざまなシナリオを実現できます。例をいくつか示します。

1.	Web またはモバイル ユーザーのフロント エンドを介して、SAP に存在するデータの一部を公開する
2.	目的の処理を行った後に SAP にデータを発行する
3.	ビジネス プロセスで使用するデータを SAP から抽出する

コネクタをロジック アプリで使用すると、フローの一部として、データをフェッチ、処理、またはプッシュ転送できます。この SAP コネクタをビジネス ワークフローに追加し、ロジック アプリ内のそのワークフローの一部としてデータを処理できます。


これらのシナリオに対しては、次の操作を実行する必要があります。

1. SAP コネクタ API アプリのインスタンスを作成する
2. API アプリがオンプレミスの SAP と通信できるようにハイブリッド接続を確立する
3. 作成した API アプリをロジックで使用して、目的のビジネス プロセスを実現する


## SAP コネクタ API アプリのインスタンスを作成する ##

コネクタは、ロジック アプリ内で作成することも、Azure Marketplace から直接作成することもできます。Marketplace からコネクタを作成するには、次の操作を実行します。

1. Azure のスタート画面で、**[Marketplace]** を選択します。
2. "SAP コネクタ" を検索して選択し、**[作成]** を選択します。
3. SAP コネクタを次のように構成します。
	1. 最初のブレードで、名前、アプリ サービス プランなどの一般的な詳細を入力します。

	2. パッケージの設定の一部として、SAP の資格情報を入力します。さらに、Azure Service Bus 接続文字列を入力します。これは、オンプレミスの SAP とのハイブリッド接続を確立するために使用されます。

	3. RFC、TRFC、BAPI、IDOC については、シナリオのニーズに基づいて構成します。複数の値を指定する必要がある場合は、値をコンマで区切ります。

![][1]

## 作成した SAP コネクタ API アプリを構成する ##

[参照]、[API アプリ]、[<Name of the API App just created>] の順に選択して、作成した API アプリを参照します。次の動作が表示されます。この段階でハイブリッド接続がまだ確立されていないため、セットアップは完了していません。![][2]

SAP コネクタでは、*任意の* SAP エンドポイントに接続するためのハイブリッド接続が必要になります。ハイブリッド接続を確立するには、次の操作を行います。

1. プライマリ接続文字列をコピーします。
2. [ダウンロードして構成] リンクをクリックします。
3. 開始されるインストール プロセスに従い、要求に応じてプライマリ接続文字列を入力します。
4. セットアップ プロセスが完了すると、次のようなダイアログ ボックスが表示されます。![][3]

詳細については、「[オンプレミスの SAP サーバーと統合する](app-service-logic-integrate-with-an-on-premise-sap-server.md)」を参照してください。

もう一度作成した API アプリを参照すると、ハイブリッド接続の状態が "接続中" と表示されます。![][4]

注: セカンダリ接続文字列に切り替える場合は、ハイブリッドのセットアップをやり直して、プライマリ接続文字列の代わりにセカンダリ接続文字列を指定します。

## ロジック アプリでの使い方 ##

SAP コネクタは、ロジック アプリ内でアクションまたはステップとしてのみ使用できます。

ロジック アプリを作成または編集するときに、前の手順で作成した SAP コネクタを選択します。これにより、選択可能な許可されているすべてのアクションが表示されます。![][5]

アクションを選択すると、そのアクションの入力パラメーターが表示されます。適切な値を入力し、チェック マーク アイコンをクリックします。![][6]

これで、ステップまたはアクションは、ロジック アプリで構成されたものとして表示されます。操作の出力が表示されます。この出力は後続のステップの入力として使用することができます。![][7]

ビジネス プロセスを定義するロジック アプリを完成させます。ロジック アプリを実行して、目的の操作を実現します。

## コネクタでできること
コネクタが作成されたため、ロジック アプリを使用してコネクタをビジネス ワークフローに追加できます。「[Logic Apps とは](app-service-logic-what-are-logic-apps.md)」を参照してください。

>[AZURE.NOTE] Azure アカウントにサインアップする前に Azure Logic Apps の使用を開始する場合は、「[Azure App Service アプリケーションの作成](https://tryappservice.azure.com/?appservice=logic)」を参照してください。App Service で有効期間の短いスターター ロジック アプリをすぐに作成できます。このサービスの利用にあたり、クレジット カードは必要ありません。契約も必要ありません。

「[Connectors and API Apps Reference (コネクタと API Apps のリファレンス)](http://go.microsoft.com/fwlink/p/?LinkId=529766)」で Swagger REST API のリファレンスを参照してください。

パフォーマンス統計をレビューし、コネクタに対するセキュリティを制御することもできます。[組み込みの API Apps とコネクタの管理と監視](app-service-logic-monitor-your-connectors.md)に関するページを参照してください。

<!--Image references-->
[1]: ./media/app-service-logic-connector-sap/Create.jpg
[2]: ./media/app-service-logic-connector-sap/BrowseSetupIncomplete.jpg
[3]: ./media/app-service-logic-connector-sap/HybridSetup.jpg
[4]: ./media/app-service-logic-connector-sap/BrowseSetupComplete.jpg
[5]: ./media/app-service-logic-connector-sap/LogicApp1.jpg
[6]: ./media/app-service-logic-connector-sap/LogicApp2.jpg
[7]: ./media/app-service-logic-connector-sap/LogicApp3.jpg

<!---HONumber=AcomDC_0420_2016-->