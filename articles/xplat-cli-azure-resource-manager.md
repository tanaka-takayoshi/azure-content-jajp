
<properties
	pageTitle="リソース マネージャーでの Azure CLI | Microsoft Azure"
	description="Azure コマンド ライン インターフェイス (CLI) を利用し、リソース グループとして複数のリソースをデプロイします。"
	editor=""
	manager="timlt"
	documentationCenter=""
	authors="dlepow"
	services="azure-resource-manager"/>

<tags
	ms.service="azure-resource-manager"
	ms.workload="multiple"
	ms.tgt_pltfrm="vm-multiple"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/20/2016"
	ms.author="danlep"/>

# Azure リソース マネージャーでの、Mac、Linux、および Windows 用 Azure CLI の使用

> [AZURE.SELECTOR]
- [Azure CLI](xplat-cli-azure-resource-manager.md)
- [Azure PowerShell](powershell-azure-resource-manager.md)



この記事では、Azure Resource Manager モードで、Azure コマンド ライン インターフェイス (Azure CLI) を使用して、Azure リソースを作成し、管理する一般的な方法を紹介します。

>[AZURE.NOTE] コマンド ラインで Azure リソースを作成して管理するには、Azure サブスクリプションが必要です ([無料の Azure アカウントはこちら](https://azure.microsoft.com/free/))。[Azure CLI をインストール](xplat-cli-install.md)し、[アカウントに関連付けられている Azure のリソースを使用するためにログインする](xplat-cli-connect.md)必要があります。これらの操作を完了した場合は、使用する準備が整いました。

## Azure リソース

Azure リソース マネージャーを使用すると、複数の_リソース_ (仮想マシン、データベース サーバー、データベース、Web サイトなどのユーザー管理のエンティティ) を_リソース グループ_と呼ばれる 1 つの論理単位にまとめて作成し、管理できます。

Azure Resource Manager の長所の 1 つは、JSON *テンプレート*にリソースのデプロイ可能なグループの構造とリレーションシップを記述するという_宣言型_の方法で Azure のリソースを作成できることです。テンプレートはパラメーターを特定します。そのパラメーターに値をコマンドの実行時にインラインで入力するか、別の JavaScript Object Notation (JSON) パラメーター ファイルに保存できます。これにより、同じテンプレートに異なるパラメーターを設定して、新しいリソースを簡単に作成できます。たとえば、Web サイトを作成するテンプレートには、サイト名や Web サイトを配置するリージョンなどの共通的な設定を用意できます。

テンプレートを使用してグループを変更または作成すると、_デプロイ_が作成されてグループに適用されます。Azure リソース マネージャーの詳細については、「[Azure リソース マネージャーの概要](resource-group-overview.md)」をご覧ください。

デプロイを作成すると、クラシック デプロイ モデルと同様に、個々のリソースをコマンド ラインで強制的に管理できます。たとえば、Resource Manager モードで CLI コマンドを使用し、[Azure Resource Manager 仮想マシン](./virtual-machines/virtual-machines-linux-cli-deploy-templates.md)などのリソースを開始、停止、削除します。

## 認証

Azure CLI を使用して Azure Resource Manager を操作するには、現在は、`azure login` コマンドを実行し、Azure Active Directory によって管理されているアカウント (職場または学校のアカウント (組織アカウント) か Microsoft アカウント) を指定して、Microsoft Azure の認証を受ける必要があります。.publishsettings ファイルによってインストールされた証明書の場合、このモードでは認証できません。

Microsoft Azure の認証の詳細については、「[Azure CLI から Azure サブスクリプションへの接続する](xplat-cli-connect.md)」をご覧ください。

>[AZURE.NOTE] Azure Active Directory によって管理されているアカウントを使用すると、Azure のロールベースのアクセス制御 (RBAC) を使用して、Azure リソースへのアクセスと使用状況を管理することもできます。詳細については、「[ロールベースの Access Control](./active-directory/role-based-access-control-configure.md)」を参照してください。

## Resource Manager モードの設定

CLI は既定では Resource Manager モードになっていないため、次のコマンドを使って Azure CLI Resource Manager コマンドを有効にしてください。

	azure config mode arm

## 場所を見つける

ほとんどの Azure リソース マネージャー コマンドには、リソースを作成し、検索するための有効な場所が必要です。次のコマンドを使用して、異なる Azure リソースに利用可能なすべての場所を検索できます。

	azure location list

これによって、"米国西部" や "米国東部" などの利用可能な Azure リージョンが一覧表示されます。利用可能なリソース プロバイダーと、そのプロバイダーを利用できる場所の詳細については、`azure provider list` コマンドの後に `azure provider show` コマンドを実行して確認できます。たとえば、次のコマンドを実行すると、Azure コンテナー サービスの場所が一覧表示されます。

    azure provider show Microsoft.ContainerService 

## リソース グループの作成

リソース グループは、ネットワーク、ストレージ、コンピューティングなどのリソースの論理グループです。Resource Manager モードのほとんどすべてのコマンドにはリソース グループが必要です。たとえば、次のコマンドを実行すると、_testRG_ という名前のリソース グループを米国西部リージョンに作成できます。

	azure group create -n "testRG" -l "West US"

テンプレートを使って Ubuntu VM を起動すると、後でこの *testRG* リソース グループがデプロイ先となります。リソース グループの作成後、仮想マシンやネットワーク、ストレージなどのリソースを追加することができます。


## リソース グループ テンプレートの利用

### リソース グループ テンプレートの検索と構成

テンプレートを使用する場合は、[独自のテンプレートを作成する](resource-group-authoring-templates.md)か、コミュニティに投稿された[クイックスタート テンプレート](https://azure.microsoft.com/documentation/templates/)のいずれかを使用できます。クイックスタート テンプレートは [GitHub](https://github.com/Azure/azure-quickstart-templates) でも取得できます。

新しいテンプレートの作成はこの記事では扱いません。そのため、出発点として、[クイックスタート テンプレート](https://azure.microsoft.com/documentation/templates/101-vm-simple-linux/)で入手できる _101-simple-vm-from-image_ テンプレートを利用しましょう。既定では、これにより、サブネットが 1 つ設定された新しい仮想ネットワークに Ubuntu 14.04.2-LTS 仮想マシンが 1 つ作成されます。リソース グループと、次のパラメーターを指定するだけで、このテンプレートを使用できます。

* VM の管理者ユーザー名 = `adminUsername`
* パスワード = `adminPassword`
* VM のドメイン名 = `dnsLabelPrefix`

>[AZURE.TIP] 以下の手順では、Azure CLI で VM テンプレートを使用する方法を 1 つだけ紹介します。他の例については、「[Azure リソース マネージャー テンプレートと Azure CLI を使用した仮想マシンのデプロイと管理](./virtual-machines/virtual-machines-linux-cli-deploy-templates.md)」を参照してください。

1. [GitHub でさらに詳しく] リンクをクリックし、[GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-linux) からローカル コンピューターの作業フォルダーに azuredeploy.json ファイルと azuredeploy.parameters.json ファイルをダウンロードします (いずれのファイルも、GitHub で _Raw_ 形式を選択してください)。

2. テキスト エディターで azuredeploy.parameters.json ファイルを開き、お使いの環境に適したパラメーター値を入力します (**ubuntuOSVersion** 値は変更しないでください)。

	```
			{
			  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
			  "contentVersion": "1.0.0.0",
			  "parameters": {
			    "adminUsername": {
			      "value": "azureUser"
			    },
			    "adminPassword": {
			      "value": "GEN-PASSWORD"
			    },
			    "dnsLabelPrefix": {
			      "value": "GEN-UNIQUE"
			    },
			    "ubuntuOSVersion": {
			      "value": "14.04.2-LTS"
			    }
			  }
			}

	```
3.  デプロイ パラメーターを変更したら、先ほど作成した *testRG* リソース グループに Ubuntu VM をデプロイします。デプロイの名前 (この例では *testRGDeploy*) を選択し、次のコマンドを使用してデプロイを実行します。

	```
	azure group deployment create -f azuredeploy.json -e azuredeploy.parameters.json testRG testRGDeploy
	```

	`-e` オプションにより、前の手順で変更した azuredeploy.parameters.json ファイルが指定されます。`-f` オプションにより、azuredeploy.json テンプレート ファイルが指定されます。

	このコマンドでは、デプロイがアップロードされると OK と表示されますが、その時点ではまだグループのリソースにデプロイが適用されていません。

4. デプロイの状態を確認するには、次のコマンドを使用します。

	```
	azure group deployment show testRG testRGDeploy
	```

	**ProvisioningState** に、デプロイの状態が表示されます。

	デプロイが成功した場合は、次のような出力が表示されます。

		azure-cli@0.8.0:/# azure group deployment show testRG testRGDeploy
		info:    Executing command group deployment show
		+ Getting deployments
		+ Getting deployments
		data:    DeploymentName     : testDeploy
		data:    ResourceGroupName  : testRG
		data:    ProvisioningState  : Succeeded
		data:    Timestamp          : 
		data:    Mode               : Incremental
		data:    Name                   Type          Value
		data:    ---------------------  ------------  ---------------------
		data:    adminUsername          String        MyUserName
		data:    adminPassword          SecureString  undefined
		data:    dnsLabelPrefix    String        MyDomainName
		data:    ubuntuOSVersion        String        14.04.2-LTS
		info:    group deployment show command OK

	>[AZURE.NOTE] 構成が適切でないことがわかった場合、または実行時間の長いデプロイを停止する必要がある場合は、次のコマンドを使用します。
	>
	> `azure group deployment stop testRG testRGDeploy`
	>
	> デプロイ名を指定しない場合、テンプレート ファイル名に基づいてデプロイ名が自動的に作成されます。作成されたデプロイ名は、`azure group create` コマンドの出力の一部として返されます。

	これで、指定したドメイン名を利用し、VM に SSH 接続できます。VM に接続するとき、 `MyDomainName.westus.cloudapp.azure.com` のような、`<domainName>.<region>.cloudapp.azure.com` 形式の完全修飾ドメイン名を使用する必要があります。

5. グループを表示するには、次のコマンドを使用します。

		azure group show testRG

	このコマンドにより、グループ内のリソースに関する情報が返されます。複数のグループがある場合は、`azure group list` コマンドでグループ名の一覧を表示した後、`azure group show` コマンドを使用して特定のグループの詳細を表示します。

テンプレートをコンピューターにダウンロードせず、[GitHub](https://github.com/Azure/azure-quickstart-templates) から直接使用することもできます。その場合、コマンドに **--template-uri** オプションを指定し、テンプレートの azuredeploy.json ファイルに URL を渡します。URL を取得するには、GitHub で _raw_ モードで azuredeploy.json を開き、ブラウザーのアドレス バーに表示される URL をコピーします。次のようにコマンドを使用し、この URL を使用してデプロイを直接作成できます。

	azure group deployment create testRG testRGDeploy --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-linux/azuredeploy.json
必要なテンプレート パラメーターを入力するように求められます。

> [AZURE.NOTE] _raw_ モードで JSON テンプレートを開くことが重要です。ブラウザーのアドレス バーに表示される URL は、通常モードで表示される URL とは異なります。GitHub でファイルを表示するときに_未加工_モードでファイルを開くには、右上にある **[未加工]** をクリックします。

## リソースの操作

テンプレート言語を使用すると、グループ全体の構成の変更を宣言できますが、場合によっては、特定のリソースのみの操作が必要になることがあります。このような場合には、`azure resource` コマンドを使用します。

> [AZURE.NOTE] `list` コマンド以外の `azure resource` コマンドを使用する場合、操作するリソースの API バージョンを `-o` パラメーターによって指定する必要があります。使用する API バージョンが不明な場合は、テンプレート ファイルでリソースの **apiVersion** フィールドを確認してください。

1. グループ内のすべてのリソースの一覧を表示するには、次のコマンドを使用します。

		azure resource list testRG

2. *MyUbuntuVM* と名付けられた VM など、グループ内の個別のリソースを表示するには、次のコマンドを使用します。

		azure resource show testRG MyUbuntuVM Microsoft.Compute/virtualMachines -o "2015-06-15"

	**Microsoft.Compute/virtualMachines** パラメーターに注意してください。このパラメーターは、情報を要求するリソースの種類を示します。前の手順でダウンロードしたテンプレート ファイルを確認すると、テンプレートに記述されている仮想マシンのリソースで、種類の定義にこの同じ値が使用されていることがわかります。

	このコマンドを実行すると、仮想マシンに関する情報が表示されます。

3. リソースの詳細を表示するとき、多くの場合、`--json` パラメーターを使用すると便利です。一部の値が入れ子構造や集合になるので、出力が読みやすくなります。次の例のコマンドを実行すると、**show** コマンドの結果を JSON ドキュメントとして表示した場合の例が示されます。

		azure resource show testRG MyUbuntuVM Microsoft.Compute/virtualMachines -o "2015-06-15" --json

	>[AZURE.NOTE] &gt; 文字を使用して出力をファイルに転送すると、JSON データをファイルに保存できます。次に例を示します。
	>
	> `azure resource show testRG MyUbuntuVM Microsoft.Compute/virtualMachines -o "2015-06-15" --json > myfile.json`

4. 既存のリソースを削除するには、次のようなコマンドを使用します。

		azure resource delete testRG MyUbuntuVM Microsoft.Compute/virtualMachines -o "2015-06-15"

## グループのログの表示

グループに実行された操作に関してログに記録された情報を表示するには、`azure group log show` コマンドを使用します。既定では、グループに対して実行された直前の操作が表示されます。すべての操作を表示するには、任意指定の `--all` パラメーターを使用します。最後のデプロイについて表示するには、`--last-deployment` を使用します。特定のデプロイについて表示するには、`--deployment` とデプロイ名を指定します。次の例では、*testRG* グループで実行されたすべての操作のログが表示されます。

	azure group log show testRG --all
    
## テンプレートとしてのリソース グループのエクスポート

既存のリソース グループがある場合は、そのリソース グループの Resource Manager テンプレートを表示できます。テンプレートのエクスポートには、2 つの利点があります。

1. ソリューションの将来のデプロイを簡単に自動化できます。すべてのインフラストラクチャがテンプレートに定義されているためです。

2. ソリューションを表す JSON を見ることでテンプレートの構文について理解を深めることができます。

Azure CLI を使用して、リソース グループの現在の状態を表すテンプレートをエクスポートするか、特定のデプロイに使用されたテンプレートをダウンロードできます。

* **リソース グループのテンプレートのエクスポート** - これは、リソース グループを変更した後で、現在の状態の JSON 表現を取得する必要があるときに便利です。しかしながら、生成されたテンプレートには、最小数のパラメーターのみが含まれ、変数はありません。テンプレートの大半の値はハードコーディングされています。生成されたテンプレートをデプロイする前に、さまざまな環境に合わせてデプロイをカスタマイズできるように、多くの値をパラメーターに変換しておくと便利です。

    リソース グループのテンプレートをローカル ディレクトリにエクスポートするには、次の例に示すように、`azure group export` コマンドを実行します (ローカル ディレクトリはお使いのオペレーティング システム環境に合わせて置き換えてください)。

        azure group export testRG ~/azure/templates/

* **特定のデプロイのテンプレートのダウンロード** -- これは、リソースのデプロイに使用された実際のテンプレートを表示する必要があるときに便利です。テンプレートには、元のデプロイに定義されていたすべてのパラメーターと変数が含まれます。ただし、組織の誰かがテンプレートに定義されている以外のリソース グループを変更した場合、そのテンプレートはそのリソース グループの現在の状態を表しません。

    特定のデプロイに使用されたテンプレートをローカル ディレクトリにダウンロードするには、`azure group deployment template download` コマンドを実行します。

        azure group deployment template download TestRG testRGDeploy ~/azure/templates/downloads/
 
>[AZURE.NOTE] テンプレートのエクスポート機能はプレビューの段階にあり、現在のところ、一部の種類のリソースでは、テンプレートをエクスポートできません。テンプレートのエクスポートを試行したとき、一部のリソースがエクスポートされなかったというエラーが表示されることがあります。必要に応じて、ダウンロード後、エクスポートされなかったリソースをテンプレート内で手動で定義できます。

## 次のステップ

* Azure リソース マネージャーで Azure PowerShell を使用する方法の詳細については、「[Azure リソース マネージャーでの Azure PowerShell の使用](powershell-azure-resource-manager.md)」を参照してください。
* Azure ポータルから Azure Resource Manager を使用する方法の詳細については、「[Azure ポータルを使用した Azure リソースのデプロイと管理](./azure-portal/resource-group-portal.md)」を参照してください。

<!---HONumber=AcomDC_0525_2016-->