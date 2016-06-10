<properties
 pageTitle="Excel と SOA 用の HPC Pack クラスター | Microsoft Azure"
 description="Azure で HPC Pack クラスターを開始して大規模な Excel と SOA ワークロードを実行する"
 services="virtual-machines-windows"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""
 tags="azure-resource-manager,hpc-pack"/>

<tags
 ms.service="virtual-machines-windows"
 ms.devlang="na"
 ms.topic="article"
 ms.tgt_pltfrm="vm-windows"
 ms.workload="big-compute"
 ms.date="02/19/2016"
 ms.author="danlep"/>

# Azure で HPC Pack クラスターを開始して Excel と SOA ワークロードを実行する

この記事では、Azure クイックスタート テンプレートまたは必要に応じて Azure PowerShell デプロイ スクリプトを使用して Azure インフラストラクチャ サービス (IaaS) に Microsoft HPC Pack クラスターをデプロイする方法を示します。HPC Pack で Microsoft Excel またはサービス指向アーキテクチャ (SOA) のワークロードを実行するように設計されている Azure Marketplace VM イメージを使用します。クラスターを使用して、オンプレミスのクライアント コンピューターから簡単な Excel HPC サービスおよび SOA サービスを実行できます。Excel の HPC サービスには、Excel ブックのオフロードと Excel ユーザー定義関数、または UDF が含まれます。

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-both-include.md)]

これから作成する HPC Pack クラスターの概要は次の図のようになります。

![Excel ワークロードを実行するノードを含む HPC クラスター][scenario]

## 前提条件

*   **クライアント コンピューター** - Azure PowerShell のクラスター デプロイ スクリプト (このデプロイ方法を選択する場合) を実行したり、Excel および SOA のサンプル ジョブをクラスターに送信したりするには、Windows ベースのクライアント コンピューターが必要です。

