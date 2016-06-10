<properties
   pageTitle="Public and private IP addressing (classic) in Azure (Azure でのパブリックおよびプライベート IP アドレス指定 (クラシック)) | Microsoft Azure"
   description="Azure でのパブリックおよびプライベート IP アドレス指定について説明します。"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor="tysonn"
   tags="azure-service-management" />
<tags
   ms.service="virtual-network"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="02/11/2016"
   ms.author="telmos" />

# Azure の IP アドレス (クラシック)
Azure リソースには、他の Azure リソース、オンプレミス ネットワーク、およびインターネットと通信するために IP アドレスを割り当てることができます。Azure で使用できる IP アドレスには、パブリックとプライベートの 2 種類があります。

パブリック IP アドレスは、Azure の公開されたサービスを含め、インターネットとの通信に使用します。

プライベート IP アドレスは、Azure 仮想ネットワーク (VNet)、クラウド サービス、およびオンプレミス ネットワーク (VPN Gateway または ExpressRoute 回線を使用してネットワークを Azure に拡張する場合) 内での通信に使用します。

[AZURE.INCLUDE [azure-arm-classic-important-include](../../includes/learn-about-deployment-models-classic-include.md)] [Resource Manager deployment model](virtual-network-ip-addresses-overview-arm.md)。

