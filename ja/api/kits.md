# キットWeb API

このAPIは、Gravwellキットの作成、インストール、および削除を実装します。 キットには、特定の問題に対するすぐに使えるソリューションを提供するためにローカルシステムにインストールされる他のコンポーネントが含まれています。 キットには以下を含めることができます：

* リソース
* スケジュールされた検索
* ダッシュボード
* 自動抽出器の定義
* テンプレート
* ピボット
* ユーザーファイル
* マクロ
* ライブラリエントリを検索する

特定のキットには、ビルド時に指定される次の属性もあります：

* ID：このキットの一意の識別子。 Androidの命名方法に従うことをお勧めします。 「com.example.my-kit」。
* 名前：キットのわかりやすい名前。例： 「マイキット」。
* 説明：キットの説明。
* バージョン：キットの整数バージョン。

## キットの作成

キットは、以下に定義するように、KitBuildRequest構造を含む `/api/kits/build`にPOSTリクエストを送信することでビルドされます。
```
type KitBuildRequest struct {
	ID                string
	Name              string
	Description       string
	Version           uint
	MinVersion        CanonicalVersion 
	MaxVersion        CanonicalVersion 
	Dashboards        []uint64 
	Templates         []uuid.UUID 
	Pivots            []uuid.UUID 
	Resources         []string 
	ScheduledSearches []int32 
	Macros            []uint64 
	Extractors        []uuid.UUID 
	Files             []uuid.UUID 
	SearchLibraries   []uuid.UUID 
	Playbooks         []uuid.UUID 
	EmbeddedItems     []KitEmbeddedItem 
	Icon              string 
	Banner            string 
	Cover             string 
	Dependencies      []KitDependency 
	ConfigMacros      []KitConfigMacro
	ScriptDeployRules map[int32]ScriptDeployConfig
}
```

