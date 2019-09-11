# Kits Web API

このAPIはGravwellキットの作成、インストール、削除を実装します。 キットには、特定の問題に対するすぐに使えるソリューションを提供するためにローカルシステムにインストールされる他のコンポーネントが含まれています。 キットには以下を含めることができます。

* Resources
* Scheduled searches
* Dashboards
* Auto-extractor definitions
* Templates
* Pivots
* User files

特定のキットには、ビルド時に指定された次の属性もあります。

* ID：このキットの一意の識別子。 Androidの命名規則に従うことをお勧めします。 「com.example.my-kit」。
* Name：キットのわかりやすい名前。 「私のキット」。
* Description: キットの説明。
* Version: キットの整数バージョン。

## キットの作成

キットは、以下に定義するように、KitBuildRequest構造を含むPOSTリクエストを `/api/kit/build`に送信することで構築されます。

```
type KitBuildRequest struct {
	ID                string
	Name              string
	Description       string
	Version           uint
	Dashboards        []uint64    
	Templates         []uuid.UUID 
	Pivots            []uuid.UUID 
	Files             []uuid.UUID 
	Resources         []string    
	ScheduledSearches []int32     
	Macros            []uint64    
	Extractors        []string    
}
```

ID、名前、説明、およびバージョンのフィールドは必須ですが、テンプレート/ピボット/ダッシュボードなどの配列はオプションです。 たとえば、2つのダッシュボード、ピボット、リソース、およびスケジュールされた検索を含むキットを作成するリクエストは次のとおりです。

```
{
	"ID": "io.gravwell.test",
	"Name": "test-gravwell",
	"Description": "Test Gravwell kit",
	"Version": 1,
	"Dashboards": [
		7,
		10
	],
	"Pivots": [
		"ae9f2598-598f-4859-a3d4-832a512b6104"
	],
	"Resources": [
		"84270dbd-1905-418e-b756-834c15661a54"
	],
	"ScheduledSearches": [
		1439174790
	]
}
```

重要：テンプレート、ピボット、およびユーザーファイルに指定するUUIDは、アイテムのリストでも報告される* ThingUUID *フィールドではなく、それらの構造に関連付けられた* GUID *である必要があります。

システムは、新しく構築されたキットを説明する構造で応答します。

```
{
	"UUID": "2f5e485a-2739-475b-810d-de4f80ae5f52",
	"Size": 8268288,
	"UID": 1
}
```

このキットは、 `/api/kit/build/<uuid>`でGETを実行することでダウンロードできます。 上記の応答があれば、 `/api/kit/build/2f5e485a-2739-475b-810d-de4f80ae5f52`からキットを取得します

## キットをアップロードする

キットをインストールする前に、まずWebサーバーにアップロードする必要があります。 キットはPOSTリクエストによって `/api/kit`にアップロードされます。 リクエストにはマルチパートフォームが含まれている必要があります。 ローカルシステムからファイルをアップロードするには、キットファイルを含む `file`という名前のフォームにファイルフィールドを追加します。 HTTPサーバーなどのリモートシステムからファイルをアップロードするには、キットのURLを含む「remote」という名前のフィールドを追加します。

## キットのリスト

`/api/kit`に対するGETリクエストは、すべての既知のキットのリストを返します。 システムに1つのキットがアップロードされているが、まだインストールされていない場合の結果を示す例を次に示します。

```
[
	{
		"UUID": "549c0805-a693-40bd-abb5-bfb29fc98ef1",
		"UID": 7,
		"GID": 0,
		"ID": "io.gravwell.test",
		"Name": "test-gravwell",
		"Description": "Test Gravwell kit",
		"Version": 1,
		"Installed": false,
		"Signed": false,
		"AdminRequired": false,
		"Items": [
			{
				"Name": "84270dbd-1905-418e-b756-834c15661a54",
				"Type": "resource",
				"AdditionalInfo": {
					"VersionNumber": 1,
					"ResourceName": "maxmind_asn",
					"Description": "ASN database",
					"Size": 6196221
				}
			},
			{
				"Name": "55c81086",
				"Type": "scheduled search",
				"AdditionalInfo": {
					"Name": "count",
					"Description": "count all entries",
					"Schedule": "* * * * *",
					"Duration": -3600,
					"Script": "var time = import(\"time\")\n\naddSelfTargetedNotification(7, \"hello\", \"/#/search/486574780\", time.Now().Add(30 * time.Second))"
				}
			},
			{
				"Name": "a",
				"Type": "dashboard",
				"AdditionalInfo": {
					"UUID": "5567707c-8508-4250-9121-0d1a9d5ebe32",
					"Name": "Foo",
					"Description": "My dashboard"
				}
			},
			{
				"Name": "ae9f2598-598f-4859-a3d4-832a512b6104",
				"Type": "pivot",
				"AdditionalInfo": {
					"UUID": "ae9f2598-598f-4859-a3d4-832a512b6104",
					"Name": "foo",
					"Description": "foobar"
				}
			}
		]
	}
]
```

## キットのインストール

アップロードしたキットをインストールするには、PUTリクエストを `/api/kit/<uui>`に送信します。UUIDはキットのリストのUUIDフィールドです。 サーバーは、インストールが成功すると200ステータスコードを返します。

## キットのアンインストール

キットを削除するには、`/api/kit/<uuid>`でDELETEリクエストを発行します。