# Web API

このセクションでは、GUIと「フロントエンド」Webサーバーとの間で使用されるWeb APIについて説明します。

APIの大部分はRESTfulです。この規則の例外は、検索からのデータの起動と監視に関わるデータ交換と転送の性質により、WebSocketを使用する検索APIです。

## 基本

* [Login](login.md)
* [User Preferences](userprefs.md)
* [Account controls](account.md)
* [Dashboards](dashboards.md)
* [Notifications](notifications.md)
* [Search Controls](searchctrl.md)
* [Downloading Search Results](download.md)
* [Search History](searchhistory.md)
* [Search Library](searchlibrary.md)
* [Logging](loglevel.md)
* [Resources](resources.md)
* [Scheduled Searches](scheduledsearches.md)
* [Ingesting Entries](ingest.md)
* [Macros](macros.md)
* [Auto-extractors](extractors.md)
* [Miscellaneous APIs](misc.md)
* [Templates and Pivots](templates.md)
* [User Files](userfiles.md)
* [System Management](management.md)

## 検索と検索統計のAPI

[Search Websocket](websocket-search.md)

[Reattaching to Searches](websocket-search-attach.md)

[Interacting with Renderers](websocket-render.md)

## システム統計

システム統計は、通信にWebソケットも使用します。これには、一般的なクラスタの状態を監視するために必要なすべての情報が含まれています。

[System Stats Websocket](websocket-stats.md)

他のいくつかの統計は、REST呼び出しを介してアクセスできます。

[REST Stats API](stats-json.md)

## テストAPI

システムには/ api / testにあるテストAPIが含まれています。これを使用して、Webサーバーが稼働中で機能しているかどうかをテストできます。テストAPIは完全に認証されておらず、常にStatusOK 200と空の本文で応答します。
