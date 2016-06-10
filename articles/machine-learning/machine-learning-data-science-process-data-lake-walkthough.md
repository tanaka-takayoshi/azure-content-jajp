<properties
	pageTitle="Azure Data Lake のスケーラブルなデータ サイエンス: エンドツーエンド チュートリアル | Microsoft Azure"
	description="Azure Data Lake を使用してデータセットに対してデータ探索と二項分類タスクを行う方法について説明します。"  
	services="machine-learning"
	documentationCenter=""
	authors="bradsev,wguo123"
	manager="paulettm"
	editor="cgronlun" />

<tags
	ms.service="machine-learning"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="05/23/2016"
	ms.author="bradsev;weig"/>


# Azure Data Lake のスケーラブルなデータ サイエンス: エンドツーエンド チュートリアル

このチュートリアルでは、NYC タクシー乗車と料金のデータセットを例にして、Azure Data Lake を使用してデータ探索タスクと二項分類タスクを実行し、料金ごとにチップが支払われるかどうかを予測します。また、データの取得から、モデルのトレーニング、モデルを公開する Web サービスのデプロイまで、[データ サイエンス プロセス](http://aka.ms/datascienceprocess)全体の手順について説明します。

**Azure Data Lake Analytics**

[Microsoft Azure Data Lake](https://azure.microsoft.com/solutions/data-lake/) には、データ科学者が、拡張性が高く、コスト効率がよい方法で、任意のサイズ、形状、および速度のデータを格納し、データ処理、高度な分析、および機械学習モデリングを行うために必要なすべての機能がそろっています。データが実際に処理されたときにのみ、ジョブごとに課金されます。Azure Data Lake Analytics には U-SQL が含まれています。U-SQL は、SQL の宣言型の性質と C# の表現力を兼ね備え、スケーラブルな分散クエリ機能を持つ言語です。読み取り時にスキーマを適用して非構造化データを処理し、カスタム ロジックと UDF を挿入できます。また、規模に応じて実行する方法を細かく制御できる拡張性もあります。U-SQL の背景にある設計理念の詳細については、[Visual Studio ブログの投稿](https://blogs.msdn.microsoft.com/visualstudio/2015/09/28/introducing-u-sql-a-language-that-makes-big-data-processing-easy/)を参照してください。

Data Lake Analytics は、Cortana Analytics Suite の重要な部分でもあり、Azure SQL Data Warehouse、Power BI、Data Factory と連携します。そのため、一式そろったクラウド ビッグ データと高度な分析プラットフォームを利用できます。

このチュートリアルでは、まず、データ サイエンス プロセスを構成する Data Lake Analytics を使用したタスクの実行に必要な前提条件およびリソースと、そのインストール方法について説明します。次に、U-SQL と Azure Machine Learning をこのプロセスを実行する手順と、Python を使用して同様のタスクを実行する方法について説明します。


**U-SQL と Azure Machine Learning**

このチュートリアルでは、Visual Studio を使用して、データセットを処理する U-SQL スクリプトを編集します。ここでは U-SQL スクリプトについて説明します。また、スクリプトは別ファイルで提供されます。プロセスには、データの取り込み、探索、サンプリングが含まれています。次に、U-SQL スクリプトで作成したジョブを Azure ポータルから実行する方法について説明します。Azure Machine Learning Studio の二項分類モデルの構築とデプロイを容易にするために、関連する HDInsight クラスターのデータについて Hive テーブルを作成します。


**Python**

このチュートリアルには、NYC タクシー乗車と料金のデータセットのサンプルに対して、Python を使用してこれらのデータ サイエンス タスクを実行する方法について説明したセクションが含まれています。このプロセスの各手順に関する Python スクリプトを含む Jupyter Notebook を提供します。この Notebook には、ここで概要を説明する二項分類モデルに加え、追加機能のエンジニアリング手順とモデル構築のコードが含まれています。たとえば、多クラス分類や回帰モデリングなどです。回帰タスクでは、チップの他の特徴に基づいてチップの金額を予測します。


**スクリプト**

このチュートリアルでは、基本的な手順の概要のみを説明します。完全な **U-SQL スクリプト**と **Jupyter Notebook** は、[GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/AzureDataLakeWalkthrough) からダウンロードできます。


## 前提条件

以下のトピックを読む前に、次の項目を用意する必要があります。

- Azure サブスクリプション。Azure サブスクリプションがない場合は、[Azure 無料試用版の取得](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)に関するページを参照してください。

- [推奨] Visual Studio 2013 または 2015。いずれかのバージョンがインストールされていない場合、無料の Community エディションを[ここ](https://www.visualstudio.com/visual-studio-homepage-vs.aspx)からダウンロードできます。[Visual Studio] セクションの **[Community 2015 のダウンロード]** ボタンをクリックします。

>[AZURE.NOTE] Visual Studio ではなく、Azure ポータルを使用して Azure Data Lake クエリを送信することもできます。Visual Studio とポータルの両方でこの手順を実行する方法については、このページの「**U-SQL を使用してデータを処理する**」を参照してください。

- Azure Data Lake プレビューにサインアップする

>[AZURE.NOTE] Azure Data Lake Store (ADLS) と Azure Data Lake Analytics (ADLA) はプレビュー期間なので、使用する承認を受ける必要があります。初めて ADLS または ADLA を作成するときに、サインアップするように求められます。サインアップするには、**[プレビューにサインアップ]** をクリックし、契約を読んで **[OK]** をクリックします。たとえば、ADLS のサインアップ ページは次のとおりです。

 ![2](./media/machine-learning-data-science-process-data-lake-walkthough/ADLA-preview-signup.PNG)
 


## Azure Data Lake のデータ サイエンス環境を準備する
このチュートリアルのデータ サイエンス環境を準備するには、次のリソースを作成します。

- Azure Data Lake Store (ADLS) 
- Azure Data Lake Analytics (ADLA)
- Azure BLOB ストレージ アカウント
- Azure Machine Learning Studio アカウント
- Azure Data Lake Tools for Visual Studio (推奨)

このセクションでは、これらの各リソースを作成する手順について説明します。Azure Data Lake Store は別に作成するか、Azure Data Lake Analytics を作成するときに既定のストレージとして作成することができます。同様に、Azure BLOB ストレージ アカウントは別に作成するか、HDInsight Linux クラスターを作成するときに既定のストレージとして作成することができます。これらの各リソースを作成する手順は、以下の別のセクションで説明していますが、この 2 つのストレージ アカウントを別の手順で作成する必要はありません。

### Azure Data Lake Store を作成する

[Azure ポータル](http://portal.azure.com)から ADLS を作成します。詳細については、「[Azure ポータルを使用して、Data Lake Store を使用する HDInsight クラスターを作成する](../data-lake-store/data-lake-store-hdinsight-hadoop-use-portal.md)」を参照してください。クラスター AAD ID は、説明されている **[オプションの構成]** ブレードの **[データソース]** ブレードに設定する必要があります。

 ![3](./media/machine-learning-data-science-process-data-lake-walkthough/create_ADLS.PNG)


### Azure Data Lake Analytics アカウントを作成する
Azure ポータルから [ADLA アカウント](http://portal.azure.com)を作成します。詳細については、「[チュートリアル: Azure ポータルで Azure Data Lake Analytics の使用を開始する](../data-lake-analytics/data-lake-analytics-get-started-portal.md)」を参照してください。

 ![4](./media/machine-learning-data-science-process-data-lake-walkthough/create_ADLA_new.PNG)


### Azure BLOB ストレージ アカウントを作成する
[Azure ポータル](http://portal.azure.com)から Azure BLOB ストレージ アカウントを作成します。詳細については、「[Azure ストレージ アカウントについて](../storage/storage-create-storage-account.md)」の「ストレージ アカウントの作成」セクションを参照してください。
	
 ![5](./media/machine-learning-data-science-process-data-lake-walkthough/Create_Azure_Blob.PNG)


### Azure Machine Learning Studio アカウントをセットアップする
[[Azure Machine Learning]](https://azure.microsoft.com/services/machine-learning/) ページから Azure Machine Learning Studio にサインアップまたはサインインします。**[今すぐご利用ください]** ボタンをクリックし、[Free Workspace] または [Standard Workspace] を選択します。選択すると、Azure ML Studio で実験を作成できるようになります。

### Azure Data Lake Tools をインストールする 
お使いの Visual Studio バージョン用の Azure Data Lake Tools は、[Azure Data Lake Tools for Visual Studio](https://www.microsoft.com/download/details.aspx?id=49504) からインストールします。

 ![10](./media/machine-learning-data-science-process-data-lake-walkthough/install_ADL_tools_VS.PNG)

インストールが正常に完了したら、Visual Studio を開きます。上部のメニューに [Data Lake] タブが表示されます。Azure アカウントにサインインすると、左側のパネルに Azure リソースが表示されます。

 ![11](./media/machine-learning-data-science-process-data-lake-walkthough/install_ADL_tools_VS_done.PNG)


## NYC タクシー乗車データセット
ここで使用したデータセットは、[NYC タクシー乗車データセット](http://www.andresmh.com/nyctaxitrips/)という一般公開されているデータセットです。NYC タクシー乗車データは、約 20 GB の圧縮された CSV ファイル (非圧縮では最大 48 GB) で構成されており、ファイルには 1 億 7300 万以上の個々の乗車と、各乗車に対して支払われた料金が記録されています。各乗車レコードには、乗車と降車の場所と時間、匿名化されたタクシー運転手の (運転) 免許番号、およびメダリオン (タクシーの一意の ID) 番号が含まれています。データには 2013 年のすべての乗車が含まれ、データは月ごとに次の 2 つのデータセットに用意されています。

 - 「trip\_data」の CSV ファイルには、乗車の詳細 (乗客数、乗車地点、降車地点、乗車時間、乗車距離など) が含まれています。いくつかのサンプル レコードを次に示します。

		medallion,hack_license,vendor_id,rate_code,store_and_fwd_flag,pickup_datetime,dropoff_datetime,passenger_count, trip_time_in_secs,trip_distance,pickup_longitude,pickup_latitude,dropoff_longitude,dropoff_latitude
		89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,1,N,2013-01-01 15:11:48,2013-01-01 15:18:10,4,382,1.00,-73.978165,40.757977,-73.989838,40.751171
		0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-06 00:18:35,2013-01-06 00:22:54,1,259,1.50,-74.006683,40.731781,-73.994499,40.75066
		0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-05 18:49:41,2013-01-05 18:54:23,1,282,1.10,-74.004707,40.73777,-74.009834,40.726002
		DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:54:15,2013-01-07 23:58:20,2,244,.70,-73.974602,40.759945,-73.984734,40.759388
		DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:25:03,2013-01-07 23:34:24,1,560,2.10,-73.97625,40.748528,-74.002586,40.747868

 - 「trip\_fare」の CSV ファイルには、各乗車に対して支払われた料金の詳細 (支払いの種類、料金、追加料金と税、チップ、道路などの通行料、および合計支払金額など) が含まれます。いくつかのサンプル レコードを次に示します。

		medallion, hack_license, vendor_id, pickup_datetime, payment_type, fare_amount, surcharge, mta_tax, tip_amount, tolls_amount, total_amount
		89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,2013-01-01 15:11:48,CSH,6.5,0,0.5,0,0,7
		0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-06 00:18:35,CSH,6,0.5,0.5,0,0,7
		0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-05 18:49:41,CSH,5.5,1,0.5,0,0,7
		DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:54:15,CSH,5,0.5,0.5,0,0,6
		DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:25:03,CSH,9.5,0.5,0.5,0,0,10.5

trip\_data と trip\_fare を結合するための一意のキーは、medallion、hack\_license、pickup\_datetime という 3 つのフィールドで構成されています。パブリック Azure ストレージ BLOB から、生の CSV ファイルにアクセスできます。

## U-SQL を使用してデータを処理する

このセクションで説明するデータ処理タスクには、データの取り込み、品質チェック、探索、サンプリングが含まれています。また、乗車テーブルと料金テーブルを結合する方法についても説明します。最後のセクションでは、U-SQL スクリプトで作成したジョブを Azure ポータルから実行する方法について説明します。以下は各セクションのリンクです。

- [データの取り込み: パブリック BLOB からデータを取り込む](#ingest)
- [データ品質チェック](#quality)
- [データの探索](#explore)
- [乗車テーブルと料金テーブルの結合](#join)
- [データのサンプリング](#sample)
- [U-SQL ジョブの実行](#run)

ここでは U-SQL スクリプトについて説明します。また、スクリプトは別ファイルで提供されます。完全な **U-SQL スクリプト**は、[GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/AzureDataLakeWalkthrough) からダウンロードできます。

U-SQL を実行するには、Visual Studio を開き、**[ファイル]、[新規作成]、[プロジェクト]** の順にクリックし、**[U-SQL プロジェクト]** を選択し、名前を付けてフォルダーに保存します。

![12](./media/machine-learning-data-science-process-data-lake-walkthough/create_USQL_project.PNG)

>[AZURE.NOTE] Visual Studio ではなく、Azure ポータルを使用して U-SQL を実行する場合、Azure Data Lake Analytics リソースに移動し、図の手順に従ってクエリを送信します。


![29](./media/machine-learning-data-science-process-data-lake-walkthough/portal_submit_job.PNG)

### <a name="ingest"></a>データの取り込み: パブリック BLOB からデータを取り込む

Azure BLOB のデータの場所は、**wasb://container_name@blob_storage_account_name.blob.core.windows.net/blob_name** と指定して参照します。**Extractors.Csv()** を使用して抽出することができます。wasb アドレスの container_name@blob_storage_account_name について、次のスクリプトのコンテナー名とストレージ アカウント名を置き換えます。ファイル名は同じ形式なので、12 個の乗車ファイルすべてで **trip\_data\_{*}.csv** を使用して読み込むことができます。

	///Read in Trip data
	@trip0 =
	    EXTRACT 
	    medallion string,
	    hack_license string,
	    vendor_id string,
	    rate_code string,
	    store_and_fwd_flag string,
	    pickup_datetime string,
	    dropoff_datetime string,
	    passenger_count string,
	    trip_time_in_secs string,
	    trip_distance string,
	    pickup_longitude string,
	    pickup_latitude string,
	    dropoff_longitude string,
	    dropoff_latitude string
    // This is reading 12 trip data from blob
    FROM "wasb://container_name@blob_storage_account_name.blob.core.windows.net/nyctaxitrip/trip_data_{*}.csv"
    USING Extractors.Csv();

1 行目にヘッダーがあるので、ヘッダーを削除し、列を適切な種類に変更する必要があります。処理したデータは **swebhdfs://data_lake_storage_name.azuredatalakestorage.net/folder_name/file_name**_ を使用して Azure Data Lake ストレージに保存するか、**wasb://container_name@blob_storage_account_name.blob.core.windows.net/blob_name** を使用して Azure BLOB ストレージ アカウントに保存することができます。

	// change data types
	@trip =
	    SELECT 
	    medallion,
	    hack_license,
	    vendor_id,
	    rate_code,
	    store_and_fwd_flag,
	    DateTime.Parse(pickup_datetime) AS pickup_datetime,
	    DateTime.Parse(dropoff_datetime) AS dropoff_datetime,
	    Int32.Parse(passenger_count) AS passenger_count,
	    Double.Parse(trip_time_in_secs) AS trip_time_in_secs,
	    Double.Parse(trip_distance) AS trip_distance,
	    (pickup_longitude==string.Empty ? 0: float.Parse(pickup_longitude)) AS pickup_longitude,
	    (pickup_latitude==string.Empty ? 0: float.Parse(pickup_latitude)) AS pickup_latitude,
	    (dropoff_longitude==string.Empty ? 0: float.Parse(dropoff_longitude)) AS dropoff_longitude,
	    (dropoff_latitude==string.Empty ? 0: float.Parse(dropoff_latitude)) AS dropoff_latitude
    FROM @trip0
    WHERE medallion != "medallion";

	////output data to ADL
	OUTPUT @trip   
	TO "swebhdfs://cdsp.azuredatalakestore.net/nyctaxi_weig/demo_trip.csv"
	USING Outputters.Csv(); 

	////Output data to blob
	OUTPUT @trip   
	TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_trip.csv"
	USING Outputters.Csv();  

同様に、料金データセットを読み込むことができます。Azure Data Lake Store を右クリックし、**[Azure ポータル] の [データ エクスプローラー]** か、Visual Studio の **[ファイル エクスプローラー]** でデータを確認することができます。

 ![13](./media/machine-learning-data-science-process-data-lake-walkthough/data_in_ADL_VS.PNG)

 ![14](./media/machine-learning-data-science-process-data-lake-walkthough/data_in_ADL.PNG)


### <a name="quality"></a>データ品質チェック

乗車テーブルと料金テーブルを読み込んだら、次の方法でデータ品質チェックを実行できます。結果の CSV ファイルは、Azure BLOB Storage または Azure Data Lake Store に出力できます。

メダリオンの数とメダリオンの一意の数を検索します。

	///check the number of medallions and unique number of medallions
	@trip2 =
	    SELECT
	    medallion,
	    vendor_id,
	    pickup_datetime.Month AS pickup_month
	    FROM @trip;
	
	@ex_1 =
	    SELECT
	    pickup_month, 
	    COUNT(medallion) AS cnt_medallion,
	    COUNT(DISTINCT(medallion)) AS unique_medallion
	    FROM @trip2
	    GROUP BY pickup_month;
	    OUTPUT @ex_1   
	TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_1.csv"
	USING Outputters.Csv(); 

乗車回数が 100 回を超えたメダリオンを検索します。

	///find those medallions that had more than 100 trips
	@ex_2 =
	    SELECT medallion,
	           COUNT(medallion) AS cnt_medallion
	    FROM @trip2
	    //where pickup_datetime >= "2013-01-01t00:00:00.0000000" and pickup_datetime <= "2013-04-01t00:00:00.0000000"
	    GROUP BY medallion
	    HAVING COUNT(medallion) > 100;
	    OUTPUT @ex_2   
	TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_2.csv"
	USING Outputters.Csv(); 

pickup\_longitude に関して無効なレコードを検索します。

	///find those invalid records in terms of pickup_longitude
	@ex_3 =
	    SELECT COUNT(medallion) AS cnt_invalid_pickup_longitude
	    FROM @trip
	    WHERE
	    pickup_longitude <- 90 OR pickup_longitude > 90;
	    OUTPUT @ex_3   
	TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_3.csv"
	USING Outputters.Csv(); 

特定の変数について欠落値を検索します。

	//check missing values
	@res =
	    SELECT *,
	           (medallion == null? 1 : 0) AS missing_medallion
	    FROM @trip;
	
	@trip_summary6 =
	    SELECT 
	        vendor_id,
	    SUM(missing_medallion) AS medallion_empty, 
	    COUNT(medallion) AS medallion_total,
	    COUNT(DISTINCT(medallion)) AS medallion_total_unique  
	    FROM @res
	    GROUP BY vendor_id;
	OUTPUT @trip_summary6
	TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_16.csv"
	USING Outputters.Csv();



### <a name="explore"></a>データの探索

データの探索を実行して、データの理解を深めることができます。

チップが払われた乗車と払われなかった乗車の分布を検索します。

	///tipped vs. not tipped distribution
	@tip_or_not =
	    SELECT *,
	           (tip_amount > 0 ? 1: 0) AS tipped
	    FROM @fare;
	
	@ex_4 =
	    SELECT tipped,
	           COUNT(*) AS tip_freq
	    FROM @tip_or_not
	    GROUP BY tipped;
	    OUTPUT @ex_4   
	TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_4.csv"
	USING Outputters.Csv(); 

0、5、10、および 20 ドルというカットオフ値でチップ金額の分布を検索します。

	//tip class/range distribution
	@tip_class =
	    SELECT *,
	           (tip_amount >20? 4: (tip_amount >10? 3:(tip_amount >5 ? 2:(tip_amount > 0 ? 1: 0)))) AS tip_class
	    FROM @fare;
	@ex_5 =
	    SELECT tip_class,
	           COUNT(*) AS tip_freq
	    FROM @tip_class
	    GROUP BY tip_class;
	    OUTPUT @ex_5   
	TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_5.csv"
	USING Outputters.Csv(); 

乗車距離の基本的な統計情報を検索します。

	// find basic statistics for trip_distance
	@trip_summary4 =
	    SELECT 
	        vendor_id,
	        COUNT(*) AS cnt_row,
	        MIN(trip_distance) AS min_trip_distance,
	        MAX(trip_distance) AS max_trip_distance,
	        AVG(trip_distance) AS avg_trip_distance 
	    FROM @trip
	    GROUP BY vendor_id;
	OUTPUT @trip_summary4
	TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_14.csv"
	USING Outputters.Csv();

乗車距離のパーセンタイルを検索します。

	// find percentiles of trip_distance
	@trip_summary3 =
	    SELECT DISTINCT vendor_id AS vendor,
	                    PERCENTILE_DISC(0.25) WITHIN GROUP(ORDER BY trip_distance) OVER(PARTITION BY vendor_id) AS median_trip_distance_disc,
	                    PERCENTILE_DISC(0.5) WITHIN GROUP(ORDER BY trip_distance) OVER(PARTITION BY vendor_id) AS median_trip_distance_disc,
	                    PERCENTILE_DISC(0.75) WITHIN GROUP(ORDER BY trip_distance) OVER(PARTITION BY vendor_id) AS median_trip_distance_disc
	    FROM @trip;
	   // group by vendor_id;
	OUTPUT @trip_summary3
	TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_13.csv"
	USING Outputters.Csv(); 


### <a name="join"></a>乗車テーブルと料金テーブルの結合

乗車テーブルと料金テーブルは、medallion、hack\_license、pickup\_time によって結合できます。

	//join trip and fare table

    @model_data_full =
    SELECT t.*, 
    f.payment_type, f.fare_amount, f.surcharge, f.mta_tax, f.tolls_amount,  f.total_amount, f.tip_amount,
    (f.tip_amount > 0 ? 1: 0) AS tipped,
    (f.tip_amount >20? 4: (f.tip_amount >10? 3:(f.tip_amount >5 ? 2:(f.tip_amount > 0 ? 1: 0)))) AS tip_class
    FROM @trip AS t JOIN  @fare AS f
    ON   (t.medallion == f.medallion AND t.hack_license == f.hack_license AND t.pickup_datetime == f.pickup_datetime)
    WHERE   (pickup_longitude != 0 AND dropoff_longitude != 0 );

	//// output to blob
	OUTPUT @model_data_full   
	TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_7_full_data.csv"
	USING Outputters.Csv(); 

	////output data to ADL
	OUTPUT @model_data_full   
	TO "swebhdfs://cdsp.azuredatalakestore.net/nyctaxi_weig/demo_ex_7_full_data.csv"
	USING Outputters.Csv(); 


各レベルの乗客数について、レコード数、平均チップ金額、チップ金額の分散、チップが支払われた乗車の割合を計算します。

	// contigency table
	@trip_summary8 =
	    SELECT passenger_count,
	           COUNT(*) AS cnt,
	           AVG(tip_amount) AS avg_tip_amount,
	           VAR(tip_amount) AS var_tip_amount,
	           SUM(tipped) AS cnt_tipped,
	           (float)SUM(tipped)/COUNT(*) AS pct_tipped
	    FROM @model_data_full
	    GROUP BY passenger_count;
	    OUTPUT @trip_summary8
	TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_17.csv"
	USING Outputters.Csv();


### <a name="sample"></a>データのサンプリング

まず、結合したテーブルから 0.1 % のデータをランダムに選択します。

	//random select 1/1000 data for modeling purpose
	@addrownumberres_randomsample =
	SELECT *,
	        ROW_NUMBER() OVER() AS rownum
	FROM @model_data_full;
	
	@model_data_random_sample_1_1000 =
	SELECT *
	FROM @addrownumberres_randomsample
	WHERE rownum % 1000 == 0;
	
	OUTPUT @model_data_random_sample_1_1000   
	TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_7_random_1_1000.csv"
	USING Outputters.Csv(); 

次に、バイナリ変数 tip\_class で階層サンプリングを実行します。

	//stratified random select 1/1000 data for modeling purpose
	@addrownumberres_stratifiedsample =
	SELECT *,
	        ROW_NUMBER() OVER(PARTITION BY tip_class) AS rownum
	FROM @model_data_full;
	
	@model_data_stratified_sample_1_1000 =
	SELECT *
	FROM @addrownumberres_stratifiedsample
	WHERE rownum % 1000 == 0;
	//// output to blob
	OUTPUT @model_data_stratified_sample_1_1000   
	TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_9_stratified_1_1000.csv"
	USING Outputters.Csv(); 
	////output data to ADL
	OUTPUT @model_data_stratified_sample_1_1000   
	TO "swebhdfs://cdsp.azuredatalakestore.net/nyctaxi_weig/demo_ex_9_stratified_1_1000.csv"
	USING Outputters.Csv(); 


### <a name="run"></a>U-SQL ジョブの実行

U-SQL スクリプトの編集を完了したら、Azure Data Lake Analytics アカウントを使用してサーバーに送信できます。**[Data Lake]**、**[ジョブの送信]** の順にクリックし、**[Analytics アカウント]** を選択し、**[並列処理]** を選択して、**[送信]** ボタンをクリックします。

 ![15](./media/machine-learning-data-science-process-data-lake-walkthough/submit_USQL.PNG)

ジョブが正常にコンパイルされると、ジョブの状態が Visual Studio に表示され、監視できるようになります。ジョブの実行が完了した後は、ジョブの実行プロセスを再現し、ボトルネックになっている手順を特定してジョブの効率を改善することもできます。また、Azure ポータルを開いて、U-SQL ジョブの状態を確認することもできます。

 ![16](./media/machine-learning-data-science-process-data-lake-walkthough/USQL_running_v2.PNG)


 ![17](./media/machine-learning-data-science-process-data-lake-walkthough/USQL_jobs_portal.PNG)


以上で、Azure BLOB ストレージまたは Azure ポータルで出力ファイルを確認できるようになりました。次の手順のモデリングには、階層サンプル データを使用します。

 ![18](./media/machine-learning-data-science-process-data-lake-walkthough/U-SQL-output-csv.PNG)

 ![19](./media/machine-learning-data-science-process-data-lake-walkthough/U-SQL-output-csv-portal.PNG)

## Azure Machine Learning でモデルを構築してデプロイする
2 つのオプションを例にして、Azure Machine Learning にデータをプルする方法について説明します。最初のオプションでは、Azure BLOB に出力されたサンプリング データ (上記の「**データ サンプリング**」手順のデータ) を使用し、Python を使用してモデルを構築し、Azure Machine Learning にデプロイします。2 つ目のオプションでは、Hive クエリを使用して、Azure Data Lake のデータを直接クエリします。このオプションの場合、新しい HDInsight クラスターを作成するか、Hive テーブルが Azure Data Lake ストレージのデータを指している既存の HDInsight クラスターを使用する必要があります。ここでは、これらの両方のオプションについて説明します。

### オプション 1: Python を使用して機械学習モデルを構築してデプロイする

それでは、Python を使用して機械学習モデルを構築し、デプロイしてみましょう。Python を使用して、Azure BLOB ストレージから処理済みデータ (上記の「**データ サンプリング**」手順で保存されたデータ) を読み取り、モデルを構築することができます。ローカル コンピューターまたは Azure Machine Learning Studio で Jupyter Notebook を作成します。この記事では、モデリングとデプロイの手順についてのみ説明します。Jupyter Notebook には、探索、データの視覚化、特徴エンジニアリング、モデリング、デプロイを行う完全なコードが含まれています。

#### Python ライブラリをインポートする

サンプルの Jupyter Notebook または Python スクリプト ファイルを実行するには、以下の Python パッケージが必要です。AzureML Notebook サービスを使用している場合、これらのパッケージはプレインストールされています。

	import pandas as pd
	from pandas import Series, DataFrame
	import numpy as np
	import matplotlib.pyplot as plt
	from time import time
	import pyodbc
	import os
	from azure.storage.blob import BlobService
	import tables
	import time
	import zipfile
	import random
	import sklearn
	from sklearn.linear_model import LogisticRegression
	from sklearn.cross_validation import train_test_split
	from sklearn import metrics
	from __future__ import division
	from sklearn import linear_model
	from azureml import services


#### BLOB のデータを読み込む

- 接続文字列   

		CONTAINERNAME = 'test1'
		STORAGEACCOUNTNAME = 'XXXXXXXXX'
		STORAGEACCOUNTKEY = 'YYYYYYYYYYYYYYYYYYYYYYYYYYYY'
		BLOBNAME = 'demo_ex_9_stratified_1_1000_copy.csv'
		blob_service = BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)
	
- テキストとして読み込む

		t1 = time.time()
		data = blob_service.get_blob_to_text(CONTAINERNAME,BLOBNAME).split("\n")
		t2 = time.time()
		print(("It takes %s seconds to read in "+BLOBNAME) % (t2 - t1))

 ![29](./media/machine-learning-data-science-process-data-lake-walkthough/python_readin_csv.PNG)
 
- 列名と別の列を追加する

		colnames = ['medallion','hack_license','vendor_id','rate_code','store_and_fwd_flag','pickup_datetime','dropoff_datetime',
		'passenger_count','trip_time_in_secs','trip_distance','pickup_longitude','pickup_latitude','dropoff_longitude','dropoff_latitude',
		'payment_type', 'fare_amount', 'surcharge', 'mta_tax', 'tolls_amount',  'total_amount', 'tip_amount', 'tipped', 'tip_class', 'rownum']
		df1 = pd.DataFrame([sub.split(",") for sub in data], columns = colnames)
	


- 一部の列を数値に変更する

		cols_2_float = ['trip_time_in_secs','pickup_longitude','pickup_latitude','dropoff_longitude','dropoff_latitude',
		'fare_amount', 'surcharge','mta_tax','tolls_amount','total_amount','tip_amount', 'passenger_count','trip_distance'
		,'tipped','tip_class','rownum']
		for col in cols_2_float:
		    df1[col] = df1[col].astype(float)

#### 機械学習モデルを構築する

ここでは、乗車にチップが支払われたかどうかを予測する二項分類モデルを構築します。Jupyter Notebook には、他に多クラス分類モデルと回帰モデルという 2 つのモデルがあります。

- まず、scikit-learn モデルで使用できるダミー変数を作成する必要があります。

		df1_payment_type_dummy = pd.get_dummies(df1['payment_type'], prefix='payment_type_dummy')
		df1_vendor_id_dummy = pd.get_dummies(df1['vendor_id'], prefix='vendor_id_dummy')

- モデリングのデータ フレームを作成する

		cols_to_keep = ['tipped', 'trip_distance', 'passenger_count']
		data = df1[cols_to_keep].join([df1_payment_type_dummy,df1_vendor_id_dummy])
		
		X = data.iloc[:,1:]
		Y = data.tipped

- 60-40 分割のトレーニングとテスト

		X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=0.4, random_state=0)

- トレーニング セットのロジスティック回帰

		model = LogisticRegression()
		logit_fit = model.fit(X_train, Y_train)
		print ('Coefficients: \n', logit_fit.coef_)
		Y_train_pred = logit_fit.predict(X_train)

       ![](./media/machine-learning-data-science-process-data-lake-walkthough/py_logit_coefficient.PNG)

- テスト データ セットにスコアを付ける

		Y_test_pred = logit_fit.predict(X_test)

- 評価メトリックを計算する

		fpr_train, tpr_train, thresholds_train = metrics.roc_curve(Y_train, Y_train_pred)
		print fpr_train, tpr_train, thresholds_train
		
		fpr_test, tpr_test, thresholds_test = metrics.roc_curve(Y_test, Y_test_pred) 
		print fpr_test, tpr_test, thresholds_test
		
		#AUC
		print metrics.auc(fpr_train,tpr_train)
		print metrics.auc(fpr_test,tpr_test)
		
		#Confusion Matrix
		print metrics.confusion_matrix(Y_train,Y_train_pred)
		print metrics.confusion_matrix(Y_test,Y_test_pred)

       ![](./media/machine-learning-data-science-process-data-lake-walkthough/py_logit_evaluation.PNG)


 
#### Web サービス API を構築して Python で使用する

機械学習モデルの構築後に、運用可能な状態にする必要があります。ここでは、二項ロジスティック モデルを例として使用します。ローカル コンピューターの scikit-learn のバージョンが 0.15.1 であることを確認してください。Azure ML Studio サービスを使用している場合、確認する必要はありません。

- Azure ML Studio 設定でワークスペースの資格情報を確認します。Azure Machine Learning Studio で、**[設定]**、**[名前]**、**[承認トークン]** の順にクリックします。 

	![](./media/machine-learning-data-science-process-data-lake-walkthough/workspace_id.PNG)


		workspaceid = 'xxxxxxxxxxxxxxxxxxxxxxxxxxx'
		auth_token = 'xxxxxxxxxxxxxxxxxxxxxxxxxxx'

- Web サービスを作成する

		@services.publish(workspaceid, auth_token) 
		@services.types(trip_distance = float, passenger_count = float, payment_type_dummy_CRD = float, payment_type_dummy_CSH=float, payment_type_dummy_DIS = float, payment_type_dummy_NOC = float, payment_type_dummy_UNK = float, vendor_id_dummy_CMT = float, vendor_id_dummy_VTS = float)
		@services.returns(int) #0, or 1
		def predictNYCTAXI(trip_distance, passenger_count, payment_type_dummy_CRD, payment_type_dummy_CSH,payment_type_dummy_DIS, payment_type_dummy_NOC, payment_type_dummy_UNK, vendor_id_dummy_CMT, vendor_id_dummy_VTS ):
		    inputArray = [trip_distance, passenger_count, payment_type_dummy_CRD, payment_type_dummy_CSH, payment_type_dummy_DIS, payment_type_dummy_NOC, payment_type_dummy_UNK, vendor_id_dummy_CMT, vendor_id_dummy_VTS]
		    return logit_fit.predict(inputArray)

- Web サービスの資格情報を取得する

		url = predictNYCTAXI.service.url
		api_key =  predictNYCTAXI.service.api_key
		
		print url
		print api_key

		@services.service(url, api_key)
		@services.types(trip_distance = float, passenger_count = float, payment_type_dummy_CRD = float, payment_type_dummy_CSH=float,payment_type_dummy_DIS = float, payment_type_dummy_NOC = float, payment_type_dummy_UNK = float, vendor_id_dummy_CMT = float, vendor_id_dummy_VTS = float)
		@services.returns(float)
		def NYCTAXIPredictor(trip_distance, passenger_count, payment_type_dummy_CRD, payment_type_dummy_CSH,payment_type_dummy_DIS, payment_type_dummy_NOC, payment_type_dummy_UNK, vendor_id_dummy_CMT, vendor_id_dummy_VTS ):
		    pass

- Web サービス API を呼び出す。前の手順の後に、5 ～ 10 秒待つ必要があります。

		NYCTAXIPredictor(1,2,1,0,0,0,0,0,1)

       ![](./media/machine-learning-data-science-process-data-lake-walkthough/call_API.PNG)

### オプション 2: Azure Machine Learning で直接モデルを作成してデプロイする

Azure Machine Learning Studio を使用して、Azure Data Lake Store から直接データを読み取り、モデルを作成してデプロイできます。Azure Machine Learning で Azure Data Lake Store のデータを直接読み取るには、Hive テーブルを作成する必要があります。この場合、別の Azure HDInsight クラスターをプロビジョニングし、そこに Azure Data Lake Store を示す Hive テーブルを作成します。次のサブセクションでは、その手順について説明します。

#### HDInsight Linux クラスターを作成する
[Azure ポータル](http://portal.azure.com)から HDInsight クラスター (Linux) を作成します。詳細については、「[Azure ポータルを使用して、Data Lake Store を使用する HDInsight クラスターを作成する](../data-lake-store/data-lake-store-hdinsight-hadoop-use-portal.md)」の「Azure Data Lake Store にアクセスできる HDInsight クラスターを作成する」セクションを参照してください。

 ![6](./media/machine-learning-data-science-process-data-lake-walkthough/create_HDI_cluster.PNG)

#### HDInsight で Hive テーブルを作成する

次に、前の手順で Azure Data Lake Store に格納したデータを使用して、HDInsight クラスターに Hive テーブルを作成します。次の手順では、Azure Machine Learning Studio で Hive テーブルを使用します。このチュートリアルの冒頭で作成した HDInsight クラスターに移動します。**[設定]**、**[プロパティ]**、**[クラスター AAD ID]**、**[ADLS アクセス]** の順にクリックし、読み取り、書き込み、実行の権限がある Azure Data Lake Store アカウントが一覧に追加されます。

 ![20](./media/machine-learning-data-science-process-data-lake-walkthough/HDI_cluster_add_ADLS.PNG)


[設定] ボタンの横にある **[ダッシュボード]** をクリックします。ウィンドウがポップアップ表示されます。ページの右上にある **[Hive ビュー]** をクリックします。**クエリ エディター**が表示されます。

 ![21](./media/machine-learning-data-science-process-data-lake-walkthough/HDI_dashboard.PNG)

 ![22](./media/machine-learning-data-science-process-data-lake-walkthough/Hive_Query_Editor_v2.PNG)


次の Hive スクリプトを貼り付けてテーブルを作成します。データ ソースの場所は、次のように Azure Data Lake Store 参照内にあります。**adl://data_lake_store_name.azuredatalakestore.net:443/folder_name/file_name**

	CREATE EXTERNAL TABLE nyc_stratified_sample
	(
	    medallion string,
	    hack_license string,
	    vendor_id string,
	    rate_code string,
	    store_and_fwd_flag string,
	    pickup_datetime string,
	    dropoff_datetime string,
	    passenger_count string,
	    trip_time_in_secs string,
	    trip_distance string,
	    pickup_longitude string,
	    pickup_latitude string,
	    dropoff_longitude string,
	    dropoff_latitude string,
	  payment_type string,
	  fare_amount string,
	  surcharge string,
	  mta_tax string,
	  tolls_amount string,
	  total_amount string,
	  tip_amount string,
	  tipped string,
	  tip_class string,
	  rownum string
	  )
	ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' lines terminated by '\n'
	LOCATION 'adl://cdsp.azuredatalakestore.net:443/nyctaxi_weig/demo_ex_9_stratified_1_1000_copy.csv';

クエリの実行が完了すると、次のような結果が表示されます。

 ![23](./media/machine-learning-data-science-process-data-lake-walkthough/Hive_Query_results.PNG)



#### Azure Machine Learning Studio でモデルを構築してデプロイする

これで、Azure Machine Learning でのモデルの作成とモデルの配置に進む準備が整いました。二項分類 (チップが支払われたかどうか)、多クラス分類 (tip\_class)、回帰 (tip\_amount) というサンプリング データの予測問題でデータを使用する準備が整いました。ここでは、Azure Machine Learning Studio で二項分類モデルを構築してデプロイする方法について説明します。

1. **リーダー** モジュール (「**データの入力と出力**」セクションで使用可能) を使用して、Azure ML にデータを取得します。詳細については、「[リーダー](https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/) モジュールのリファレンス」ページを参照してください。
2. **Hive クエリ**を、**[プロパティ]** パネルの**データ ソース**として選択します。
3. **Hive データベース クエリ** エディターに次の Hive スクリプトを貼り付けます。

    	select * from nyc_stratified_sample;

4. HDInsight クラスターの URI (Azure ポータルで確認できます)、Hadoop の資格情報、出力データの場所、Azure ストレージ アカウントの名前/キー/コンテナー名を入力します。

 ![24](./media/machine-learning-data-science-process-data-lake-walkthough/reader_module_v3.PNG)

Hive テーブルからデータを読み取る二項分類の実験の例を次の図に示します。

 ![25](./media/machine-learning-data-science-process-data-lake-walkthough/AML_exp.PNG)

実験の作成後に、**[Web サービスのセットアップ]**、**[Predictive Web Service]** の順にクリックします。

 ![26](./media/machine-learning-data-science-process-data-lake-walkthough/AML_exp_deploy.PNG)

自動的に作成されたスコア付け実験を実行し、完了したら、**[Web サービスのデプロイ]** をクリックします。

 ![27](./media/machine-learning-data-science-process-data-lake-walkthough/AML_exp_deploy_web.PNG)

Web サービス ダッシュボードがすぐに表示されます。

 ![28](./media/machine-learning-data-science-process-data-lake-walkthough/AML_web_api.PNG)


## 概要

このチュートリアルで実行した内容のまとめ:

- Azure Data Lake でスケーラブルなエンドツーエンド ソリューションを構築するデータ サイエンス環境を作成する
- U-SQL と Python を使用して、モデル トレーニングによるデータの取得から、Web サービスとしてのモデルのデプロイまで、データ サイエンス プロセスの正規の手順を使用して大規模なパブリック データセットを取得します。

<!---HONumber=AcomDC_0525_2016-->