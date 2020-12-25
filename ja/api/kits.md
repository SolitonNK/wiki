# Kits Web API

この API は、Gravwell Kitsの作成、インストール、削除を実装します。Kitsには、特定の問題に対するすぐに使えるソリューションを提供するためにローカルシステムにインストールされる他のコンポーネントが含まれています。Kitsには、以下のものを含めることができます。

* リソース
* 検索のスケジュール
* ダッシュボード
* 自動抽出器の定義
* テンプレート
* ピボット
* ユーザーファイル
* マクロ
* ライブラリのエントリを検索する

指定されたKitは、ビルド時に指定された以下の属性も持ちます。

* ID: このkitのユニークな識別子です。Android の命名方法に従うことをお勧めします。
* Name: 覚えやすい名前、例えば「My Kit」。
* Description: Kitの説明です。
* Version: Kitの整数版.

## Kitの構築

Kitsは、以下に定義するKitBuildRequest構造体を含むPOSTリクエストを `/api/kits/build` に送信することでビルドされます。

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
	SearchLibraries   []uuid.UUID    
	Extractors        []uuid.UUID
	Icon              string    
	Dependencies      []KitDependency
	ConfigMacros	  []KitConfigMacro
	ScriptDeployRules map[int32]ScriptDeployConfig
}
```

ID、名前、説明、バージョンフィールドは必須ですが、テンプレート/ピボット/ダッシュボードなどの配列はオプションであることに注意してください。例えば、2つのダッシュボード、ピボット、リソース、スケジュールされた検索を含むキットを構築するリクエストを以下に示します。

```
{
    "ConfigMacros": [
        {
            "DefaultValue": "windows",
            "Description": "Tag or tags containing Windows event entries",
            "MacroName": "KIT_WINDOWS_TAG"
        }
    ],
    "Dashboards": [
        7,
        10
    ],
    "Description": "Test Gravwell kit",
    "ID": "io.gravwell.test",
    "Name": "test-gravwell",
    "Pivots": [
        "ae9f2598-598f-4859-a3d4-832a512b6104"
    ],
    "Resources": [
        "84270dbd-1905-418e-b756-834c15661a54"
    ],
    "ScheduledSearches": [
        1439174790
    ],
    "ScriptDeployRules": {
        "1439174790": {
            "Disabled": true,
            "RunImmediately": false
        }
    },
    "Version": 1
}

