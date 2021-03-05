# その他のAPI

一部のAPIは、メインのカテゴリにうまく適合しません。 それらはここにリストされています。

## 接続テスト

このAPIは、バックエンドがHTTPリクエストに応答していることを検証するためのものです。`/api/test`のGETは、本文の内容がない200ステータスを返す必要があります。認証は必要ありません。

## バージョンAPI

`/api/version`でGETを実行して、バージョン情報を取得します。認証は必要ありません。

```
{
    "API": {
        "Major": 0,
        "Minor": 1
    },
    "Build": {
        "BuildDate": "2020-05-04T00:00:00Z",
        "BuildID": "6c48dd4c",
        "GUIBuildID": "b5c8cd58",
        "Major": 4,
        "Minor": 0,
        "Point": 0
    }
}
```

## タグリスト

Webサーバーは、インデクサーが認識しているすべてのタグのリストを維持します。 このリストは、`/api/tags`のGETリクエストで取得できます。 これにより、タグのリストが返されます：

```
["default", "gravwell", "pcap", "windows"]
```

## 検索モジュールリスト

利用可能なすべての検索モジュールのリストと各モジュールに関する情報を取得するには、`/api/info/searchmodules`でGETを実行します。 これにより、モジュール情報構造のリストが返されます：

```
[
    {
        "Collapsing": true,
        "Examples": [
            "min by src",
            "min by someKey"
        ],
        "FrontendOnly": false,
        "Info": "No information available",
        "Name": "min",
        "Sorting": true
    },
    {
        "Collapsing": true,
        "Examples": [
            "unique",
            "unique chuck",
            "unique chuck,testa"
        ],
        "FrontendOnly": false,
        "Info": "No information available",
        "Name": "unique",
        "Sorting": false
    },
[...]
    {
        "Collapsing": false,
        "Examples": [
            "alias src dst"
        ],
        "FrontendOnly": false,
        "Info": "Alias enumerated values",
        "Name": "alias",
        "Sorting": false
    },
    {
        "Collapsing": true,
        "Examples": [
            "count",
            "count by chuck",
            "count by src",
            "count by someKey"
        ],
        "FrontendOnly": false,
        "Info": "No information available",
        "Name": "count",
        "Sorting": true
    }
]
```

## レンダリングモジュールリスト

使用可能なすべてのレンダーモジュールのリストと各モジュールに関する情報を取得するには、`/api/info/rendermodules`でGETを実行します。 これにより、モジュール情報構造のリストが返されます：

```
[
    {
        "Description": "A raw entry storage system, it can store and handle anything.",
        "Examples": [
            "raw"
        ],
        "Name": "raw",
        "SortRequired": false
    },
    {
        "Description": "A chart storage system system.\n\t       Chart looks for numeric types, storing them.\n\t       Requested entries will be a set of types with column names.",
        "Examples": [
            "chart"
        ],
        "Name": "chart",
        "SortRequired": false
    },
[...]
    {
        "Description": "A point mapping system that supports condensing and geofencing",
        "Examples": [],
        "Name": "point2point",
        "SortRequired": false
    }
]
```

## GUI設定

このAPIは、ユーザーインターフェイスの基本情報を提供します。`/api/settings`でGETを実行すると、次のような構造が返されます：

```
{
  "DisableMapTileProxy": false,
  "DistributedWebservers": false,
  "MapTileUrl": "http://localhost:8080/api/maps",
  "MaxFileSize": 8388608,
  "MaxResourceSize": 134217728,
  "ServerTime": "2020-11-30T11:50:29.478092519-08:00",
  "ServerTimezone": "PST",
  "ServerTimezoneOffset": -28800
}

```

* `DisableMapTileProxy`は、trueの場合、Gravwellプロキシを使用するのではなく、マップリクエストをOpenStreetMapサーバーに直接送信する必要があることをUIに通知します。
* `MapTileUrl`は、UIがマップタイルをフェッチするために使用する必要があるURLです。
* データストアを介して調整する複数のWebサーバーがある場合、`DistributedWebservers`はtrueに設定されます。
* `MaxFileSize`は、`/api/files` APIにアップロードできる最大許容ファイルサイズ（バイト単位）です。
* `MaxResourceSize`は、バイト単位の最大許容リソースサイズです。
* `ServerTime`はウェブサーバーの現在の時刻です。
* `ServerTimezone`はウェブサーバーのタイムゾーンです。
* `ServerTimezoneOffset`は、UTCからの秒単位のWebサーバーのタイムゾーンオフセットです。

## スクリプトライブラリ

このAPIを使用すると、自動化スクリプトで `require`関数を使用してgithubリポジトリからライブラリをインポートできます。すべてのユーザーのリポジトリでgitプルをトリガーするエンドポイントもあります。

### ライブラリの取得

このエンドポイントは、検索エージェントがライブラリ関数を介して使用する場合にのみ役立つ可能性がありますが、完全を期すために含まれています。特定のリポジトリからファイルをフェッチするには、URLにパラメータを指定してGETを実行します。例：

```
/api/libs?repo=github.com/gravwell/libs&commit=40e98d216bb6e69642df392b255e8edc0f57eb06&path=utils/links.ank
```

「repo」と「commit」の値は省略できます。「repo」を省略すると、デフォルトで github.com/gravwell/libs になります。「commit」を省略すると、デフォルトでマスターブランチの先端になります。

### ライブラリの更新

リポジトリのセットは、ユーザーごとに維持されます。 ユーザーは、GETリクエストを `/api/libs/pull`に送信することで、自分のリポジトリセットに`gitpull`を強制することができます。 これには時間がかかる場合があることに注意してください。