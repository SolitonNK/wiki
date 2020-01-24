# 検索ライブラリ

/api/libraryにあるREST API

検索ライブラリAPIは、保存された検索クエリを保存および取得するために使用されます。 検索ライブラリは、名前、メモ、タグなど、日々の運用に役立つ可能性のあるものを含む貴重なクエリを作成するのに便利なシステムです。

検索ライブラリは許可されたAPIです。つまり、各ユーザーは検索ライブラリを所有し、オプションでグループと共有できます。 各ライブラリエントリにはグローバルフラグもあります。つまり、すべてのユーザーがライブラリエントリを読み取ることができます。 管理者のみがグローバルフラグを設定できます。

ライブラリエントリを削除できるのは所有者のみです。別のユーザーがグループメンバーシップを介してライブラリエントリにアクセスできる場合でも、エントリを削除することはできません。

## 管理操作

管理者は、他のすべてのユーザーと同じ方法で検索ライブラリを操作します。 管理者がライブラリAPIへの管理アクセスを必要とする場合、所有者以外のライブラリエントリをリスト、削除、または変更するかどうかにかかわらず、リクエストに `admin` フラグを追加する必要があります。

たとえば、 `/api/library`でGETリクエストを実行すると、呼び出し元ユーザーのライブラリエントリ（adminまたはnot）のみが返されます。  `/api/library?admin=true`で同じGETリクエストを実行すると、すべてのユーザーのすべてのライブラリエントリが返されます。 管理者以外のユーザーの場合、adminフラグは無視されます。

## 基本的なAPIの概要

ライブラリAPIは`/api/library`をルートとし、次のリクエストメソッドに応答します。

| 方法 | 説明 | 管理者呼び出しをサポート |
| ------ | ----------- | -------------------- |
| GET    | 利用可能なライブラリエントリのリストを取得する| TRUE |
| POST   | 新しいライブラリエントリを追加する | FALSE |
| PUT    | 既存のライブラリエントリを更新する | TRUE |
| DELETE | 既存のライブラリエントリを削除する | TRUE |

`DELETE`および`PUT`メソッドでは、特定のライブラリエントリのGUIDをURLに追加する必要があります。 たとえば、`5f72d51e-d641-11e9-9f54-efea47f6014a` a`PUT`のGUIDでエントリを更新するには、`/api/library/5f72d51e-d641-11e9-9f54-efea47f6014a`に対して`PUT`リクエストが発行されます リクエストの本文にエンコードされたエントリ構造。

`GET`メソッドはオプションでGUIDを追加して特定のライブラリエントリを要求できます。GUIDが存在しない場合、GETメソッドは利用可能なすべてのエントリのリストを返します。 ユーザーがGUIDで指定された特定のエントリにアクセスできない場合、Webサーバーは403の応答を返します。

ライブラリエントリの構造は次のとおりです。
```
struct {
	ThingUUID GUID
	UID         int32
	GIDS        []int32
	Global      boolean
	Name        string
	Description string
	Query       string
	Labels      []string
	Metadata    RawObject
}
```

構造体のメンバーは次のとおりです。

| メンバー     | 説明                   | 空の場合は省略 |
| ----------- | ------------------------------- | ---------------- |
| ThingUUID   | グローバルに一意の識別子      |                  |
| UID         | 所有者システムのユーザーID          |                  |
| GIDs        | エントリが共有されるグループIDのリスト | X |
| Global      | エントリがグローバルに読み取り可能かどうかを示すブール値 | |
| Name        | クエリの人間が読める名前 | |
| Description | 質問の人読み取り可能説明 | |
| Query       | クエリ文字列 | |
| Labels      | クエリの分類に使用される人間が読めるラベルのリスト | X |
| Metadata    | 任意のパラメーターストレージに使用される不透明なJSON blob。有効なJSONであるが、特定の構造はない | X |

以下は、UID 1のユーザーが所有し、3つのグループで共有されるエントリの例です。 UIDとGIDの値は、ユーザーAPIを使用してユーザー名とグループ名にマッピングし直す必要があります。

```
{
	"ThingUUID": "69755a85-d5b1-11e9-89c2-0242ac130005",
	"UID": 1,
	"GIDs": [1, 3, 5],
	"Global": false,
	"Name": "syslog counting query",
	"Description": "A simple chart that shows total syslog over time",
	"Query": "tag=syslog syslog Appname | stats count by Appname | chart count by Appname",
	"Labels": [
		"syslog",
		"chart",
		"trending"
	],
	"Metadata": {}
}

```