```

注意。テンプレート、ピボット、およびユーザーファイルに指定されたUUIDは、アイテムのリストでも報告される*ThingUUID*フィールドではなく、それらの構造体に関連付けられた*GUID*でなければなりません。

システムは、新しく構築されたKitを記述した構造体で応答します。

```
{
	"UUID": "2f5e485a-2739-475b-810d-de4f80ae5f52",
	"Size": 8268288,
	"UID": 1
}
```

このKitは、`/api/kits/build/<uuid>`でGETすることでダウンロードできます。上記のレスポンスがあれば、`/api/kits/build/2f5e485a-2739-475b-810d-de4f80ae5f52`からKitを取得します。

### 依存関係

Kitは他のKitに依存している場合があります。これらの依存関係を依存関係配列にリストアップするには、以下のようにしてください。

```
{
	ID			string
	MinVersion	uint
}
```

ID フィールドは依存関係のIDを指定します。MinVersion」フィールドは、インストールする必要があるKitの最小バージョンを指定します。

### Config Macros

Kitは「Config Macros」を定義することができます。これは、Kitがインストールされたときに Gravwell によって作成される特別なマクロです。Config Macrosは以下のようになります。

```
{
	"MacroName": "KIT_WINDOWS_TAG",
	"Description": "Tag or tags containing Windows event entries",
	"DefaultValue": "windows",
	"Value": "",
	"Type": "TAG"
}
```

UI は、インストール時にマクロの希望する値を要求し、ユーザーの応答を KitConfig 構造体に含める必要があります。

Config macroの定義には、マクロが期待する値の種類についてのヒントとなるTypeフィールドを含めることができます。現在、以下のオプションが定義されています。

	* "TAG": 値は有効なタグでなければなりません。このタグは必ずしも現在のシステムに存在している必要はありませんが、存在しないタグを入力した場合にチェックして警告するのに便利な場合があります。
	* "STRING": 値は自由形式の文字列です。

Typeが指定されていない場合は、"STRING"（自由形式入力）とします。

### スクリプトのデプロイ設定

デフォルトでは、Kitに含まれるスクリプトはインストール時に有効に設定されます。この動作は、スクリプトのデプロイ設定構造を通して制御することができます。

```
{
	"Disabled": false,
	"RunImmediately": true,
}
```

構造体には、「Disabled」と「RunImmediately」の2つのフィールドが含まれています。Disabledがtrueに設定されている場合、スクリプトは無効な状態でインストールされます。RunImmediatelyがtrueに設定されている場合、スクリプトはインストール後できるだけ早く実行されます。

スクリプトのデプロイオプションは、*kit build time*に設定したり、*kit deploy time*に設定したりして、Kitの組み込みオプションを上書きすることができます。

Kitをビルドする際、`ScriptDeployRules` フィールドには、スケジュールされたスクリプトID番号 (`ScheduledSearch` フィールドにリストされている) とスクリプトデプロイの設定構造とのマッピングが含まれていなければなりません。

Kitをインストールする際、`ScriptDeployRules` フィールドには、スケジュールされたスクリプトの *names* から設定へのマッピングが含まれている必要があります。デフォルトを上書きしたい場合は、インストール時にデプロイメントオプションを指定する必要があることに注意してください。

## Kitのアップロード

Kitをインストールする前に、まずウェブサーバにアップロードしなければなりません。Kitは `/api/kits` への POST リクエストでアップロードされます。リクエストにはマルチパートフォームを含める必要があります。ローカルシステムからファイルをアップロードするには、Kitファイルを含む `file` という名前のフォームにファイルフィールドを追加します。HTTPサーバなどのリモートシステムからファイルをアップロードするには、KitのURLを含む `remote` というフィールドを追加します。

リクエストに `metadata` という名前のフィールドを追加することもできます。このフィールドの内容はサーバによって解析されず、代わりにアップロードされたKitのメタデータフィールドに追加されます。これにより、例えばKitの発信元のURLや、Kitがアップロードされた日付などを追跡することができます。

サーバーは、例えば、アップロードされたKitの説明を返信します:

```
{
    "AdminRequired": false,
    "ConfigMacros": [
        {
            "DefaultValue": "windows",
            "Description": "Tag or tags containing Windows event entries",
            "MacroName": "KIT_WINDOWS_TAG",
            "Type": "TAG",
            "Value": "winlog"
        }
    ],
    "ConflictingItems": [
        {
            "AdditionalInfo": {
                "Description": "ASN database",
                "ResourceName": "maxmind_asn",
                "Size": 6196221,
                "VersionNumber": 1
            },
            "Name": "84270dbd-1905-418e-b756-834c15661a54",
            "Type": "resource"
        }
    ],
    "Description": "Test Gravwell kit",
    "GID": 0,
    "ID": "io.gravwell.test",
    "Installed": false,
    "Items": [
        {
            "AdditionalInfo": {
                "Description": "ASN database",
                "ResourceName": "maxmind_asn",
                "Size": 6196221,
                "VersionNumber": 1
            },
            "Name": "84270dbd-1905-418e-b756-834c15661a54",
            "Type": "resource"
        },
        {
            "AdditionalInfo": {
                "Description": "My dashboard",
                "Name": "Foo",
                "UUID": "5567707c-8508-4250-9121-0d1a9d5ebe32"
            },
            "Name": "a",
            "Type": "dashboard"
        },
        {
            "AdditionalInfo": {
                "DefaultDeploymentRules": {
                    "Disabled": false,
                    "RunImmediately": true
                },
                "Description": "A script",
                "Name": "myScript",
                "Schedule": "* * * * *",
                "Script": "println(\"hi\")"
            },
            "Name": "5aacd602-e6ed-11ea-94d9-c771bfc07a39",
            "Type": "scheduled search"
        }
    ],
    "ModifiedItems": [
        {
            "AdditionalInfo": {
                "Description": "My dashboard",
                "Name": "Foo",
                "UUID": "5567707c-8508-4250-9121-0d1a9d5ebe32"
            },
            "Name": "a",
            "Type": "dashboard"
        }
    ],
    "Name": "test-gravwell",
    "RequiredDependencies": [
        {
            "AdminRequired": false,
            "Assets": [
                {
                    "Featured": true,
                    "Legend": "Littering AAAAAAND",
                    "Source": "cover.jpg",
                    "Type": "image"
                },
                {
                    "Featured": false,
                    "Legend": "",
                    "Source": "readme.md",
                    "Type": "readme"
                }
            ],
            "Created": "2020-03-23T15:36:00.294625802-06:00",
            "Dependencies": null,
            "Description": "A simple test kit that just provides a resource",
            "ID": "io.gravwell.testresource",
            "Ingesters": [
                "simplerelay"
            ],
            "Items": [
                {
                    "AdditionalInfo": {
                        "Description": "hosts",
                        "ResourceName": "devlookup",
                        "Size": 610,
                        "VersionNumber": 1
                    },
                    "Name": "devlookup",
                    "Type": "resource"
                },
                {
                    "AdditionalInfo": "Testkit resource\n\nThis really has no restrictions, go nuts!\n",
                    "Name": "LICENSE",
                    "Type": "license"
                }
            ],
            "MaxVersion": {
                "Major": 0,
                "Minor": 0,
                "Point": 0
            },
            "MinVersion": {
                "Major": 0,
                "Minor": 0,
                "Point": 0
            },
            "Name": "Testing resource kit",
            "Signed": true,
            "Size": 10240,
            "Tags": [
                "syslog"
            ],
            "UUID": "d2a0cb10-ff25-4426-8b87-0dd0409cae48",
            "Version": 1
        }
    ],
    "Signed": false,
    "UID": 7,
    "UUID": "549c0805-a693-40bd-abb5-bfb29fc98ef1",
    "Version": 2
}
```

"ModifiedItems" フィールドに注意してください。このKitの以前のバージョンが既にインストールされている場合、このフィールドには *ユーザーが変更された* アイテムのリストが含まれます。ステージ版Kitをインストールするとこれらの項目が上書きされますので、ユーザーに通知して変更を保存する機会を与えてください。

"ConflictingItems "は、ユーザが作成したオブジェクトと競合するように見えるアイテムをリストアップします。この例では、ユーザが以前に "maxmind_asn "という名前のリソースを作成しているようです。もし `OverwriteExisting` を true に設定してインストールリクエストを送信した場合、そのリソースはKitのバージョンで上書きされます; false に設定した場合、インストールプロセスはエラーを返します。

"RequiredDependencies" フィールドには、このKitの現在アンインストールされている依存関係のメタデータ構造のリストが含まれています。

ConfigMacros フィールドには、このKitによってインストールされる設定マクロのリスト (前のセクションを参照) が含まれています。このKitの以前のバージョン（または別のKit）が同じ名前のマクロを既にインストールしている場合、ウェブサーバは「値」フィールドにマクロの現在の値を事前に入力します。*ユーザー*が以前に同じ名前のマクロをインストールした場合、ウェブーサーバはエラーを返します。

"myScript"という名前のスケジュール検索、特に`DefaultDeploymentRules` フィールドに注目してください。これはスクリプトがどのようにインストールされるかを説明しています。

## リスティングKit

`/api/kits`を GET リクエストすると、既知のすべてのKitのリストが返ってきます。ここでは、システムが一つのKitをアップロードしたがまだインストールされていない場合の結果を示す例を示します。

```
[
    {
        "AdminRequired": false,
        "Description": "Test Gravwell kit",
        "GID": 0,
        "ID": "io.gravwell.test",
        "Installed": false,
        "Items": [
            {
                "AdditionalInfo": {
                    "Description": "ASN database",
                    "ResourceName": "maxmind_asn",
                    "Size": 6196221,
                    "VersionNumber": 1
                },
                "Name": "84270dbd-1905-418e-b756-834c15661a54",
                "Type": "resource"
            },
            {
                "AdditionalInfo": {
                    "DefaultDeploymentRules": {
                        "Disabled": false,
                        "RunImmediately": true
                    },
                    "Description": "count all entries",
                    "Duration": -3600,
                    "Name": "count",
                    "Schedule": "* * * * *",
                    "Script": "var time = import(\"time\")\n\naddSelfTargetedNotification(7, \"hello\", \"/#/search/486574780\", time.Now().Add(30 * time.Second))"
                },
                "Name": "55c81086",
                "Type": "scheduled search"
            },
            {
                "AdditionalInfo": {
                    "Description": "My dashboard",
                    "Name": "Foo",
                    "UUID": "5567707c-8508-4250-9121-0d1a9d5ebe32"
                },
                "Name": "a",
                "Type": "dashboard"
            },
            {
                "AdditionalInfo": {
                    "Description": "foobar",
                    "Name": "foo",
                    "UUID": "ae9f2598-598f-4859-a3d4-832a512b6104"
                },
                "Name": "ae9f2598-598f-4859-a3d4-832a512b6104",
                "Type": "pivot"
            }
        ],
        "Name": "test-gravwell",
        "Signed": false,
        "UID": 7,
        "UUID": "549c0805-a693-40bd-abb5-bfb29fc98ef1",
        "Version": 1
    }
]
```

キットアイテムの各タイプで使用できる「AdditionalInfo」フィールドのリストについては、このページの最後にあるリストを参照してください。

## Kit情報

`/api/kits/<GUID>`に対するGETリクエスト（`<GUID>`は特別にインストールまたはステージングされたキットのGUID）は、その特定のキットに関する情報を提供します。

たとえば、`/api/kits/549c0805-a693-40bd-abb5-bfb29fc98ef1`に対するGETリクエストは、次のようになります。

```
{
    "AdminRequired": false,
    "Description": "Test Gravwell kit",
    "GID": 0,
    "ID": "io.gravwell.test",
    "Installed": false,
    "Items": [
        {
            "AdditionalInfo": {
                "Description": "ASN database",
                "ResourceName": "maxmind_asn",
                "Size": 6196221,
                "VersionNumber": 1
            },
            "Name": "84270dbd-1905-418e-b756-834c15661a54",
            "Type": "resource"
        },
        {
            "AdditionalInfo": {
                "DefaultDeploymentRules": {
                    "Disabled": false,
                    "RunImmediately": true
                },
                "Description": "count all entries",
                "Duration": -3600,
                "Name": "count",
                "Schedule": "* * * * *",
                "Script": "var time = import(\"time\")\n\naddSelfTargetedNotification(7, \"hello\", \"/#/search/486574780\", time.Now().Add(30 * time.Second))"
            },
            "Name": "55c81086",
            "Type": "scheduled search"
        },
        {
            "AdditionalInfo": {
                "Description": "My dashboard",
                "Name": "Foo",
                "UUID": "5567707c-8508-4250-9121-0d1a9d5ebe32"
            },
            "Name": "a",
            "Type": "dashboard"
        },
        {
            "AdditionalInfo": {
                "Description": "foobar",
                "Name": "foo",
                "UUID": "ae9f2598-598f-4859-a3d4-832a512b6104"
            },
            "Name": "ae9f2598-598f-4859-a3d4-832a512b6104",
            "Type": "pivot"
        }
    ],
    "Name": "test-gravwell",
    "Signed": false,
    "UID": 7,
    "UUID": "549c0805-a693-40bd-abb5-bfb29fc98ef1",
    "Version": 1
}

