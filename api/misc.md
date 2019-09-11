# Miscellaneous APIs

一部のAPIはメインカテゴリにうまく適合しません。それらはここにリストされています。

## タグリスト

Webサーバーは、インデクサーに認識されているすべてのタグのリストを管理します。このリストはGETリクエストで取得することができます/api/tags。これはタグのリストを返します。

```
["default", "gravwell", "pcap", "windows"]
```

## 検索モジュール一覧

利用可能なすべての検索モジュールのリストとそれぞれの情報を取得するには、/api/info/searchmodules/でGETを実行します。 これはモジュール情報構造体のリストを返すでしょう：

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

利用可能なすべてのレンダーモジュールのリストと各モジュールに関する情報を取得するには、/api/info/rendermodules/でGETを実行します。 これはモジュール情報構造体のリストを返すでしょう：

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