## APIの相互作用の例

このセクションには、検索ライブラリAPIエンドポイントとの相互作用の例が含まれています。 これらの例は、`-debug`フラグを使用してGravwell CLIを使用して生成されました。

### 新しいエントリを作成する

リクエスト：
```
POST /api/library
{
	"GIDs": [1, 2],
	"Global": false,
	"Name": "netflow agg",
	"Description": "Total traffic using netflow data",
	"Query": "tag=netflow netflow Bytes | stats sum(Bytes) as TotalTraffic | chart TotalTraffic",
	"Labels": [
		"netflow",
		"traffic",
		"aggs"
	]
}
```

応答：
```
{
	"ThingUUID": "c9169d15-d643-11e9-99d3-0242ac130005",
	"UID": 1,
	"GIDs": [1, 2],
	"Global": false,
	"Name": "netflow agg",
	"Description": "Total traffic using netflow data",
	"Query": "tag=netflow netflow Bytes | stats sum(Bytes) as TotalTraffic | chart TotalTraffic",
	"Labels": [
		"netflow",
		"traffic",
		"aggs"
	]
}

```
### エントリの取得

リクエスト：

```
GET http://172.19.0.5:80/api/library
```

応答：
```
[
	{
		"ThingUUID": "0b5a66cb-d642-11e9-931c-0242ac130005",
		"UID": 1,
		"Global": false,
		"Name": "netflow agg",
		"Description": "Total traffic using netflow data",
		"Query": "tag=netflow netflow Bytes | stats sum(Bytes) as TotalTraffic | chart TotalTraffic",
		"Labels": [
			"netflow",
			"traffic",
			"aggs"
		],
		"Metadata": {
			"value": 1,
			"extra": "some extra field value"
		}
	},
	{
		"ThingUUID": "69755a85-d5b1-11e9-89c2-0242ac130005",
		"UID": 1,
		"Global": false,
		"Name": "test2",
		"Description": "testing second",
		"Query": "tag=foo grep bar",
		"Labels": [
			"foo",
			"bar",
			"baz"
		],
	}
]
```

### 特定のエントリを要求する

リクエスト：

```
GET http://172.19.0.5:80/api/library/0b5a66cb-d642-11e9-931c-0242ac130005
```

応答：
```
{
	"ThingUUID": "0b5a66cb-d642-11e9-931c-0242ac130005",
	"UID": 1,
	"Global": false,
	"Name": "netflow agg",
	"Description": "Total traffic using netflow data",
	"Query": "tag=netflow netflow Bytes | stats sum(Bytes) as TotalTraffic | chart TotalTraffic",
	"Labels": [
		"netflow",
		"traffic",
		"aggs"
	],
	"Metadata": {
		"value": 1,
		"extra": "some extra field value"
	}
}
```

### エントリーの更新

リクエスト：
```
PUT /api/library/69755a85-d5b1-11e9-89c2-0242ac130005
{
	"ThingUUID": "69755a85-d5b1-11e9-89c2-0242ac130005",
	"Global": false,
	"Name": "SyslogAgg",
	"Description": "Updated Syslog aggregate",
	"Query": "tag=syslog length | stats sum(length) | chart sum",
	"Labels": [
		"syslog",
		"agg",
		"totaldata"
	],
	"Metadata": {}
}
```

応答：
```
{
	"ThingUUID": "69755a85-d5b1-11e9-89c2-0242ac130005",
	"UID": 1,
	"Global": false,
	"Name": "SyslogAgg",
	"Description": "Updated Syslog aggregate",
	"Query": "tag=syslog length | stats sum(length) | chart sum",
	"Labels": [
		"syslog",
		"agg",
		"totaldata"
	]
}
```

### エントリを削除する

リクエスト:
```
DELETE /api/library/69755a85-d5b1-11e9-89c2-0242ac130005
```

### 管理者によるエントリの削除

非管理者が管理フラグを追加すると、Webサーバーはフラグを無視します。非管理者が指定されたエントリの所有者である場合、アクション（この場合はDELETE）は引き続き機能します。 非管理者ユーザーがエントリを所有していない場合、Webサーバーは403 StatusForbiddenで応答します。

リクエスト:
```
DELETE /api/library/69755a85-d5b1-11e9-89c2-0242ac130005?admin=true
```