```

Kitが存在しない場合は404が返され、ユーザーが要求された特定のKitへのアクセス権を持っていない場合は400が返されます。

## Kitのインストール

アップロードされたKitをインストールするには、`/api/kits/<uuid>` にPUTリクエストを送ります。サーバはいくつかの予備チェックを行い、整数値を返します。これはインストールステータス API (以下を参照) を使ってインストールの進捗状況を問い合わせるために使うことができます。

インストール中、必要な依存関係（ステージングレスポンスのRequiredDepdenciesフィールドにリストされている）はすべてステージングされ、Kit自体の最終的なインストールの前に自動的にインストールされます。

追加のKitインストールオプションは、リクエストの本文に設定構造体を渡すことで指定することができます。

```
{
    "AllowUnsigned": false,
    "ConfigMacros": [
        {
            "DefaultValue": "windows",
            "Description": "Tag or tags containing Windows event entries",
            "MacroName": "KIT_WINDOWS_TAG",
            "Value": "winlog"
        }
    ],
    "Global": true,
    "InstallationGroup": 3,
    "Labels": [
        "foo",
        "bar"
    ],
    "OverwriteExisting": true,
    "ScriptDeployRules": {
        "myScript": {
            "Disabled": true,
            "RunImmediately": false
        }
    }
}
```

注意: 以下の項目はすべてオプションです。デフォルトのオプションを使用するには、単にリクエストからボディを省略してください。

設定されている場合、`OverwriteExisting` は、Kitのバージョンと同じ名前のユニークな識別子を持つ既存のアイテムを単純に置き換えるようインストーラに指示します。

`Global`フラグは管理者のみが設定できます。設定されている場合、すべてのアイテムはグローバルとしてマークされ、すべてのユーザがアクセスできることを意味します。

通常のユーザーは、Gravwell から適切に署名されたKitしかインストールできません。`AllowUnsigned`が設定されている場合、*administrators* は署名なしKitをインストールすることができます。

`InstallationGroup` は、インストールするユーザが自分の属するグループの一つとKitの内容を共有することを可能にする。

`Labels`は追加のラベルのリストで、インストール時にKit内のすべてのラベル付け可能なアイテムに適用されるべきである。Gravwellは、Kitをインストールしたアイテムに"kit"とキットのID(例:"io.gravwell.coredns")で自動的にラベルを付けることに注意してください。

`ConfigMacros`はKit情報構造体にある ConfigMacros のリストで、"Value" フィールドにはユーザが望む値を任意に設定することができます。Value" フィールドが空白の場合、ウェブサーバは "DefaultValue" を使用します。

`ScriptDeployRules`には、オーバーライドしたいKit内のスケジュールされたスクリプトのデプロイルールのオーバーライドが含まれていなければなりません。この例では、"myScript"という名前のスクリプトが無効な状態でインストールされます。デフォルトのデプロイメントオプションを使用してもよい場合は、このフィールドは空のままにしておくことができます。

### インストール状態のAPI

多くの依存関係を持つ大きなパッケージのインストールには時間がかかることがあるため、インストール要求が送信されると、サーバはその要求を処理のためのキューに入れます。サーバはインストール要求に対して、例えば `2019727887` のような整数値で応答します。これはインストールステータスAPIを使って`/api/kits/status/<id>`にGETリクエストを送ることでインストールの進捗状況を問い合わせることができます。

```
{
    "CurrentStep": "Done",
    "Done": true,
    "Error": "",
    "InstallID": 2019727887,
    "Log": "\nQueued installation of kit io.gravwell.testresource, with 0 dependencies also to be installed\nBeginning installation of io.gravwell.testresource (9b701e75-76ee-40fc-b9b5-4c7e1706339d) for user Admin John (1)\nInstalling requested kit io.gravwell.testresource\nDone",
    "Owner": 1,
    "Percentage": 1,
    "Updated": "2020-03-25T15:39:37.184221203-06:00"
}
```

"Owner" は、インストール要求を送信したユーザーの UID です。"Done" は、Kitが完全にインストールされたときに true に設定されます。"percentage" は、インストールがどの程度完了したかを示す 0 から 1 の間の値です。"CurrentStep "はインストールの現在の状態を表し、"Log "はインストール全体の状態の完全な記録を保持します。"Error "は、インストールプロセスで何か問題が発生していない限り、空になります。"Updated "は、ステータスが最後に変更された時刻です。

また、`/api/kits/status`をGETすることで、*all* Kitのインストールステータスのリストを要求することもできます。デフォルトでは、現在のユーザのステータスのみを返すことに注意してください。管理者は、システム上の*すべての*ステータスを取得するためにURLに `?admin=true` を追加することができます。

## kitキットのアンインストール

Kitを削除するには、`/api/kits/<uuid>`でDELETEリクエストを発行してください。Kit内のアイテムがインストール後にユーザによって変更されている場合、レスポンスには400のステータスコードが表示され、何が変更されたかを詳細に示す構造体が含まれています。

```
{
    "Error": "Kit items have been modified since installation, set ?force=true to override",
    "ModifiedItems": [
        {
            "AdditionalInfo": {
                "Description": "Network services (protocol + port) database",
                "ResourceName": "network_services",
                "Size": 531213,
                "VersionNumber": 1
            },
            "ID": "2e4c8f31-92a4-48b5-a040-d2c895caf0b2",
            "KitID": "io.gravwell.networkenrichment",
            "KitName": "Network enrichment",
            "KitVersion": 1,
            "Name": "network_services",
            "Type": "resource"
        }
    ]
}
```

Kitを強制的に削除するには、`?force=true` パラメータをリクエストに追加します。

## リモートKit Serverへの問い合わせ

Gravwell Kit Serverからリモートキットのリストを取得するには、`/api/kits/remote/list`でGETを発行します。 これは、利用可能なすべてのキットの最新バージョンを表すキットメタデータ構造のJSONエンコードされたリストを返します。 APIパス `/api/kits/remote/list/all` は、すべてのバージョンのすべてのキットを提供します。

メタデータの構造は以下の通りです:

```
type KitMetadata struct {
	ID            string
	Name          string
	GUID          string
	Version       uint
	Description   string
	Signed        bool
	AdminRequired bool
	MinVersion    CanonicalVersion
	MaxVersion    CanonicalVersion
	Size          int64
	Created       time.Time
	Ingesters     []string //ingesters associated with the kit
	Tags          []string //tags associated with the kit
	Assets        []KitMetadataAsset
}