## パブリック IP アドレス
パブリック IP アドレスを使用すると、Azure リソースはインターネットのほか、[Azure Redis Cache](https://azure.microsoft.com/services/cache/)、[Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/)、[SQL データベース](../sql-database/sql-database-technical-overview.md)、[Azure ストレージ](../storage/storage-introduction.md)など、Azure の公開されたサービスと通信できます。

パブリック IP アドレスは、次の種類のリソースと関連付けられます。

- クラウド サービス
- IaaS 仮想マシン (VM)
- PaaS ロール インスタンス
- VPN ゲートウェイ
- アプリケーション ゲートウェイ

### 割り当て方法
パブリック IP アドレスを Azure リソースに割り当てる必要がある場合は、リソースが作成された場所内の使用可能なパブリック IP アドレスのプールから*動的に*割り当てられます。この IP アドレスは、リソースが停止したときに解放されます。クラウド サービスの場合、この開放はすべてのロール インスタンスが停止したときに発生します。これは、*静的* (予約済み) IP アドレスを使用することで回避できます (「[Cloud Services](#Cloud-services)」を参照)。

>[AZURE.NOTE] Azure リソースへのパブリック IP アドレスの割り当てに使用する IP アドレス範囲のリストは、「[Azure データセンターの IP アドレス範囲](https://www.microsoft.com/download/details.aspx?id=41653)」で公開されています。

### DNS ホスト名の解決
クラウド サービスまたは IaaS VM を作成する場合は、Azure 内のすべてのリソース間で一意のクラウド サービス DNS 名を指定する必要があります。これにより、Azure で管理される DNS サーバー内で、リソースのパブリック IP アドレスへの *dnsname*.cloudapp.net のマッピングが作成されます。たとえば、クラウド サービス DNS 名 **contoso** を持つクラウド サービスを作成すると、完全修飾ドメイン名 (FQDN) **contoso.cloudapp.net** がクラウド サービスのパブリック IP アドレス (VIP) に解決されます。この FQDN を使用して、Azure 内のパブリック IP アドレスをポイントするカスタム ドメイン CNAME レコードを作成できます。

### クラウド サービス
クラウド サービスには、仮想 IP アドレス (VIP) と呼ばれるパブリック IP アドレスが必ずあります。クラウド サービスのエンドポイントを作成して、VIP 内のさまざまなポートをクラウド サービス内の VM およびロール インスタンス上の内部ポートに関連付けることができます。

クラウド サービスには複数の IaaS VM または PaaS ロール インスタンスを含めることができます。すべて同じクラウド サービス VIP から公開されます。[複数の VIP をクラウド サービスに](../load-balancer/load-balancer-multivip.md)割り当てることもできます。これにより、SSL ベースの Web サイトを含むマルチ テナント環境のようなマルチ VIP シナリオが可能になります。

すべてのロール インスタンスが停止した場合でも、[予約済み IP](virtual-networks-reserved-public-ip.md) と呼ばれる*静的*パブリック IP アドレスを使用することで、クラウド サービスのパブリック IP アドレスが変わらないようにできます。特定の場所で静的 (予約済み) IP リソースを作成し、その場所内の任意のクラウド サービスに割り当てることができます。予約済み IP に対して実際の IP アドレスを指定することはできません。これは、作成された場所内の使用可能な IP アドレスのプールから割り当てられます。この IP アドレスは、明示的に削除するまで解放されません。

静的 (予約済み) パブリック IP アドレスは、通常は次のようなクラウド サービスのシナリオで使用されます。

- エンドユーザーがファイアウォール規則をセットアップする必要がある。
- 外部 DNS 名の解決に依存し、動的 IP が A レコードの更新を必要とする。
- IP ベースのセキュリティ モデルを使用する外部 Web サービスを使用する。
- IP アドレスにリンクされている SSL 証明書を使用する。

>[AZURE.NOTE] 従来の VM を作成するとき、コンテナー *クラウド サービス*が Azure により作成され、仮想 IP アドレス (VIP) が与えられます。ポータル経由で作成すると、クラウド サービス VIP から VM に接続できるように、既定の RDP または SSH *エンドポイント*がポータルにより構成されます。このクラウド サービス VIP は予約できます。VM に接続するために予約された IP アドレスが効率的に与えられます。追加のエンドポイントを構成、追加のポートを開くことができます。

### IaaS VM と PaaS ロール インスタンス
パブリック IP アドレスは、クラウド サービス内の IaaS [VM](../virtual-machines/virtual-machines-linux-about.md) または PaaS ロール インスタンスに直接割り当てることができます。これはインスタンス レベル パブリック IP アドレス ([ILPIP](virtual-networks-instance-level-public-ip.md)) と呼ばれます。このパブリック IP アドレスは動的にのみ割り当てることができます。

>[AZURE.NOTE] これは、IaaS VM または PaaS ロール インスタンスのコンテナーである、クラウド サービスの VIP とは異なります。すべて同じクラウド サービス VIP から公開される、複数の IaaS VM または PaaS ロール インスタンスをクラウド サービスに含めることができるためです。

### VPN ゲートウェイ
[VPN Gateway](../vpn-gateway/vpn-gateway-about-vpngateways.md) は、Azure VNet を他の Azure VNet またはオンプレミス ネットワークに接続するために使用できます。VPN Gateway には、パブリック IP アドレスが*動的に*割り当てられ、リモート ネットワークとの通信が可能になります。

### アプリケーション ゲートウェイ
Azure [Application Gateway](../application-gateway/application-gateway-introduction.md) は、HTTP に基づいてネットワーク トラフィックをルーティングするために Layer7 負荷分散に使用できます。Application Gateway には、パブリック IP アドレスが*動的に*割り当てられます。これは負荷分散された VIP として機能します。

### 概略
次の表は、各リソースの種類と、可能な割り当て方法 (動的または静的) および複数のパブリック IP アドレスの割り当てが可能かどうかを示しています。

|リソース|動的|静的|複数の IP アドレス|
|---|---|---|---|
|クラウド サービス|あり|はい|あり|
|IaaS VM または PaaS ロール インスタンス|あり|なし|なし|
|VPN Gateway|あり|なし|なし|
|Application Gateway|あり|なし|なし|

## プライベート IP アドレス
プライベート IP アドレスを使用すると、Azure リソースは、インターネットが到達可能な IP アドレスを使用せずに、クラウド サービス内の他のリソース、[仮想ネットワーク](virtual-networks-overview.md) (VNet)、あるいはオンプレミス ネットワーク (VPN Gateway または ExpressRoute 回線経由) と通信することができます。

Azure クラシック デプロイ モデルでは、プライベート IP アドレスを Azure の以下のリソースに割り当てることができます。

- IaaS VM と PaaS ロール インスタンス
- 内部ロード バランサー
- Application Gateway

### IaaS VM と PaaS ロール インスタンス
クラシック デプロイ モデルで作成された仮想マシン (VM) は、PaaS ロール インスタンスと同様に、常にクラウド サービス内に配置されます。したがって、プライベート IP アドレスの動作はこれらのリソースに似てます。

クラウド サービスは次の 2 つの方法でデプロイできることに注意してください。

- *スタンドアロン* クラウド サービスとして。この場合、クラウド サービスは仮想ネットワーク内にはありません。
- 仮想ネットワークの一部として。

#### 割り当て方法
*スタンドアロン* クラウド サービスの場合、リソースは、Azure データ センターのプライベート IP アドレス範囲から*動的に*割り当てられたプライベート IP アドレスを取得します。これは、同じクラウド サービス内の他の VM との通信にのみ使用できます。この IP アドレスは、リソースが停止して開始されると変わる可能性があります。

仮想ネットワーク内にデプロイされたクラウド サービスの場合、リソースは、関連付けられているサブネットのアドレス範囲から割り当てられたプライベート IP アドレスを取得します (そのネットワーク構成で指定)。このプライベート IP アドレスは、VNet 内のすべての VM 間の通信に使用できます。

また、VNet 内のクラウド サービスの場合、プライベート IP アドレスは既定で (DHCP を使用して) *動的に*割り当てられます。これは、リソースが停止して開始されると変わる可能性があります。IP アドレスが変わらないようにするには、割り当て方法を*静的*に設定し、対応するアドレスの範囲内の有効な IP アドレスを指定する必要があります。

静的プライベート IP アドレスは、通常は次のものに使用されます。

 - ドメイン コントローラーまたは DNS サーバーとして機能する VM。
 - IP アドレスを使用するファイアウォール規則を必要とする VM。
 - IP アドレスを使用して他のアプリからアクセスされる、サービスを実行している VM。

#### 内部 DNS ホスト名の解決
カスタムの DNS サーバーを明示的に構成しない限り、既定ではすべての Azure VM および PaaS ロール インスタンスが、[Azure で管理される DNS サーバー](virtual-networks-name-resolution-for-vms-and-role-instances.md#azure-provided-name-resolution)で構成されます。これらの DNS サーバーは、同じ VNet またはクラウド サービス内に存在する VM およびロール インスタンスの内部名前解決を可能にします。

VM を作成すると、そのプライベート IP アドレスへのホスト名のマッピングが、Azure で管理される DNS サーバーに追加されます。複数の NIC を備える VM の場合、ホスト名はプライマリ NIC のプライベート IP アドレスにマップされます。ただし、このマッピング情報は、同じクラウド サービスまたは VNet 内のリソースに限定されます。

*スタンドアロン* クラウド サービスの場合は、同じクラウド サービス内のみのすべての VM/ロール インスタンスのホスト名を解決できます。VNet 内のクラウド サービスの場合は、VNet 内のすべての VM/ロール インスタンスのホスト名を解決できます。

### 内部ロード バランサー (ILB) と Application Gateway
[Azure 内部ロード バランサー](../load-balancer/load-balancer-internal-overview.md) (ILB) または [Azure Application Gateway](../application-gateway/application-gateway-introduction.md) の**フロント エンド**構成にプライベート IP アドレスを割り当てることができます。このプライベート IP アドレスは内部エンドポイントとして機能し、その仮想ネットワーク (VNet) 内のリソースおよび VNet に接続されたリモート ネットワークからのみアクセスできます。フロント エンド構成には、動的または静的のどちらかのプライベート IP アドレスを割り当てることができます。複数のプライベート IP アドレスを割り当てて、マルチ VIP シナリオを有効にすることもできます。

### 概略
次の表は、各リソースの種類と、可能な割り当て方法 (動的または静的) および複数のプライベート IP アドレスの割り当てが可能かどうかを示しています。

|リソース|動的|静的|複数の IP アドレス|
|---|---|---|---|
|VM (*スタンドアロン* クラウド サービス内)|あり|はい|あり|
|PaaS ロール インスタンス (*スタンドアロン* クラウド サービス内)|あり|なし|あり|
|VM または PaaS ロール インスタンス (VNet 内)|あり|はい|あり|
|内部ロード バランサーのフロント エンド|あり|はい|あり|
|Application Gateway のフロント エンド|あり|はい|あり|

## 制限

サブスクリプションごとに Azure で IP アドレスを指定する際に課せられる IP の制限を下表に示します。ビジネス上のニーズに基づいて既定の制限を上限まで引き上げるには、[サポートにお問い合わせ](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade)ください。

||既定の制限|上限|
|---|---|---|
|パブリック IP アドレス (動的)|5|サポートにお問い合わせください|
|予約済みパブリック IP アドレス|20|サポートにお問い合わせください|
|デプロイあたりのパブリック VIP (クラウド サービス)|5|サポートにお問い合わせください|
|デプロイあたりのプライベート VIP (ILB) (クラウド サービス)|1|1|

Azure における[ネットワークの制限](azure-subscription-service-limits.md#networking-limits)に関する情報を必ずご確認ください。

## 価格

ほとんどの場合、パブリック IP アドレスは無料です。追加のパブリック IP アドレスまたは静的な IP アドレスを使用する場合は標準の料金が発生します。[パブリック IP の料金体系](https://azure.microsoft.com/pricing/details/ip-addresses/)を必ず把握してください。

## リソース マネージャーとクラシック デプロイの相違点
リソース マネージャーの IP アドレス指定機能とクラシック デプロイ モデルとの比較を次に示します。

||リソース|クラシック|リソース マネージャー|
|---|---|---|---|
|**パブリック IP アドレス**|VM|ILPIP と呼ばれる (動的のみ)|パブリック IP と呼ばれる (動的または静的)|
|||IaaS VM や PaaS ロール インスタンスに割り当てられる|VM の NIC に関連付けられる|
||インターネットに接続するロード バランサー|VIP (動的) または予約済み IP (静的) と呼ばれる|パブリック IP と呼ばれる (動的または静的)|
|||クラウド サービスに割り当てられる|ロード バランサーのフロントエンド構成に関連付けられる|
||||
|**プライベート IP アドレス**|VM|DIP と呼ばれる|プライベート IP アドレスと呼ばれる|
|||IaaS VM や PaaS ロール インスタンスに割り当てられる|VM の NIC に割り当てられる|
||内部ロード バランサー (ILB)|ILB に割り当てられる (動的または静的)|ILB のフロントエンド構成に割り当てられる (動的または静的)|

## 次のステップ
- クラシック ポータルを使用して、[静的プライベート IP アドレスを持つ VM をデプロイ](virtual-networks-static-private-ip-classic-pportal.md)します。

<!---HONumber=AcomDC_0323_2016-->