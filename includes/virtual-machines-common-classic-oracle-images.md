


Oracle イメージに基づく Virtual Machines を作成するには、[Azure ポータル](https://portal.azure.com/)にサインインして **[Marketplace]** をクリック後、**[Compute]** をクリックし、検索ボックスに「**Oracle**」と入力します。イメージを選択し、手順に従って Microsoft Azure でイメージを設定します。[Azure ポータル](https://portal.azure.com/)では、Microsoft による Oracle イメージは Windows で、Oracle による Oracle イメージは Oracle Linux で実行されることに注意してください。

![](./media/virtual-machines-common-classic-oracle-images/image1.png)

##Windows ベースの仮想マシン イメージ
Azure の Windows Server 上で実行される、利用可能な Oracle 仮想マシン イメージの一覧を次に示します。これらのイメージは従量課金であり、Oracle のライセンス料にはこれらのイメージの使用が含まれています。独自のライセンスを持ち込んで、Windows または Linux 上で Oracle ソフトウェアを実行することもできます。Azure Virtual Machines の料金体系、ライセンス、仮想マシンのギャラリー イメージの詳細は[ここ](https://azure.microsoft.com/pricing/details/virtual-machines/#oracle-software)にあります。特定の Oracle の詳細価格を参照するには、**[Oracle]** タブをクリックします。

###Oracle データベース仮想マシン イメージ
- Windows Server 2012 の Oracle Database 12c Enterprise Edition
- Windows Server 2012 の Oracle Database 12c Standard Edition
- 基本的なオプションの Oracle Database 12c
- 詳細オプションの Oracle Database 12c
- Windows Server 2008 R2 の Oracle Database 11g R2 Enterprise Edition
- Windows Server 2008 R2 の Oracle Database 11g R2 Standard Edition
- 基本的なオプションのOracle Database 11g R2 EE
- 詳細オプションのOracle Database 11g R2 EE  

###Oracle WebLogic Server 仮想マシン イメージ
- Windows Server 2012 の Oracle WebLogic Server 12c Enterprise Edition
- Windows Server 2012 の Oracle WebLogic Server 12c Standard Edition
- Windows Server 2008 R2 の Oracle WebLogic Server 11g Enterprise Edition
- Windows Server 2008 R2 の Oracle WebLogic Server 11g Standard Edition  

###Oracle データベースおよび Oracle WebLogic Server の仮想マシン イメージ  
- Windows Server 2012 の Oracle Database 12c および WebLogic Server 12c Enterprise Edition
- Windows Server 2012 の Oracle Database 12c および WebLogic Server 12c Standard Edition
- Windows Server 2008 R2 の Oracle Database 11g および WebLogic Server 11g Enterprise Edition
- Windows Server 2008 R2 の Oracle Database 11g および WebLogic Server 11g Standard Edition

### Java 仮想マシン イメージ
-	Windows Server 2012 R2 の JDK 8
-	Windows Server 2012 R2 の JDK 7
-	Windows Server 2012 の JDK 6


##Oracle Linux 仮想マシン イメージ
Azure の Oracle Linux 上で実行される、事前に構成された利用可能な Oracle 仮想マシン イメージの一覧を次に示します。Oracle のライセンス料には、これらの事前構成された仮想マシン イメージの使用は含まれていません。これらのイメージについて、独自に所有するライセンスを使用しなければなりません。独自のライセンスを持ち込んで、Windows または Linux のカスタム Virtual Machines に Oracle ソフトウェアをインストールして実行することもできます。[Azure での Oracle ライセンス](http://www.oracle.com/technetwork/topics/cloud/faq-1963009.html#support)の詳細を以下に説明します。また、[独自のイメージ](../articles/virtual-machines/virtual-machines-windows-classic-createupload-vhd.md)を使用した仮想マシン作成の詳細についても説明します。これについて、また Oracle や他のワークロードを Azure へ移行する別の方法については、「[Windows ベースの仮想マシンを作成するさまざまな方法](../articles/virtual-machines/virtual-machines-windows-creation-choices.md)」を参照してください。

- Oracle Database 12c Enterprise Edition on Oracle Linux
- Oracle Database 12c Standard Edition on Oracle Linux
- Oracle WebLogic Server 12c Enterprise Edition on Oracle Linux
- Oracle Linux 6.4.0.0.0
- Oracle Linux 7.0.0.0.0

##その他のリソース
[Azure Marketplace の新しい一体型の Oracle イメージ](https://msopentech.com/blog/2015/02/19/new-one-oracle-images-azure-marketplace/)

[Oracle 仮想マシン イメージ - 他の考慮事項](#miscellaneous-considerations-for-oracle-virtual-machine-images-new-article)

<!---HONumber=AcomDC_0330_2016------>
