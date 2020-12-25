# その他のAPI

いくつかのAPIは、主要なカテゴリにうまく収まらないものがあります。それらはここにリストアップされています。

## 接続テスト

このAPIはバックエンドがHTTPリクエストに応答しているかどうかを検証するためのものです。`API/api/test` の GET は 200 ステータスを返し、Bodyの内容はありません。認証は必要ありません。

## バージョンAPI

バージョン情報を取得するために `/api/version` をGETします。認証は不要です。

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

ウェブサーバは、インデクサが知っているすべてのタグのリストを保持しています。このリストは `/api/tags` のGETリクエストで取得できます。これはタグのリストを返します。

```
["default", "gravwell", "pcap", "windows"]
```

## 検索モジュール一覧

利用可能なすべての検索モジュールのリストとそれぞれの情報を取得するには、`/api/info/searchmodules`でGETしてください。これはモジュール情報構造体のリストを返す。

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

## レンダーモジュール一覧

利用可能なすべてのレンダーモジュールのリストとそれぞれの情報を取得するには、`/api/info/rendermodules`をGETしてください。これはモジュール情報構造体のリストを返します。

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

## GUIの設定

このAPIは、ユーザーインターフェースの基本情報を提供します。APIの`/api/settings`をGETすると、以下のような構造体が返される。

```
{
    "DisableMapTileProxy": false,
    "DistributedWebservers": false,
    "MapTileUrl": "http://localhost:8080/api/maps",
    "MaxFileSize": 8388608,
    "MaxResourceSize": 134217728
}
```

* `DisableMapTileProxy` は、真ならば、Gravwellのプロキシを使わずにOpenStreetMapサーバに直接地図要求を送るようにUIに指示する。
* `MapTileUrl` は、UIが地図タイルを取得する際に用いるURLである。
* `DistributedWebservers` は複数のウェブサーバがデータストアを介して連携している場合にtrueに設定されます。
* `MaxFileSize` は `/api/files` APIにアップロードできる最大のファイルサイズ(バイト単位)である。
* `MaxResourceSize` は最大許容リソースサイズをバイト単位で表したものである。

## スクリプトライブラリ

このAPIを使うと、自動化スクリプトが `require` 関数を使ってgithubリポジトリからライブラリをインポートできるようになります。また、すべてのユーザーのリポジトリをgit pullするエンドポイントもあります。

### ライブラリの取得

このエンドポイントは、おそらくライブラリ関数を介して検索エージェントが使用するためにのみ有用ですが、完全性のために含まれています。与えられたリポジトリからファイルを取得するには、URLにパラメータを指定してGETします。

```
/api/libs?repo=github.com/gravwell/libs&commit=40e98d216bb6e69642df392b255e8edc0f57eb06&path=utils/links.ank
```

"repo"と"commit"の値は省略することができます。"repo"が省略された場合、デフォルトは github.com/gravwell/libs となります。"commit"が省略された場合は、マスターブランチの先端がデフォルトとなります。

### ライブラリの更新

リポジトリのセットは各ユーザごとに管理されています。ユーザは `/api/libs/pull` にGETリクエストを送ることで、自分のリポジトリセットに対して `git pull` を強制的に実行することができます。これには時間がかかるかもしれないことに注意してください。