ID、名前、説明、バージョンの各フィールドは必須ですが、テンプレート/ピボット/ダッシュボードなどの配列はオプションであることに注意してください。 たとえば、2つのダッシュボード、ピボット、リソース、およびスケジュールされた検索を含むキットを作成するリクエストは次のとおりです：

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
    "Description":"testing\n\n## TESTING",
    "Pivots": [
        "ae9f2598-598f-4859-a3d4-832a512b6104"
    ],
    "Resources": [
        "84270dbd-1905-418e-b756-834c15661a54"
    ],
    "ScheduledSearches": [
        1439174790
    ],
    "EmbeddedItems":[
        {
           "Name":"TEST",
           "Type":"license",
           "Content":"VGVzdCBsaWNlbnNlIHRoYXQgYWxsb3dzIEdyYXZ3ZWxsIHRvIGdpdmUgeW91ciBmaXJzdCBib3JuIHNvbiBhIHN0ZXJuIHRhbGtpbmcgdG8h"
        }
    ],
    "Files":[
        "810a014d-1373-4d57-95b6-0638a7a01442",
        "09a26a2e-e449-4857-88d1-56cede1b8d95",
        "92bcfe5e-2c9a-4f39-9083-dd3f7a6f9738"
    ],
    "MinVersion":{"Major":4,"Minor":0,"Point":0},
    "MaxVersion":{"Major":4,"Minor":2,"Point":0},
    "Icon":"810a014d-1373-4d57-95b6-0638a7a01442",
    "Banner":"09a26a2e-e449-4857-88d1-56cede1b8d95",
    "Cover":"92bcfe5e-2c9a-4f39-9083-dd3f7a6f9738",
    "ScriptDeployRules": {
        "1439174790": {
            "Disabled": true,
            "RunImmediately": false
        }
    },
    "Version": 1
}
```

重要：テンプレート、ピボット、およびユーザーファイルに指定されるUUIDは、アイテムのリストでも報告される*ThingUUID*フィールドではなく、それらの構造に関連付けられた*GUID*である必要があります。

重要：バナー、カバー、およびアイコンに指定されたUUIDは、ビルド要求のファイルのリストに含まれている必要があります。 ビルドリクエストにメインファイルリクエストに含まれていないファイルUUIDへの参照が含まれている場合、APIサーバーはリクエストを拒否します。

システムは、新しく構築されたキットを説明する構造で応答します：

```
{
	"UUID": "2f5e485a-2739-475b-810d-de4f80ae5f52",
	"Size": 8268288,
	"UID": 1
}
```

このキットは、`/api/kits/build/<uuid>`でGETを実行することでダウンロードできます。 上記の応答が与えられると、`/api/kits/build/2f5e485a-2739-475b-810d-de4f80ae5f52`からキットをフェッチします。

### 依存関係

キットは他のキットに依存する場合があります。次の構造を使用して、これらの依存関係をDependencies配列にリストします：

```
{
	ID			string
	MinVersion	uint
}
```

IDフィールドは、依存関係のIDを指定します（例：io.gravwell.testresource）。 MinVersionフィールドは、インストールする必要のあるキットの最小バージョンを指定します。

### 構成マクロ

キットは、キットのインストール時にGravwellによって作成される特別なマクロである「構成マクロ」を定義する場合があります。 構成マクロは次のようになります。

```
{
	"MacroName": "KIT_WINDOWS_TAG",
	"Description": "Tag or tags containing Windows event entries",
	"DefaultValue": "windows",
	"Value": "",
	"Type": "TAG"
}
```

UIは、インストール時にマクロの目的の値の入力を求め、KitConfig構造にユーザーの応答を含める必要があります。

構成マクロ定義には、マクロが期待する値の種類に関するヒントを提供するTypeフィールドを含めることができます。現在、次のオプションが定義されています：

	* 「TAG」：値は有効なタグである必要があります。 このタグは必ずしも現在のシステムに存在する必要はありませんが、存在しないタグを入力した場合は、ユーザーを確認して警告することが役立つ場合があります。
	* 「STRING」：値は自由形式の文字列にすることができます。

タイプが指定されていない場合は、「STRING」（自由形式のエントリ）と見なします。

### スクリプトデプロイ構成

デフォルトでは、キットに含まれているスクリプトはインストール時に有効に設定されます。 この動作は、スクリプトのデプロイ構成構造を介して制御できます:

```
{
	"Disabled": false,
	"RunImmediately": true,
}
```

この構造には、「Disabled」と「RunImmediately」の2つのフィールドが含まれています。 Disabledがtrueに設定されている場合、スクリプトは無効な状態でインストールされます。 RunImmediatelyがtrueに設定されている場合、*スクリプトが無効になっている場合でも*、インストール後できるだけ早くスクリプトが実行されます。

スクリプトの展開オプションは、*キットのビルド時間*または*キットの展開時間*に設定して、キットの組み込みオプションを上書きできます。

キットを作成する場合、`ScriptDeployRules`フィールドには、スケジュールされたスクリプトID番号（`ScheduledSearches`フィールドにリストされている）からスクリプト展開構成構造へのマッピングが含まれている必要があります。

キットをインストールする場合、`ScriptDeployRules`フィールドには、スケジュールされたスクリプト*名前*から構成へのマッピングが含まれている必要があります。 デフォルトを上書きする場合にのみ、展開オプションをインストール時に指定する必要があることに注意してください。

## キットのアップロード

キットをインストールする前に、まずWebサーバーにアップロードする必要があります。 キットはPOSTリクエストによって`/api/kits`にアップロードされます。 リクエストにはマルチパートフォームが含まれている必要があります。 ローカルシステムからファイルをアップロードするには、キットファイルを含む`file`という名前のフォームにファイルフィールドを追加します。 HTTPサーバーなどのリモートシステムからファイルをアップロードするには、キットのURLを含む`remote`という名前のフィールドを追加します。

`metadata`という名前のフィールドをリクエストに追加することもできます。 このフィールドの内容はサーバーによって解析されません。 代わりに、アップロードされたキットのメタデータフィールドにコンテンツを追加します。 これにより、たとえば次のことを追跡できます。 キットの作成元のURL、キットがアップロードされた日付など。

サーバーは、アップロードされたキットの説明で応答します。
例：

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

「ModifiedItems」フィールドに注意してください。このキットの以前のバージョンがすでにインストールされている場合、このフィールドには*ユーザーが変更した*アイテムのリストが含まれます。ステージキットをインストールするとこれらのアイテムが上書きされるため、ユーザーに通知し、変更を保存する機会を与える必要があります。

「ConflictingItems」には、ユーザーが作成したオブジェクトと競合しているように見えるアイテムが一覧表示されます。この例では、ユーザーが以前に「maxmind_asn」という名前の独自のリソースを作成したようです。 `OverwriteExisting`をtrueに設定してインストール要求が送信された場合、そのリソースはキット内のバージョンで上書きされます。 falseに設定すると、インストールプロセスはエラーを返します。

[RequiredDependencies]フィールドには、このキットの現在アンインストールされている依存関係のメタデータ構造のリストが含まれています。これには、表示する必要のあるライセンスが含まれている可能性のあるアイテムセットが含まれます。

ConfigMacrosフィールドには、このキットによってインストールされる構成マクロ（前のセクションを参照）のリストが含まれています。このキットの以前のバージョン（または別のキット全体）に同じ名前のマクロが既にインストールされている場合、Webサーバーは[値]フィールドにマクロの現在の値を事前入力します。 *ユーザー*が以前に同じ名前のマクロをインストールした場合、Webサーバーはエラーを返します。

「myScript」という名前のスケジュールされた検索、特に「DefaultDeploymentRules」フィールドに注意してください。これは、スクリプトがどのようにインストールされるかを説明しています。スクリプトは有効とマークされ、できるだけ早く実行されます。

## リストキット

`/api/kits`に対するGETリクエストは、すべての既知のキットのリストを返します。 これは、システムに1つのキットがアップロードされているが、まだインストールされていない場合の結果を示す例です：

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

## キット情報

`/api/kits/<GUID>`に対するGETリクエスト（`<GUID>`は特別にインストールまたはステージングされたキットのGUID）は、その特定のキットに関する情報を提供します。

たとえば、`/api/kits/549c0805-a693-40bd-abb5-bfb29fc98ef1`に対するGETリクエストは、次のようになります：

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

キットが存在しない場合は404が返され、ユーザーが要求された特定のキットにアクセスできない場合は400が返されます。

## キットのインストール

アップロード後にキットをインストールするには、PUTリクエストを `/api/kits/<uuid>`に送信します。ここで、UUIDはキットのリストのUUIDフィールドです。 サーバーはいくつかの予備チェックを実行し、整数を返します。整数は、インストールステータスAPIを使用してインストールの進行状況を照会するために使用できます（以下を参照）。

インストール中に、必要なすべての依存関係（ステージング応答のRequiredDepdenciesフィールドにリストされている）がステージングされ、キット自体の最終インストールの前に自動的にインストールされます。

追加のキットインストールオプションは、リクエストの本文に構成構造を渡すことで指定できます。例：

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

注：以下はすべてオプションです。 それらの一部またはすべてを省略できます。 デフォルトのオプションを使用するには、リクエストから本文を省略します。

設定されている場合、`OverwriteExisting`は、キットのバージョンとして一意の識別子という名前を持つ既存のアイテムを単に置き換えるようにインストーラーに指示します。

「グローバル」フラグは、管理者のみが設定できます。 設定すると、すべてのアイテムがグローバルとしてマークされ、すべてのユーザーがアクセスできるようになります。

通常のユーザーは、Gravwellから適切に署名されたキットのみをインストールできます。`AllowUnsigned`が設定されている場合、*管理者*は署名されていないキットをインストールできます。

`InstallationGroup`を使用すると、インストールユーザーはキットの内容を自分が属するグループの1つと共有できます。

「ラベル」は、インストール時にキット内のすべてのラベル付け可能なアイテムに適用する必要がある追加のラベルのリストです。 Gravwellは、キットにインストールされたアイテムに「キット」とキットのID（「io.gravwell.coredns」など）のラベルを自動的に付けることに注意してください。

`ConfigMacros`は、キットの情報構造にあるConfigMacrosのリストであり、オプションで「値」フィールドがユーザーの希望に設定されています。 「値」フィールドが空白の場合、Webサーバーは「DefaultValue」を使用します。

`ScriptDeployRules`には、展開ルールをオーバーライドするキット内のスケジュールされたスクリプトのオーバーライドが含まれている必要があります。 この例では、「myScript」という名前のスクリプトが無効な状態でインストールされます。 デフォルトの展開オプションが受け入れられる場合、このフィールドは空のままにすることができます。

### インストールステータスAPI

インストール要求が送信されると、依存関係の多い大きなパッケージのインストールには時間がかかる場合があるため、サーバーは要求を処理のためにキューに入れます。サーバーは、インストール要求に整数で応答します。 `2019727887`。これをインストールステータスAPIとともに使用して、GETリクエストを `/api/kits/status/<id>`に送信することで、インストールの進行状況を照会できます。 `/api/kits/status/2019727887`はこれを返す可能性があります：

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

「所有者」は、インストール要求を送信したユーザーのUIDです。 キットが完全にインストールされると、「完了」がtrueに設定されます。 「パーセンテージ」は、インストールがどれだけ完了したかを示す0から1までの値です。 「CurrentStep」はインストールの現在のステータスであり、「Log」はインストール全体のステータスの完全な記録を保持します。 インストールプロセスで問題が発生しない限り、「エラー」は空になります。 「更新済み」は、ステータスが最後に変更された時刻です。

`/api/kits/status`でGETを実行して、*すべての*キットのインストールステータスのリストを要求することもできます。これにより、上記の種類のオブジェクトの配列が返されます。 デフォルトでは、これは現在のユーザーのステータスのみを返すことに注意してください。 管理者は、URLに`？admin = true`を追加して、システムの*すべて*のステータスを取得できます。

## キットのアンインストール

キットを削除するには、`/api/kits/<uuid>`でDELETEリクエストを発行します。 キット内のアイテムのいずれかがインストール後にユーザーによって変更された場合、応答には400ステータスコードが含まれ、変更内容の詳細を示す構造が含まれます。

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

UIは、この時点でユーザーにプロンプトを表示する必要があります。 キットを強制的に削除するには、リクエストに `？force = true`パラメータを追加します。

## リモートキットサーバーへのクエリ

Gravwell Kit Serverからリモートキットのリストを取得するには、 `/api /kits/remote/list`でGETを発行します。 これにより、利用可能なすべてのキットの最新バージョンを表すキットメタデータ構造のJSONエンコードリストが返されます。 APIパス `/api/kits/remote/list/all`は、すべてのバージョンのすべてのキットを提供します。

メタデータの構造は次のとおりです：

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

次に例を示します：

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

## Pull Single Kit Information

リモートキットAPIは、`/api/kits/remote/<guid>`で `GET`を発行することにより、特定のキットに関する情報をプルバックすることもサポートします。これにより、単一の`KitMetadata`構造が返されます。

たとえば、`/api/kits/remote/c2870b48-ff31-4550-bd58-7b2c1c10eeb3`で`GET`を発行すると、ウェブサーバーは次を返します：

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

###　リモートキットサーバーからキットアセットをプルする

キットには、実際にキットをダウンロード/インストールする前に、キットの目的を調べるのに役立つ画像、マークダウン、ライセンス、および追加ファイルを表示するために使用できるアセットも含まれています。 これらのアセットは、`api/kits/remote/<guid>/<asset>`でGETリクエストを実行することにより、リモートシステムから取得できます。 たとえば、タイプ「画像」と凡例「TEAMRAMROD！」のアセットを取り戻したい場合です。 GUIDが `c2870b48-ff31-4550-bd58-7b2c1c10eeb3`のキットの場合、`/api/kits/remote/c2870b48-ff31-4550-bd58-7b2c1c10eeb3/cover.jpg`でGETを発行します。


##　キットアイテムの「追加情報」フィールド

キットを一覧表示する場合（`/api/ kits`でGET）、各キットにはAddditionalInfoフィールドを含むアイテムのリストが含まれます。 これらのフィールドには、キット内のアイテムに関する詳細情報が表示されます。 内容はアイテムタイプによって異なり、以下に列挙されています。

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

## Kit Build Request History

成功したキットビルドリクエストはウェブサーバーに保存されます。 GETリクエストを `/api/kits/build/history`に送信することで、現在のユーザーのビルドリクエストのリストを取得できます。 応答は、ビルド要求の配列になります。

```
[{"ID":"io.gravwell.test","Name":"test","Description":"","Version":1,"MinVersion":{"Major":0,"Minor":0,"Point":0},"MaxVersion":{"Major":0,"Minor":0,"Point":0},"Macros":[4,41],"ConfigMacros":null}]
```

注：このストアは、UID +キットIDでキー設定されています。 「io.gravwell.test」という名前のキットを再度作成すると、ストア内のバージョンが上書きされます。

DELETEリクエストを `/api/kits/build/history/<id>`に送信することで、特定のアイテムを削除できます。例：`/api/kits/build/history/io.gravwell.test`。