<properties 
	pageTitle="メディア プロセッサを作成する方法 | Microsoft Azure" 
	description="メディア プロセッサ コンポーネントを作成し、Azure Media Services 用にメディア コンテンツのエンコード、形式の変換、暗号化、または復号化を行う方法について説明します。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="erikre" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="03/01/2016"  
	ms.author="juliako"/>


#方法: メディア プロセッサ インスタンスを取得する


> [AZURE.SELECTOR]
- [.NET](media-services-get-media-processor.md)
- [REST ()](media-services-rest-get-media-processor.md)

##概要

メディア プロセッサは、Media Services のコンポーネントとして、メディア コンテンツのエンコード、形式変換、暗号化、復号化など、特定の処理タスクを担います。通常、メディア コンテンツのエンコード、暗号化、形式変換を行うタスクの作成時にメディア プロセッサを作成します。

次の表は、利用可能なメディア プロセッサの名前と説明の一覧です。

メディア プロセッサ名|説明|詳細情報
---|---|---
メディア エンコーダー スタンダード|オンデマンド エンコードのための標準機能を備えています。 |[Azure オンデマンド メディア エンコーダーの概要と比較](media-services-encode-asset.md)
メディア エンコーダー Premium ワークフロー|メディア エンコーダー Premium ワークフローを使用してエンコード タスクを実行できます。|[Azure オンデマンド メディア エンコーダーの概要と比較](media-services-encode-asset.md)
Azure Media Indexer| メディア ファイルとコンテンツを検索可能にすると共に、クローズド キャプション トラックの生成を可能にします。|[Azure Media Indexer](media-services-index-content.md)
Azure Media Hyperlapse (プレビュー)|ビデオ安定化を使用して、ビデオの "凸凹" を取り除いて滑らかにすることができます。コンテンツをすばやく使用可能なクリップにすることもできます。|[Azure Media Hyperlapse](media-services-hyperlapse-content.md)
Azure Media Encoder|償却対象
Storage Decryption| 償却対象|
Azure Media Packager|償却対象|
Azure Media Encryptor|償却対象|

##MediaProcessor の取得

>[AZURE.NOTE] Media Services REST API を使用する場合は、次のことに考慮します。
>
>Media Services でエンティティにアクセスするときは、HTTP 要求で特定のヘッダー フィールドと値を設定する必要があります。詳細については、「[Media Services REST API の概要](media-services-rest-how-to-use.md)」をご覧ください。

>https://media.windows.net に正常に接続すると、別の Media Services URI が指定された 301 リダイレクトが表示されます。「[Media Services REST API を使用して Media Services アカウントに接続する](media-services-rest-connect_programmatically.md)」で説明するとおり、続けて新しい URI を呼び出す必要があります。


次の REST 呼び出しは、メディア プロセッサ インスタンスを名前 (ここでは **Media Encoder Standard**) で取得する方法を示しています。



	
要求:

	GET https://media.windows.net/api/MediaProcessors()?$filter=Name%20eq%20'Media%20Encoder%20Standard' HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	User-Agent: Microsoft ADO.NET Data Services
	Authorization: Bearer <token>
	x-ms-version: 2.11
	Host: media.windows.net
	
応答:
		
	. . .
	
	{  
	   "odata.metadata":"https://media.windows.net/api/$metadata#MediaProcessors",
	   "value":[  
	      {  
	         "Id":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
	         "Description":"Media Encoder Standard",
	         "Name":"Media Encoder Standard",
	         "Sku":"",
	         "Vendor":"Microsoft",
	         "Version":"1.1"
	      }
	   ]
	}


##Media Services のラーニング パス

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##フィードバックの提供

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]


##次のステップ

これで、メディア プロセッサ インスタンスを取得する方法がわかりました。次は、「[Media Encoder Standard を使用して資産をエンコードする方法](media-services-rest-get-started.md)」に進んでください。このトピックでは、Media Encoder Standard を使用して資産をエンコードする方法を説明します。

<!---HONumber=AcomDC_0302_2016-->