<properties
	pageTitle="Azure Functions における Mobile Apps のバインド | Microsoft Azure"
	description="Azure Functions で Azure Mobile Apps のバインドを使用する方法について説明します。"
	services="functions"
	documentationCenter="na"
	authors="christopheranderson"
	manager="erikre"
	editor=""
	tags=""
	keywords="Azure Functions, 関数, イベント処理, 動的コンピューティング, サーバーなしのアーキテクチャ"/>

<tags
	ms.service="functions"
	ms.devlang="multiple"
	ms.topic="reference"
	ms.tgt_pltfrm="multiple"
	ms.workload="na"
	ms.date="05/16/2016"
	ms.author="chrande"/>

# Azure Functions における Mobile Apps のバインド

この記事では、Azure Functions で Azure Mobile Apps のバインドを構成したりコーディングしたりする方法について説明します。

[AZURE.INCLUDE [intro](../../includes/functions-bindings-intro.md)]

Azure App Service Mobile Apps を使用すると、テーブル エンドポイント データをモバイル クライアントに公開できます。この同じ表形式のデータは、Azure Functions の入力バインドと出力バインドの両方で使用できます。動的スキーマをサポートしているため、Node.js バックエンドのモバイル アプリは、関数で使用するために表形式のデータを公開するのに最適です。動的スキーマは既定で有効になっていますが、運用環境のモバイル アプリでは無効にする必要があります。Node.js バックエンドにおけるテーブル エンドポイントの詳細については、「[概要: テーブル操作](../app-service-mobile/app-service-mobile-node-backend-how-to-use-server-sdk.md#TableOperations)」を参照してください。Mobile Apps では、Node.js バックエンドは、ポータル内の参照とテーブルの編集をサポートします。詳細については、Node.js SDK トピックの「[ポータルでの編集](../app-service-mobile/app-service-mobile-node-backend-how-to-use-server-sdk.md#in-portal-editing)」を参照してください。Azure Functions で .NET バックエンドのモバイル アプリを使用する場合は、関数からの必要に応じて、データ モデルを手動で更新する必要があります。.NET バックエンドのモバイル アプリのテーブル エンドポイントの詳細については、.NET バックエンドの SDK トピックの「[方法: テーブル コントローラーを定義する](../app-service-mobile/app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#define-table-controller)」を参照してください。

## <a id="mobiletablesapikey"></a>API キーを使用した Mobile Apps テーブル エンドポイントへの安全なアクセス

Azure Functions のモバイル テーブルのバインドでは、API キーを指定できます。これは、関数以外のアプリからの望ましくないアクセスを回避するために使用できる共有シークレットです。Mobile Apps には、API キー認証向けのサポートが組み込まれていません。ただし、「[API キーを実装する Azure App Service Mobile Apps バックエンド](https://github.com/Azure/azure-mobile-apps-node/tree/master/samples/api-key)」の例に従って、Node.js バックエンド モバイル アプリに API キーを実装できます。同様に、[.NET バックエンドのモバイル アプリ](https://github.com/Azure/azure-mobile-apps-net-server/wiki/Implementing-Application-Key)に API キーを実装することができます。

>[AZURE.IMPORTANT] この API キーはモバイル アプリ クライアントで配布する必要があり、Azure Functions のようなサービス側のクライアントにのみ安全に配布する必要があります。

## <a id="mobiletablesinput"></a>Azure Mobile Apps の入力バインド

入力バインドでは、モバイル テーブル エンドポイントからレコードを読み込んで、バインドに直接渡すことができます。レコード ID は、関数を呼び出したトリガーに基づいて決定されます。C# 関数で、レコードに加えられた変更は、関数が正常に終了したときに、テーブルに自動的に再送信されます。

#### Mobile Apps 入力バインドの function.json

*function.json* ファイルは、次のプロパティをサポートします。

- `name`: 新しいレコードの関数コードで使用される変数名。
- `type`: バインドの型は *mobileTable* に設定する必要があります。
- `tableName`: 新しいレコードの作成先のテーブル。
- `id`: 取得するレコードの ID。このプロパティは、`{queueTrigger}` と同様のバインドをサポートします。ここでは、レコード ID としてキュー メッセージの文字列値を使用します。
- `apiKey`: モバイル アプリ用のオプションの API キーを指定するアプリケーション設定である文字列。これは、モバイル アプリが API キーを使用してクライアント アクセスを制限するときに必要です。
- `connection`: モバイル アプリの URI を指定するアプリケーション設定である文字列。
- `direction`: バインドの方向。*in* に設定する必要があります。

*function.json* ファイルの例:

	{
	  "bindings": [
	    {
	      "name": "record",
	      "type": "mobileTable",
	      "tableName": "MyTable",
	      "id" : "{queueTrigger}",
	      "connection": "My_MobileApp_Uri",
	      "apiKey": "My_MobileApp_Key",
	      "direction": "in"
	    }
	  ],
	  "disabled": false
	}

#### Azure Mobile Apps のコード例 (C# キュー トリガー)

上記の function.json の例に基づいて、入力バインドはキュー メッセージ文字列と一致する ID を持つ Mobile Apps テーブル エンドポイントからレコードを取得し、それを *record* パラメーターに渡します。レコードが検出されなかった場合、パラメーターは null になります。関数の終了時に、レコードは新しい *Text* 値で更新されます。

	#r "Newtonsoft.Json"	
	using Newtonsoft.Json.Linq;
	
	public static void Run(string myQueueItem, JObject record)
	{
	    if (record != null)
	    {
	        record["Text"] = "This has changed.";
	    }    
	}

#### Azure Mobile Apps のコード例 (Node.js キュー トリガー)

上記の function.json の例に基づいて、入力バインドはキュー メッセージ文字列と一致する ID を持つ Mobile Apps テーブル エンドポイントからレコードを取得し、それを *record* パラメーターに渡します。Node.js 関数では、更新されたレコードは再びテーブルに送信されません。このコード例では、取得されたレコードをログに書き込みます。

	module.exports = function (context, input) {    
	    context.log(context.bindings.record);
	    context.done();
	};


## <a id="mobiletablesoutput"></a>Azure Mobile Apps の出力バインド

関数により、出力バインドを使用して、レコードを Mobile Apps テーブル エンドポイントに書き込めます。

#### Mobile Apps 出力バインドの function.json

function.json ファイルは、次のプロパティをサポートします。

- `name`: 新しいレコードの関数コードで使用される変数名。
- `type`: *mobileTable* に設定する必要があるバインドの型。
- `tableName`: 新しいレコードが作成されるテーブル。
- `apiKey`: モバイル アプリ用のオプションの API キーを指定するアプリケーション設定である文字列。これは、モバイル アプリが API キーを使用してクライアント アクセスを制限するときに必要です。
- `connection`: モバイル アプリの URI を指定するアプリケーション設定である文字列。
- `direction`: バインドの方向。*out* に設定する必要があります。

function.json の例:

	{
	  "bindings": [
	    {
	      "name": "record",
	      "type": "mobileTable",
	      "tableName": "MyTable",
	      "connection": "My_MobileApp_Uri",
	      "apiKey": "My_MobileApp_Key",
	      "direction": "out"
	    }
	  ],
	  "disabled": false
	}

#### Azure Mobile Apps のコード例 (C# キュー トリガー)

この C# コード例では、*Text* プロパティを持つ Mobile Apps テーブル エンドポイントを、上のバインドで指定したテーブルに挿入します。

	public static void Run(string myQueueItem, out object record)
	{
	    record = new {
	        Text = $"I'm running in a C# function! {myQueueItem}"
	    };
	}

#### Azure Mobile Apps のコード例 (Node.js キュー トリガー)

この Node.js コード例では、*Text* プロパティを持つ Mobile Apps テーブル エンドポイントを、上のバインドで指定したテーブルに挿入します。

	module.exports = function (context, input) {
	
	    context.bindings.record = {
	        text : "I'm running in a Node function! Data: '" + input + "'"
	    }   
	
	    context.done();
	};

## 次のステップ

[AZURE.INCLUDE [次のステップ](../../includes/functions-bindings-next-steps.md)]

<!---HONumber=AcomDC_0525_2016-->