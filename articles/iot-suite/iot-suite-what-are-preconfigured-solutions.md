<properties
 pageTitle="Azure IoT の構成済みソリューション | Microsoft Azure"
 description="Azure IoT の構成済みソリューションとそのアーキテクチャ (追加リソースのリンクを含む) の説明。"
 services=""
 suite="iot-suite"
 documentationCenter=""
 authors="dominicbetts"
 manager="timlt"
 editor=""/>

<tags
 ms.service="iot-suite"
 ms.devlang="na"
 ms.topic="get-started-article"
 ms.tgt_pltfrm="na"
 ms.workload="na"
 ms.date="05/25/2016"
 ms.author="dobett"/>

# Azure IoT Suite の構成済みソリューションとは

Azure IoT Suite の構成済みソリューションとは、サブスクリプションを使用して Azure にデプロイできる一般的な IoT ソリューション パターンの実装です。構成済みソリューションは次の用途で使用できます。

- 独自の IoT ソリューションの開始点として。
- IoT ソリューションの設計と開発で一般的なパターンについて学習するため。

構成済みの各ソリューションは、シミュレートされたデバイスを使用してテレメトリを生成することができる、完全なエンド ツー エンドの実装です。

Azure にソリューションをデプロイして実行するだけでなく、完全なソース コードをダウンロードして、特定の IoT 要件を満たすようにソリューションをカスタマイズして拡張することができます。

> [AZURE.NOTE] 構成済みソリューションのいずれかをデプロイするには、[Microsoft Azure IoT Suite][lnk-azureiotsuite] にアクセスしてください。[IoT 事前構成済みソリューションの使用][lnk-preconf-get-started]に関する記事では、ソリューションのいずれかをデプロイして実行する方法について詳しく説明しています。

次の表は、ソリューションが特定の IoT 機能にどのようにマップされるかを示しています。

| 解決策 | データの取り込み | デバイス ID | コマンドと制御 | ルールとアクション | 予測分析 |
|------------------------|-----|-----|-----|-----|-----|
| [リモート監視][lnk-preconf-get-started] | あり | あり | あり | あり | - | 
| [予測的なメンテナンス][lnk-predictive-maintenance] | あり | あり | あり | あり | あり |

- *データの取り込み*: クラウドへの大規模なデータの取り込み。
- *デバイス ID*: すべての接続されたデバイスの一意の ID を管理します。
- *コマンドと制御*: クラウドからデバイスにメッセージを送信して、デバイスで何らかのアクションを実行します。
- *ルールとアクション*: ソリューション バックエンドでは、ルールを使用して、特定のデバイスからクラウドへのデータを操作します。
- *予測分析*: ソリューション バックエンドでは、デバイスからクラウドへのデータを分析して、特定のアクションを実行するタイミングを予測します。たとえば、航空機エンジンのテレメトリを分析して、エンジンのメンテナンス時期を判断できます。

## リモート監視の構成済みソリューションの概要

この記事では、リモート監視の構成済みソリューションについて説明します。このソリューションを選択したのは、他のソリューションと共通する一般的な設計要素がたくさん使用されているためです。

次の図は、リモート監視ソリューションの主な要素を示しています。以降のセクションでは、これらの要素について詳しく説明します。

![リモート監視の構成済みソリューションのアーキテクチャ][img-remote-monitoring-arch]

## デバイス

リモート監視の構成済みソリューションをデプロイすると、冷却デバイスをシミュレートする 4 つのシミュレートされたデバイスがソリューション内で事前にプロビジョニングされます。これらのシミュレートされたデバイスには、テレメトリを出力する温度と湿度モデルが組み込まれています。これらのシミュレートされたデバイスが含まれているのは、ソリューションを使用したエンド ツー エンドのデータ フローを示すためと、カスタム実装の開始点としてソリューションを使用してバックエンド アプリケーションを開発する場合に、テレメトリの便利なソースとコマンドのターゲットを提供するためです。

デバイスがリモート監視の構成済みソリューションの IoT Hub に初めて接続すると、IoT Hub に送信されるデバイス情報メッセージに、デバイスが応答できるコマンドの一覧が列挙されます。リモート監視の構成済みソリューションには、次のコマンドが用意されています。

- *デバイスの ping*: デバイスは、確認応答を伴って、このコマンドに応答します。これは、デバイスがまだアクティブでリッスンしていることを確認するのに便利です。
- *テレメトリの開始*: テレメトリの送信を開始するようデバイスに指示します。
- *テレメトリの停止*: テレメトリの送信を停止するようデバイスに指示します。
- *設定点温度の変更*: デバイスが送信するシミュレートされた温度テレメトリ値を制御します。これは、バックエンド ロジックのテストに便利です。
- *診断テレメトリ*: デバイスが外部温度をテレメトリとして送信するかどうかを制御します。
- *デバイス状態の変更*: デバイスが報告するデバイスの状態のメタデータ プロパティを設定します。これは、バックエンド ロジックのテストに便利です。

同じテレメトリを出力し、同じコマンドに応答するシミュレートされたデバイスをソリューションに追加できます。

## IoT Hub

この構成済みソリューションの IoT Hub インスタンスは、一般的な [IoT ソリューション アーキテクチャ][lnk-what-is-azure-iot]の*クラウド ゲートウェイ*に相当します。

IoT Hub は、1 つのエンドポイントのデバイスからテレメトリを受信します。また、IoT Hub は、デバイス固有のエンドポイントも保守します。各デバイスは、送信されたコマンドをエンドポイントで取得できます。

IoT Hub は、受信したテレメトリをサービス側のテレメトリ読み取りエンドポイントを介して使用できるようにします。

