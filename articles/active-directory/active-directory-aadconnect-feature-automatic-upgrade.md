<properties
   pageTitle="Azure AD Connect: 自動アップグレード | Microsoft Azure"
   description="このトピックでは、Azure AD Connect 同期の組み込みの自動アップグレード機能について説明します。"
   services="active-directory"
   documentationCenter=""
   authors="AndKjell"
   manager="StevenPo"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="05/19/2016"
   ms.author="andkjell"/>

# Azure AD Connect: 自動アップグレード
この機能は、ビルド 1.1.105.0 (2016 年 2 月リリース) で導入されました。

## 概要
Azure AD Connect のインストールを常に最新の状態に保つことは、**自動アップグレード**機能によって、これまでよりも簡単になりました。この機能は、高速インストールと DirSync のアップグレード用に既定で有効になっています。インストールは新しいバージョンのリリース時に自動的にアップグレードされます。

自動アップグレードは、次の場合に既定で有効です。

- 簡単設定インストールと DirSync のアップグレード。
- SQL Express LocalDB を使用している (簡単設定で常に使用されます)。SQL Express を使用した DirSync でも、LocalDB が使用されます。
- AD アカウントが、簡単設定と DirSync によって作成される既定の MSOL\_ アカウントである場合。
- メタバース内のオブジェクト数が 100,000 未満。

自動アップグレードの現在の状態は、PowerShell コマンドレット `Get-ADSyncAutoUpgrade` で表示できます。次のような状態があります。

State | コメント
---- | ----
有効 | 自動アップグレードが有効です。
Suspended | システムによる設定だけが可能です。システムは自動アップグレードを受け付けることができなくなりました。
無効 | 自動アップグレードが無効です。

`Set-ADSyncAutoUpgrade` を使用して、**有効**と**無効**を切り替えることができます。システムだけが、状態を**保留**に設定することができます。

自動アップグレードでは、アップグレード インフラストラクチャに Azure AD Connect Health を使用しています。自動アップグレードを正しく動作させるには、「[Office 365 URL および IP アドレス範囲](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2)」に記載されているように、**Azure AD Connect Health** 用にプロキシ サーバーで URL を開いておく必要があります。

**Synchronization Service Manager** UI がサーバーで実行されている場合は、UI が閉じられるまで、アップグレードが中断されます。

## トラブルシューティング
インストール済みの Connect が想定されているとおりに自動的にアップグレードしない場合は、次の手順を実行して問題の原因を特定してください。

注意点として、自動アップグレードは新しいバージョンがリリースされた当日に試行されるわけではありません。アップグレードを試行する前に意図的にランダムな時間をおくため、アップグレードがすぐに実行されなくても心配する必要はありません。

問題が生じていると思われる場合は、最初に `Get-ADSyncAutoUpgrade` を実行して、自動アップグレードが有効になっていることを確認してください。

イベント ビューアーを起動し、**[アプリケーション]** イベントログを確認します。ソースとして **[Azure AD Connect Upgrade]** (Azure AD Connect のアップグレード) を選択し、イベント ID 範囲に「**300-399**」を指定したイベントログ フィルターを追加します。![自動アップグレードのイベントログ フィルター](./media/active-directory-aadconnect-feature-automatic-upgrade/eventlogfilter.png)

これで、自動アップグレードの状態に関連するイベントログが表示されます。![自動アップグレードのイベントログ フィルター](./media/active-directory-aadconnect-feature-automatic-upgrade/eventlogresult.png)

結果コードには、状態についての概要を含むプレフィックスが付加されています。

結果コードのプレフィックス | 説明
--- | ---
成功 | 正常にアップグレードしました。
UpgradeAborted | 一時的な状況により、アップグレードが停止しました。後で再試行され、正常終了することが予想されます。
UpgradeNotSupported | システム内に、自動アップグレードをブロックしている構成があります。状態が変更中であるかどうかを確認するためにもう一度試行されますが、システムを手動でアップグレードする必要があると考えられます。

特によく表示されるメッセージの一覧を次に示します。これですべてではありませんが、結果メッセージを見ると、何が問題となっているかが把握しやすくなります。

結果メッセージ | 説明
--- | ---
**UpgradeAborted** |
UpgradeAbortedCouldNotSetUpgradeMarker | レジストリに書き込むことができません。
UpgradeAbortedInsufficientDatabasePermissions | 組み込みの Administrators グループにデータベースへのアクセス許可がありません。これを解決するには、最新バージョンの Azure AD Connect に手動でアップグレードしてください。
UpgradeAbortedInsufficientDiskSpace | アップグレードをサポートするのに十分なディスク領域がありません。
UpgradeAbortedSecurityGroupsNotPresent | 同期エンジンで使用されるすべてのセキュリティ グループが見つからず、解決できません。
UpgradeAbortedServiceCanNotBeStarted | NT サービス **Microsoft Azure AD Sync** を起動できませんでした。
UpgradeAbortedServiceCanNotBeStopped | NT サービス **Microsoft Azure AD Sync** を停止できませんでした。
UpgradeAbortedServiceIsNotRunning | NT サービス **Microsoft Azure AD Sync** が実行されていません。
UpgradeAbortedSyncCycleDisabled | [スケジューラ](active-directory-aadconnectsync-feature-scheduler.md)の SyncCycle オプションが無効になっています。
UpgradeAbortedSyncExeInUse | サーバーで [Sychronization Service Manager UI](active-directory-aadconnectsync-service-manager-ui.md) が開いています。
UpgradeAbortedSyncOrConfigurationInProgress | インストール ウィザードが実行されているか、同期がスケジューラ以外の場所でスケジュールされました。
**UpgradeNotSupported** |
UpgradeNotSupportedCustomizedSyncRules | ユーザーが構成に独自のカスタム ルールを追加しました。
UpgradeNotSupportedDeviceWritebackEnabled | ユーザーが[デバイスの書き戻し](active-directory-aadconnect-feature-device-writeback.md)機能を有効にしました。
UpgradeNotSupportedGroupWritebackEnabled | ユーザーが[グループの書き戻し](active-directory-aadconnect-feature-preview.md#group-writeback)機能を有効にしました。
UpgradeNotSupportedInvalidPersistedState | インストールが簡単設定でも DirSync のアップグレードでもありません。
UpgradeNotSupportedMetaverseSizeExceeeded | メタバース内のオブジェクトが 100,000 を超えています。
UpgradeNotSupportedMultiForestSetup | 現在、複数のフォレストに接続しています。高速セットアップで接続するフォレストは 1 つのみです。
UpgradeNotSupportedNonLocalDbInstall | SQL Server Express LocalDB データベースが使用されていません。
UpgradeNotSupportedNonMsolAccount | [AD コネクタ アカウント](active-directory-aadconnect-accounts-permissions.md#active-directory-account)は、既定の MSOL\_ アカウントではなくなりました。
UpgradeNotSupportedStagingModeEnabled | サーバーが[ステージング モード](active-directory-aadconnectsync-operations.md#staging-mode)に設定されています。
UpgradeNotSupportedUserWritebackEnabled | ユーザーが[ユーザーの書き戻し](active-directory-aadconnect-feature-preview.md#user-writeback)機能を有効にしました。

## 次のステップ
「[オンプレミス ID と Azure Active Directory の統合](active-directory-aadconnect.md)」をご覧ください。

<!---HONumber=AcomDC_0525_2016-->