*   **Azure サブスクリプション** - Azure サブスクリプションがない場合は、[無料アカウント](https://azure.microsoft.com/pricing/free-trial/)を数分で作成することができます。

*   **コア クォータ** - 場合によっては、コアのクォータを増やす必要があります。特に、マルチコア VM サイズのクラスター ノードをいくつかデプロイする場合に必要になる可能性があります。Azure クイックスタート テンプレートを使用する場合、リソース マネージャーのコア クォータは Azure リージョンごとであることに注意してください。特定のリージョンのクォータを増やす必要がある可能性があります。「[Azure サブスクリプションとサービスの制限、クォータ、制約](../azure-subscription-service-limits.md)」を参照してください。クォータを増やすには、[オンライン カスタマー サポートに申請](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/) (無料) してください。

*   **Microsoft Office ライセンス** - Microsoft Excel を含む Marketplace HPC Pack VM イメージを使用してコンピューティング ノードをデプロイすると、30 日間の評価バージョンの Microsoft Excel Professional Plus 2013 がコンピューティング ノードにインストールされます。評価期間の終了後もワークロードを引き続き実行するには、有効な Microsoft Office ライセンスを使用して Excel をアクティブ化する必要があります。この記事で後述する [Excel のアクティブ化](#excel-activation)を参照してください。


## 手順 1.Azure で HPC Pack クラスターをセットアップする

クラスターをセットアップする 2 つの方法を説明します。1 つめは Azure クイックスタート テンプレートと Azure ポータルを使用する方法で、2 つめは Azure PowerShell デプロイ スクリプトを使用する方法です。


### クイックスタート テンプレートを使用する
Azure クイックスタート テンプレートを使用すると、Azure ポータルで HPC Pack クラスターをすばやく簡単にデプロイできます。プレビュー ポータルでテンプレートを開くと表示される簡単な UI で、クラスターの設定を入力します。手順は次のようになります。

>[AZURE.TIP]必要に応じて、Excel ワークロードと似たクラスターを作成できる [Azure Marketplace テンプレート](https://portal.azure.com/?feature.relex=*%2CHubsExtension#create/microsofthpc.newclusterexcelcn)を使用します。この場合、以下の手順とは一部異なります。

1.  [GitHub の HPC クラスター テンプレートの作成に関するページ](https://github.com/Azure/azure-quickstart-templates/tree/master/create-hpc-cluster)を参照します。必要な場合は、テンプレートとソース コードに関する情報を確認します。

2.  Azure ポータルで **[Azure にデプロイ]** をクリックして、テンプレートによるデプロイを開始します。

    ![テンプレートを Azure にデプロイする][github]

3.  ポータルで以下の手順に従って、HPC クラスター テンプレートのパラメーターを入力します。

    a.**[パラメーター]** ページで、テンプレート パラメーターの値を入力します (各設定の隣のアイコンをクリックするとヘルプ情報が表示されます)。 次の画面に示されているのはサンプルの値です。この例では、1 つのヘッド ノードと 2 つのコンピューティング ノードで構成される *hpc01* という名前の新しい HPC Pack クラスターが、*hpc.local* ドメインに作成されます。コンピューティング ノードは、Microsoft Excel を含む HPC Pack VM イメージから作成されます。

    ![パラメーターを入力する][parameters]

    >[AZURE.NOTE]ヘッド ノードの VM は、Windows Server 2012 R2 上の HPC Pack 2012 R2 の[最新の Marketplace イメージ](https://azure.microsoft.com/marketplace/partners/microsoft/hpcpack2012r2onwindowsserver2012r2/)から自動的に作成されます。現時点では、イメージは HPC Pack 2012 R2 Update 3 に基づいています。
    >
    >コンピューティング ノードの VM は、選択したコンピューティング ノード ファミリの最新のイメージから作成されます。Microsoft Excel Professional Plus 2013 の評価版を含む最新の HPC Pack コンピューティング ノード イメージの **ComputeNodeWithExcel** オプションを選択します。一般的な SOA セッション用または Excel UDF オフロード用にクラスターをデプロイする場合は、**ComputeNode** オプションを選択します (Excel はインストールされません)。

    b.サブスクリプションを選択します。

    c.クラスターの新しいリソース グループを作成します (*hpc01RG* など)。

    d.リソース グループの場所を選択します (米国中部など)。

    e.**[法律条項]** ページで、条項を確認します。同意する場合は、**[作成]** をクリックします。テンプレートの値の設定が完了したら、**[作成]** をクリックします。

4.  デプロイが完了したら (通常約 30 分かかります)、クラスターのヘッド ノードからクラスターの証明書ファイルをエクスポートします。後の手順でこのパブリック証明書をクライアント コンピューターにインポートし、セキュリティで保護された HTTP バインディングのサーバー側認証を提供します。

    a.Azure ポータルからリモート デスクトップでヘッド ノードに接続します。

     ![ヘッド ノードに接続する][connect]

    b.標準的な手順で証明書マネージャーを使用して、秘密キーを含まないヘッド ノード証明書 (Cert:\\LocalMachine\\My の下にあります) をエクスポートします。この例では、*CN = hpc01.eastus.cloudapp.azure.com* をエクスポートします。

    ![証明書をエクスポートする][cert]

### HPC Pack IaaS デプロイ スクリプトを使用する

HPC Pack IaaS デプロイ スクリプトは、HPC Pack クラスターをデプロイするためのもう 1 つの汎用性の高い方法です。クラシック デプロイ モデルでクラスターを作成しながら、テンプレートに Azure リソース マネージャー デプロイ モデルを使用します。また、スクリプトは Azure Global サービスまたは Azure China サービスのサブスクリプションと互換性があります。

**追加の前提条件**

* **Azure PowerShell** - クライアント コンピューターに [Azure PowerShell をインストールして構成します](../powershell-install-configure.md) (バージョン 0.8.10 以降)。

* **HPC Pack IaaS デプロイ スクリプト** - [Microsoft Download Center](https://www.microsoft.com/download/details.aspx?id=44949) から最新版のスクリプトをダウンロードし、解凍します。`New-HPCIaaSCluster.ps1 –Version` を実行して、スクリプトのバージョンを確認します。この記事はバージョン 4.5.0 以降のスクリプトに基づきます。

**構成ファイルを作成する**

 HPC Pack IaaS デプロイ スクリプトは HPC クラスターのインフラストラクチャについて記載された XML 構成ファイルを入力として使用します。Microsoft Excel を含むコンピューティング ノード イメージから作成された 1 個のヘッド ノードと 18 個のコンピューティング ノードで構成されるクラスターをデプロイするには、次のサンプル構成ファイルを実際の環境の値に置き換えます。構成ファイルの詳細については、スクリプト フォルダーにある Manual.rtf ファイルおよび「[Create an HPC cluster with the HPC Pack IaaS deployment script (HPC Pack IaaS デプロイ スクリプトを使用した HPC クラスターの作成)](virtual-machines-windows-classic-hpcpack-cluster-powershell-script.md)」参照してください。

```
<?xml version="1.0" encoding="utf-8"?>
<IaaSClusterConfig>
  <Subscription>
    <SubscriptionName>MySubscription</SubscriptionName>
    <StorageAccount>hpc01</StorageAccount>
  </Subscription>
  <Location>West US</Location>
  <VNet>
    <VNetName>hpc-vnet01</VNetName>
    <SubnetName>Subnet-1</SubnetName>
  </VNet>
  <Domain>
    <DCOption>NewDC</DCOption>
    <DomainFQDN>hpc.local</DomainFQDN>
    <DomainController>
      <VMName>HPCExcelDC01</VMName>
      <ServiceName>HPCExcelDC01</ServiceName>
      <VMSize>Medium</VMSize>
    </DomainController>
  </Domain>
   <Database>
    <DBOption>LocalDB</DBOption>
  </Database>
  <HeadNode>
    <VMName>HPCExcelHN01</VMName>
    <ServiceName>HPCExcelHN01</ServiceName>
    <VMSize>Large</VMSize>
    <EnableRESTAPI/>
    <EnableWebPortal/>
    <PostConfigScript>C:\tests\PostConfig.ps1</PostConfigScript>
  </HeadNode>
  <ComputeNodes>
    <VMNamePattern>HPCExcelCN%00%</VMNamePattern>
    <ServiceName>HPCExcelCN01</ServiceName>
    <VMSize>Medium</VMSize>
    <NodeCount>18</NodeCount>
    <ImageName>HPCPack2012R2_ComputeNodeWithExcel</ImageName>
  </ComputeNodes>
</IaaSClusterConfig>
```

**構成ファイルに関する注意事項**

* ヘッド ノードの **VMName** は、**ServiceName** と正確に**一致している必要があります**。一致していないと、SOA ジョブの実行に失敗します。

* ヘッド ノード証明書が生成されてエクスポートされるように、**EnableWebPortal** を必ず指定してください。

* Azure ストレージ接続文字列、ヘッド ノードからのコンピューティング ノード ロールの削除、デプロイ後の全ノードのオンライン化など、ヘッド ノードに特定の設定を構成する構成後 PowerShell スクリプト PostConfig.ps1 をファイルに指定します。スクリプトのサンプルを次に示します。

```
    # add the HPC Pack powershell cmdlets
        Add-PSSnapin Microsoft.HPC

    # set the Azure storage connection string for the cluster
        Set-HpcClusterProperty -AzureStorageConnectionString 'DefaultEndpointsProtocol=https;AccountName=<yourstorageaccountname>;AccountKey=<yourstorageaccountkey>'

    # remove the compute node role for head node to make sure the Excel workbook won’t run on head node
        Get-HpcNode -GroupName HeadNodes | Set-HpcNodeState -State offline | Set-HpcNode -Role BrokerNode

    # total number of nodes in the deployment including the head node and compute nodes, which should match the number specified in the XML configuration file
        $TotalNumOfNodes = 19

        $ErrorActionPreference = 'SilentlyContinue'

    # bring nodes online when they are deployed until all nodes are online
        while ($true)
        {
          Get-HpcNode -State Offline | Set-HpcNodeState -State Online -Confirm:$false
          $OnlineNodes = @(Get-HpcNode -State Online)
          if ($OnlineNodes.Count -eq $TotalNumOfNodes)
          {
             break
          }
          sleep 60
        }
```

**スクリプトを実行する**

1.  管理者としてクライアント コンピューターで PowerShell を開きます。

2.  ディレクトリをスクリプト フォルダー (この例では、E:\\IaaSClusterScript) に変更します。

    ```
    cd E:\IaaSClusterScript
    ```
    
3.  下のコマンドを実行し、HPC Pack クラスターをデプロイします。この例では、構成ファイルは E:\\HPCDemoConfig.xml にあるものと想定しています。

    ```
    .\New-HpcIaaSCluster.ps1 –ConfigFile E:\HPCDemoConfig.xml –AdminUserName MyAdminName
    ```

HPC Pack デプロイ スクリプトの実行には少し時間がかかります。スクリプトでの処理の 1 つとして、クラスター証明書がエクスポートされてダウンロードされ、クライアント コンピューター上の現在のユーザーの Documents フォルダーに保存されます。スクリプトでは、次のようなメッセージが生成されます。次の手順では、適切な証明書ストアに証明書をインポートします。
    
    You have enabled REST API or web portal on HPC Pack head node. Please import the following certificate in the Trusted Root Certification Authorities certificate store on the computer where you are submitting job or accessing the HPC web portal:
    C:\Users\hpcuser\Documents\HPCWebComponent_HPCExcelHN004_20150707162011.cer

## 手順 2.Excel ブックをオフロードし、オンプレミスのクライアントから UDF を実行する

### Excel のアクティブ化

運用ワークロードに ComputeNodeWithExcel VM イメージを使用する場合、有効な Microsoft Office ライセンス キーを使用してコンピューティング ノードの Excel をアクティブ化する必要があります。アクティブ化しないと、評価版の Excel は 30 日間で期限が切れ、実行中の Excel ワークロードは COMException (0x800AC472) で失敗するようになります。この例外が発生する場合、ヘッド ノードにログオンし、HPC Cluster Manager を使用してすべての Excel コンピューティング ノードで clusrun `%ProgramFiles(x86)%\Microsoft Office\Office15\OSPPREARM.exe` を実行します。
    
その結果、Excel の評価期間が 30 日間延長されます。この処理は最大 2 回実行できます。その後は、有効な Office ライセンス キーを入力する必要があります。

この VM イメージにインストールされる Office Professional Plus 2013 は、汎用ボリューム ライセンス キー (GVLK) を使用したボリューム エディションです。アクティブ化には、キー管理サービス (KMS)/Active Directory ベースのアクティブ化 (AD-BA) またはマルチ ライセンス認証キー (MAK) を使用する必要があります。KMS/AD-BA を使用するには、Microsoft Office 2013 ボリューム ライセンス パックを使用して、既存の KMS サーバーを使用するか、(ヘッド ノードなどに) 新しいサーバーを設定します。次に、インターネットまたは電話で KMS ホスト キーをアクティブ化します。次に、clusrun `ospp.vbs` を実行して KMS サーバーとポートを設定し、すべての Excel コンピューティング ノードで Office をアクティブ化します。MAK を使用するには、まず clusrun `ospp.vbs` を実行してキーを入力し、インターネットまたは電話ですべての Excel コンピューティング ノードをアクティブ化します。

>[AZURE.NOTE]この VM イメージには、Office Professional Plus 2013 の市販のプロダクト キーを使用できません。この Office Professional Plus 2013 ボリューム エディション以外の Office または Excel エディションの有効なキーとインストール メディアがある場合、このボリューム エディションをアンインストールし、所有しているエディションをインストールすることもできます。再インストールした Excel コンピューティング ノードは、カスタマイズされた VM イメージとしてキャプチャされ、大規模なデプロイで使用されます。

### Excel ブックをオフロードする

以下の手順に従って、Azure の HPC Pack クラスターで実行するように Excel ブックをオフロードします。そのためには、Excel 2010 または 2013 がクライアント コンピューターに既にインストールされている必要があります。

1. 手順 1 のいずれかの方法を使用して、Excel コンピューティング ノード イメージを含む HPC Pack クラスターをデプロイします。クラスター証明書ファイル (.cer) およびクラスターのユーザー名とパスワードを取得します。

2. クライアント コンピューターで、クラスター証明書を Cert:\\CurrentUser\\Root にインポートします。

3. Excel がインストールされていることを確認します。次のような内容の Excel.exe.config ファイルを作成し、クライアント コンピューター上の Excel.exe と同じフォルダーに保存します。これにより、HPC Pack 2012 R2 の Excel COM アドインが正常に読み込まれます。

    ```
    <?xml version="1.0"?>
    <configuration>
        <startup useLegacyV2RuntimeActivationPolicy="true">
            <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.0"/>
        </startup>
    </configuration>
    ```
    
4.	[HPC Pack 2012 R2 Update 3 インストール](http://www.microsoft.com/download/details.aspx?id=49922)の完全版をダウンロードして HPC Pack クライアントをインストールするか、または [HPC Pack 2012 R2 Update 3 クライアント ユーティリティ](https://www.microsoft.com/download/details.aspx?id=49923)とコンピューターに適した Visual C++ 2010 再頒布可能パッケージをダウンロードしてインストールします ([x64](http://www.microsoft.com/download/details.aspx?id=14632)、[x86](https://www.microsoft.com/download/details.aspx?id=5555))。

5.	この例では、[ここ](https://www.microsoft.com/ja-JP/download/details.aspx?id=2939)からダウンロードできる ConvertiblePricing\_Complete.xlsb という名前のサンプル Excel ブックを使用します。

6.	Excel ブックを D:\\Excel\\Run などの作業フォルダーにコピーします。

7.	Excel ブックを開きます。**[開発]** リボンで **[COM アドイン]** をクリックし、次の画面のように、HPC Pack Excel COM アドインが正常に読み込まれていることを確認します。

    ![HPC Pack 用の Excel アドイン][addin]

8.	VBA マクロ HPCControlMacros を Excel で編集し、次のスクリプトで示されているようにコメント行を変更します。実際の環境に合わせて適切な値に置き換えます。

    ![HPC Pack 用の Excel マクロ][macro]

    ```
    'Private Const HPC_ClusterScheduler = "HEADNODE_NAME"
    Private Const HPC_ClusterScheduler = "hpc01.eastus.cloudapp.azure.com"

    'Private Const HPC_NetworkShare = "\\PATH\TO\SHARE\DIRECTORY"
    Private Const HPC_DependFiles = "D:\Excel\Upload\ConvertiblePricing_Complete.xlsb=ConvertiblePricing_Complete.xlsb"

    'HPCExcelClient.Initialize ActiveWorkbook
    HPCExcelClient.Initialize ActiveWorkbook, HPC_DependFiles

    'HPCWorkbookPath = HPC_NetworkShare & Application.PathSeparator & ActiveWorkbook.name
    HPCWorkbookPath = "ConvertiblePricing_Complete.xlsb"

    'HPCExcelClient.OpenSession headNode:=HPC_ClusterScheduler, remoteWorkbookPath:=HPCWorkbookPath
    HPCExcelClient.OpenSession headNode:=HPC_ClusterScheduler, remoteWorkbookPath:=HPCWorkbookPath, UserName:="hpc\azureuser", Password:="<YourPassword>"
```

9.	VBA マクロの HPC\_DependsFiles 定数で指定されている D:\\Excel\\Upload などの upload ディレクトリに、Excel ブックをコピーします。

10.	ワークシートの **[Cluster]** ボタンをクリックし、Azure IaaS クラスターでブックを実行します。

### Excel UDF を実行する

Excel の UDF を実行するには、前記の手順 1 ～ 3 に従ってクライアント コンピューターを設定します。Excel UDF の場合、Excel アプリケーションがコンピューティング ノードにインストールされている必要はないので、手順 1 では、Excel を含むコンピューティング ノード イメージではなく、通常のコンピューティング ノード イメージを選択できます。

>[AZURE.NOTE] Excel 2010 および 2013 クラスター コネクタのダイアログ ボックスには 34 文字の制限があります。完全なクラスター名が長い場合は (例: hpcexcelhn01.southeastasia.cloudapp.azure.com)、ダイアログ ボックスに収まりません。回避策は、長いクラスター名の値に *CCP\_IAASHN* などのマシン全体の変数を設定し、*%CCP\_IAASHN%* をダイアログ ボックスにクラスター ヘッドのノード名として入力します。

クラスターが正常にデプロイされた後、引き続き以下の手順に従って、サンプルの組み込み Excel UDF を実行します。Excel UDF をカスタマイズした場合は、[リソース](http://social.technet.microsoft.com/wiki/contents/articles/1198.windows-hpc-and-microsoft-excel-resources-for-building-cluster-ready-workbooks.aspx)を参考にして、XLL を作成し、IaaS クラスターにそれをデプロイしてください。

1.	新しい Excel ブックを開きます。**[開発]** リボンで **[アドイン]** をクリックします。次に、ダイアログ ボックスで **[参照]** をクリックし、%CCP\_HOME%Bin\\XLL32 フォルダーに移動して、サンプルの ClusterUDF32.xll を選択します。ClusterUDF32 がクライアント コンピューターに存在しない場合は、ヘッド ノードの %CCP\_HOME%Bin\\XLL32 フォルダーからコピーすることができます。

    ![UDF を選択する][udf]

2.	**[ファイル]**、**[オプション]**、**[詳細]** の順にクリックします。**[数式]** の **[コンピューティング クラスターでユーザー定義の XLL 関数を実行できるようにする]** をオンにします。**[オプション]** をクリックし、**[クラスター ヘッド ノード名]** に完全なクラスター名を入力します (説明したように、この入力ボックスには 34 文字の制限があり、長いクラスター名が入らない可能性があります。ここでは、長いクラスター名にマシン全体の変数を使用できます)。

    ![UDF を構成する][options]

3.	値が =XllGetComputerNameC() であるセルをクリックし、Enter キーを押して、IaaS クラスターで UDF の計算を実行します。この関数は、UDF が実行しているコンピューティング ノードの名前を取得するだけです。初めて実行したときは、IaaS クラスターに接続するためのユーザー名とパスワードの入力を求める資格情報ダイアログ ボックスが表示されます。

    ![UDF を実行する][run]

    計算するセルが多い場合は、Alt + Shift + Ctrl + F9 キーを押してすべてのセルに対して計算を実行します。

## 手順 3.オンプレミスのクライアントから SOA ワークロードを実行する

一般的な SOA アプリケーションを HPC Pack IaaS クラスターで実行するには、まず、手順 1 のいずれかの方法に従い、汎用コンピューティング ノード イメージを使用して IaaS クラスターをデプロイします (コンピューティング ノードでは Excel が必要ないため)。次に、以下の手順に従います。

1. クラスター証明書を取得した後、クライアント コンピューターで Cert:\\CurrentUser\\Root にインポートします。

2. SOA クライアント アプリケーションを作成して実行できるように、[HPC Pack 2012 R2 Update 3 SDK](http://www.microsoft.com/download/details.aspx?id=49921) および [HPC Pack 2012 R2 Update 3 クライアント ユーティリティ](https://www.microsoft.com/download/details.aspx?id=49923)をインストールします。

3. HelloWorldR2 [サンプル コード](https://www.microsoft.com/download/details.aspx?id=41633)をダウンロードします。Visual Studio 2010 または 2012 で HelloWorldR2.sln を開きます。

4. 最初に EchoService プロジェクトをビルドし、オンプレミスのクラスターにデプロイするのと同じ方法で、IaaS クラスターにサービスをデプロイします。詳細な手順については、HelloWordR2 の Readme.doc を参照してください。以下で説明するように HellWorldR2 および他のプロジェクトを変更してビルドし、オンプレミスのクライアント コンピューターから Azure IaaS クラスター上で実行する SOA クライアント アプリケーションを生成します。

### Azure ストレージ キューありで Http バインディングを使用する

Azure ストレージ キューで Http バインディングを使用するには、サンプル コードにいくつかの変更を加えます。

* クラスター名を更新します。

    ```
// Before
const string headnode = "[headnode]";
// After e.g.
const string headnode = "hpc01.eastus.cloudapp.azure.com";
or
const string headnode = "hpc01.cloudapp.net";
```

* 必要に応じて、SessionStartInfo の既定の TransportScheme を使用するか、または Http に明示的に設定します。

```
    info.TransportScheme = TransportScheme.Http;
```

* BrokerClient に既定のバインディングを使用します。

    ```
// Before
using (BrokerClient<IService1> client = new BrokerClient<IService1>(session, binding))
// After
using (BrokerClient<IService1> client = new BrokerClient<IService1>(session))
```

    または、basicHttpBinding の使用を明示的に設定します。

    ```
BasicHttpBinding binding = new BasicHttpBinding(BasicHttpSecurityMode.TransportWithMessageCredential);
binding.Security.Message.ClientCredentialType = BasicHttpMessageCredentialType.UserName;    binding.Security.Transport.ClientCredentialType = HttpClientCredentialType.None;
```

* 必要に応じて、SessionStartInfo で UseAzureQueue フラグを true に設定します。これを設定しないと、クラスター名に Azure ドメイン サフィックスが含まれ、TransportScheme が Http の場合は、既定で true に設定されます。

    ```
    info.UseAzureQueue = true;
```

###Azure ストレージ キューなしで Http バインディングを使用する

そのためには、SessionStartInfo で UseAzureQueue フラグを明示的に false に設定します。

```
    info.UseAzureQueue = false;
```

### NetTcp バインディングを使用する

NetTcp バインディングを使用するための構成は、オンプレミスのクラスターに接続する場合と似ています。ヘッド ノード VM でいくつかのエンドポイントを開く必要があります。クラスターを作成する HPC Pack IaaS デプロイ スクリプトを使用した場合、Azure クラシック ポータルで次の手順を実行します。


1. VM を停止します。

2. セッション用、ブローカー用、ブローカー ワーカー用、Data Services 用に、それぞれ TCP ポート 9090、9087、9091、9094 を追加します。

    ![エンドポイントを構成する][endpoint]

3. VM を起動します。

SOA クライアント アプリケーションでは、IaaS クラスターの完全な名前にヘッド名を変更する以外の変更は不要です。

## 次のステップ

* HPC Pack での Excel ワークロードの実行に関する詳細については、[これらのリソース](http://social.technet.microsoft.com/wiki/contents/articles/1198.windows-hpc-and-microsoft-excel-resources-for-building-cluster-ready-workbooks.aspx)を参照してください。

* HPC Pack での SOA サービスのデプロイと管理について詳しくは、「[サービス](https://technet.microsoft.com/library/ff919412.aspx)」を参照してください。

<!--Image references-->
[scenario]: ./media/virtual-machines-windows-excel-cluster-hpcpack/scenario.png
[github]: ./media/virtual-machines-windows-excel-cluster-hpcpack/github.png
[template]: ./media/virtual-machines-windows-excel-cluster-hpcpack/template.png
[parameters]: ./media/virtual-machines-windows-excel-cluster-hpcpack/parameters.png
[create]: ./media/virtual-machines-windows-excel-cluster-hpcpack/create.png
[connect]: ./media/virtual-machines-windows-excel-cluster-hpcpack/connect.png
[cert]: ./media/virtual-machines-windows-excel-cluster-hpcpack/cert.png
[addin]: ./media/virtual-machines-windows-excel-cluster-hpcpack/addin.png
[macro]: ./media/virtual-machines-windows-excel-cluster-hpcpack/macro.png
[options]: ./media/virtual-machines-windows-excel-cluster-hpcpack/options.png
[run]: ./media/virtual-machines-windows-excel-cluster-hpcpack/run.png
[endpoint]: ./media/virtual-machines-windows-excel-cluster-hpcpack/endpoint.png
[udf]: ./media/virtual-machines-windows-excel-cluster-hpcpack/udf.png

<!---HONumber=AcomDC_0323_2016-->