type KitMetadataAsset struct {
	Type     string
	Source   string //URL
	Legend   string //some description about the asset
	Featured bool
}

type CanonicalVersion struct {
	Major uint32
	Minor uint32
	Point uint32
}
```

ここでは一例を紹介します:

```
WEB GET http://172.19.0.2:80/api/kits/remote/list:
[
	{
		"ID": "io.gravwell.test",
		"Name": "testkit",
		"GUID": "c2870b48-ff31-4550-bd58-7b2c1c10eeb3",
		"Version": 1,
		"Description": "Testing a kit with a license in it",
		"Signed": true,
		"AdminRequired": false,
		"MinVersion": {
			"Major": 0,
			"Minor": 0,
			"Point": 0
		},
		"MaxVersion": {
			"Major": 0,
			"Minor": 0,
			"Point": 0
		},
		"Size": 0,
		"Created": "2020-02-10T16:31:23.03192303Z",
		"Ingesters": [
			"SimpleRelay",
			"FileFollower"
		],
		"Tags": [
			"syslog",
			"auth"
		],
		"Assets": [
			{
				"Type": "image",
				"Source": "cover.jpg",
				"Legend": "TEAM RAMROD!",
				"Featured": true
			},
			{
				"Type": "readme",
				"Source": "readme.md",
				"Legend": "",
				"Featured": false
			},
			{
				"Type": "image",
				"Source": "testkit.jpg",
				"Legend": "",
				"Featured": false
			}
		]
	}
]
```

## プルシングルKit情報

リモートKit APIは、`/api/kits/remote/<guid>` で `GET` を発行することで、特定のKitに関する情報を取得することもサポートしています。

例えば、`/api/kits/remote/c2870b48-ff31-4550-bd58-7b2c1c10eeb3` に `GET` を発行すると、ウェブサーバは戻ります。

```
{
	"ID": "io.gravwell.test",
	"Name": "testkit",
	"GUID": "c2870b48-ff31-4550-bd58-7b2c1c10eeb3",
	"Version": 1,
	"Description": "Testing a kit with a license in it",
	"Signed": true,
	"AdminRequired": false,
	"MinVersion": {
		"Major": 0,
		"Minor": 0,
		"Point": 0
	},
	"MaxVersion": {
		"Major": 0,
		"Minor": 0,
		"Point": 0
	},
	"Size": 0,
	"Created": "2020-02-10T16:31:23.03192303Z",
	"Ingesters": [
		"SimpleRelay",
		"FileFollower"
	],
	"Tags": [
		"syslog",
		"auth"
	],
	"Assets": [
		{
			"Type": "image",
			"Source": "cover.jpg",
			"Legend": "TEAM RAMROD!",
			"Featured": true
		},
		{
			"Type": "readme",
			"Source": "readme.md",
			"Legend": "",
			"Featured": false
		},
		{
			"Type": "image",
			"Source": "testkit.jpg",
			"Legend": "",
			"Featured": false
		}
	]
}
```

### リモートのKitサーバからKitアセットを引き出す

Kitには、実際にKitをダウンロード、インストールする前にKitの目的を探るのに役立つ画像、マークダウン、ライセンス、追加ファイルを表示するために使用できるアセットも含まれています。これらのアセットは、`api/kits/remote/<guid>/<asset>`でGETリクエストを実行することで、リモートシステムから取得することができます。例えば、`c2870b48-ff31-4550-bd58-7b2c1c10eeb3`というguidを持つKitのタイプ「image」とレジェンド「TEAM RAMROD！」のアセットを取得したい場合、`/api/kits/remote/c2870b48-ff31-4550-bd58-7b2c1c10eeb3/cover.jpg`にGETを発行します。


## Kitアイテム「AddditionalInfo」フィールド

Kitをリストアップする場合(`/api/kits`でGET)、各キットにはAddditionalInfoフィールドを含むアイテムのリストが含まれます。これらのフィールドは、Kit内のアイテムに関するより多くの情報を提供します、内容はアイテムのタイプによって異なり、以下に列挙します。

```
Resources:
		VersionNumber int
		ResourceName  string
		Description   string
		Size          uint64