## Azure Stream Analytics

構成済みソリューションでは、次の 3 つの [Azure Stream Analytics][lnk-asa] \(ASA) ジョブを使用して、デバイスのテレメトリ ストリームをフィルターします。


- *DeviceInfo ジョブ* - デバイスが最初に接続するときに、または**デバイス状態の変更**コマンドに応答して送信されたデバイス登録固有のメッセージをソリューションのデバイス レジストリ (DocumentDB データベース) にルーティングするイベント ハブにデータを出力します。 
- *Telemetry ジョブ* - Azure Blob Storage にコールド ストレージの未加工のテレメトリをすべて送信し、ソリューションのダッシュボードに表示されるテレメトリの集計を行います。
- *Rules ジョブ* - テレメトリ ストリームをフィルターして、いずれかのルールのしきい値を超える値を絞り込み、そのデータをイベント ハブに出力します。ルールが実行されると、ソリューション ポータルのダッシュボード ビューにこのイベントがアラーム履歴テーブルの新しい行として表示され、ソリューション ポータルのルールとアクション ビューで定義した設定に基づいてアクションがトリガーされます。

この構成済みソリューションでは、ASA ジョブは一般的な [IoT ソリューション アーキテクチャ][lnk-what-is-azure-iot]の **IoT ソリューション バックエンド**の一部です。

## イベント プロセッサ

この構成済みソリューションでは、イベント プロセッサは一般的な [IoT ソリューション アーキテクチャ][lnk-what-is-azure-iot]の **IoT ソリューション バックエンド**の一部です。

**DeviceInfo** と **Rules** ASA ジョブは、他のバックエンド サービスに配信するために出力をイベント ハブに送信します。このソリューションでは、[Web ジョブ][lnk-web-job]で実行される [EventPocessorHost][lnk-event-processor] インスタンスを使用して、これらのイベント ハブからメッセージを読み取ります。**EventProcessorHost** では、**DeviceInfo** のデータを使用して、DocumentDB データベースのデバイス データを更新し、**Rules** のデータを使用して、ロジック アプリを呼び出し、ソリューション ポータルのアラートの表示を更新します。

## デバイス ID レジストリと DocumentDB

すべての IoT Hub には、デバイス キーを格納する[デバイス ID レジストリ][lnk-identity-registry]が含まれています。IoT Hub は、この情報を使用してデバイスを認証します (ハブに接続する前に、デバイスが登録され、有効なキーを持っている必要があります)。

このソリューションでは、デバイスに関する追加情報 (状態、サポートしているコマンド、その他のメタデータなど) を格納します。また、DocumentDB データベースを使用して、このソリューションに固有のデバイス データを格納します。ソリューション ポータルでは、表示および編集のために、この DocumentDB データベースからデータを取得します。

このソリューションでは、デバイス ID レジストリの情報と DocumentDB データベースのコンテンツの同期が維持されている必要もあります。**EventProcessorHost** では、**DeviceInfo** Stream Analytics ジョブのデータを使用して、同期を管理します。

## ソリューション ポータル

![ソリューションのダッシュボード][img-dashboard]

ソリューション ポータルは、構成済みソリューションの一部としてクラウドにデプロイされている Web ベースの UI です。ソリューション ポータルでは、次の操作を実行できます。

- ダッシュボードにテレメトリとアラームの履歴を表示します。
- 新しいデバイスをプロビジョニングします。
- デバイスを管理し、監視します。
- 特定のデバイスにコマンドを送信します。
- ルールとアクションを管理します。

この構成済みソリューションでは、ソリューション ポータルは **IoT ソリューション バックエンド**の一部です。また、一般的な [IoT ソリューション アーキテクチャ][lnk-what-is-azure-iot]の**処理とビジネスの接続**に含まれています。

## 次のステップ

IoT ソリューション アーキテクチャの詳細については、「[Microsoft Azure IoT Reference Architecture (Microsoft Azure IoT 参照アーキテクチャ)][lnk-refarch]」を参照してください。

IoT の構成済みソリューションの詳細については、次のリソースをご覧ください。

- [IoT 事前構成済みソリューションの使用][lnk-preconf-get-started]
- [予測的なメンテナンスの構成済みソリューションの概要][lnk-predictive-maintenance]

[img-remote-monitoring-arch]: ./media/iot-suite-what-are-preconfigured-solutions/remote-monitoring-arch1.png
[img-dashboard]: ./media/iot-suite-what-are-preconfigured-solutions/dashboard.png
[lnk-what-is-azure-iot]: iot-suite-what-is-azure-iot.md
[lnk-asa]: https://azure.microsoft.com/documentation/services/stream-analytics/
[lnk-event-processor]: ../event-hubs/event-hubs-programming-guide.md#event-processor-host
[lnk-web-job]: ../app-service-web/web-sites-create-web-jobs.md
[lnk-document-db]: https://azure.microsoft.com/documentation/services/documentdb/
[lnk-identity-registry]: ../iot-hub/iot-hub-devguide.md#device-identity-registry
[lnk-suite-overview]: iot-suite-overview.md
[lnk-preconf-get-started]: iot-suite-getstarted-preconfigured-solutions.md
[lnk-predictive-maintenance]: iot-suite-predictive-overview.md
[lnk-azureiotsuite]: https://www.azureiotsuite.com/
[lnk-refarch]: http://download.microsoft.com/download/A/4/D/A4DAD253-BC21-41D3-B9D9-87D2AE6F0719/Microsoft_Azure_IoT_Reference_Architecture.pdf

<!---HONumber=AcomDC_0601_2016-->