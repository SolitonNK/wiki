# API

このセクションでは、GUIと "フロントエンド "ウェブサーバの間で使用されるウェブAPIについて説明します。

API の大部分は RESTful です。このルールの例外は検索APIで、検索からのデータの起動と観測に関連するデータ交換と転送の性質上、ウェブソケットを使用します。

## 基本API

* [ログイン](login.md)
* [ユーザー設定](userprefs.md)
* [ユーザーアカウントの制御](account.md)
* [ユーザーグループの制御](groups.md)
* [お知らせ](notifications.md)
* [検索制御](searchctrl.md)
* [検索結果のダウンロード](download.md)
* [検索履歴](searchhistory.md)
* [ロギング](loglevel.md)
* [インジェストエントリー](ingest.md)
* [その他のAPI](misc.md)
* [システム管理](management.md)

## Gravwell内のオブジェクト

ユーザーを作成したり、変更したりすることができる様々な「もの」があります。このセクションでは、それらのAPIをリストアップしています。

* [自動抽出器](extractors.md)
* [ダッシュボード](dashboards.md)
* [Kits](kits.md)
* [マクロ](macros.md)
* [プレイブック](playbooks.md)
* [リソース](resources.md)
* [予定されている検索](scheduledsearches.md)
* [ライブラリー検索](searchlibrary.md)
* [テンプレート](templates.md)
* [ピボット（アクションアブル）](pivots.md)
* [ユーザーファイル](userfiles.md)

## 検索と検索統計

[ウェブソケットを検索](websocket-search.md)

[検索結果に再接続](websocket-search-attach.md)

[レンダラとのインタラクション](websocket-render.md)

## システム統計

また、システム統計は通信にWebsocketを使用しています。これには、一般的なクラスタの健全性を監視するために必要なすべての情報が含まれています。

[システム統計 Websocket](websocket-stats.md)

他のいくつかの統計情報は、REST呼び出し、アクセスすることができます。

[REST 統計 API](stats-json.md)

## APIのテスト

システムには _/api/test_ にあるテスト API が含まれており、ウェブサーバが生きていて機能しているかどうかをテストするために使用することができます。 テスト API は完全に認証されておらず、常に StatusOK 200 と空のボディで応答します。