Scheduled Search:
		Name                    string
		Description             string
		Schedule                string
		SearchString            string 
		Duration                int64  
		Script                  string 
		DefaultDeploymentRules  ScriptDeployConfig

Dashboard:
		UUID        string
		Name        string
		Description string

Extractor:
		Name   string 
		Desc   string 
		Module string 
		Tag    string 

Template:
		UUID        string
		Name        string
		Description string

Pivot:
		UUID        string
		Name        string
		Description string

File:
		UUID        string
		Name        string
		Description string
		Size        int64
		ContentType string

Macro:
		Name      string
		Expansion string

Search Library:
		Name        string
		Description string
		Query       string

Playbook:
		UUID        string
		Name        string
		Description string

License:
		(contents of license file itself)
```

## Kit ビルドリクエスト履歴

成功したKitのビルドリクエストはウェブサーバに保存されます。現在のユーザのビルドリクエストのリストは、`/api/kits/build/history` に GETリクエストを送ることで取得できます。レスポンスはビルドリクエストの配列になります。

```
[{"ID":"io.gravwell.test","Name":"test","Description":"","Version":1,"MinVersion":{"Major":0,"Minor":0,"Point":0},"MaxVersion":{"Major":0,"Minor":0,"Point":0},"Macros":[4,41],"ConfigMacros":null}]
```

注意: このストアはUID + Kit IDがキーになっています; "io.gravwell.test "という名前のKitを再度ビルドすると、ストア内のバージョンが上書きされます。

特定の項目を削除するには、`/api/kits/build/history/<id>`にDELETEリクエストを送ることで、例えば、`/api/kits/build/history/io.gravwell.test`のように、`/api/kits/build/history/io.gravwell.test`のように削除することができます。
