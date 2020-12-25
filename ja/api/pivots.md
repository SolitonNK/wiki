# ピボット/アクショナブル API

ピボットは*アクショナブル*とも呼ばれ、Gravwellに格納されたオブジェクトで、Web GUIが検索結果データから「ピボット」するために使用します。例えば、アクショナブルはIPアドレスで実行可能なクエリのセットを、IPアドレスを*マッチ*させる正規表現とともに定義することができます。ユーザーが結果にIPアドレスを含むクエリを実行すると、それらのアドレスがクリック可能になり、事前に定義されたクエリを起動するためのメニューが表示されます。

## データ構造

ピボットの構造には、以下のフィールドが含まれています。:

* GUID: テンプレートのグローバル参照。キットのインストール全体に渡って持続します。(次のセクションを参照)
* ThingUUID: この特定のテンプレートインスタンスの一意のID。(次のセクションを参照)
* UID: テンプレートの所有者の数値IDです。
* GIDs: このテンプレートが共有されているグループIDの数値の配列。
* Global: ブール値で、テンプレートをすべてのユーザーに表示する場合はtrueに設定してください（管理者のみ）。
* Name: テンプレートの名前です。
* Description: テンプレートのより詳細な説明。
* Updated: テンプレートの最終更新時刻を表すタイムスタンプ。
* Labels: [ラベル](#!gui/labels/labels.md)を含む文字列の配列。
* Disabled: ピボットが無効化されているかどうかを示すブール値。
* Contents: 実際のテンプレート自体の定義（後述）。

ウェブサーバは `Contents` フィールドに何が入るかを気にしませんが(有効なJSONでなければならないことを除いて)、**GUI**が使用する特定のフォーマットがあります。以下はアクション可能な構造体の完全なTypescriptの定義で、Contentsフィールドと使用される様々な型の説明を含みます。

```
interface Actionable {
    GUID: UUID;
    ThingUUID: UUID;
    UID: NumericID;
    GIDs: null | Array<NumericID>;
    Global: boolean;
    Name: string;
    Description: string; // Empty string is null
    Updated: string; // Timestamp
    Contents: {
        menuLabel: null | string;
        actions: Array<ActionableAction>;
        triggers: Array<ActionableTrigger>;
    };
    Labels: null | Array<string>;
    Disabled: boolean;
}

type UUID = string;
type NumericID = number;

interface ActionableTrigger {
    pattern: string;
    hyperlink: boolean;
}

interface ActionableAction {
    name: string;
    description: string | null;
    placeholder: string | null;
    start?: ActionableTimeVariable;
    end?: ActionableTimeVariable;
    command: ActionableCommand;
}

type ActionableTimeVariable =
    | { type: 'timestamp'; format: null | string; placeholder: null | string }
    | { type: 'string'; format: null | string; placeholder: null | string };

type ActionableCommand =
    | { type: 'query'; reference: string; options?: {} }
    | { type: 'template'; reference: UUID; options?: {} }
    | { type: 'savedQuery'; reference: UUID; options?: {} }
    | { type: 'dashboard'; reference: UUID; options?: { variable?: string } }
    | { type: 'url'; reference: string; options: { modal?: boolean; modalWidth?: string } };
```

## ネーミング: GUIDs と ThingUUIDs

ピボット/アクショナブルには、GUIDとThingUUIDという2つの異なるIDが添付されています。これらは両方ともUUIDで、紛らわしいかもしれません--なぜ1つのオブジェクトに2つの識別子があるのでしょうか？このセクションでは、それを明らかにしようと思います。

例を考えてみましょう。ピボットをゼロから作成すると、ランダムな GUID `e80293f0-5732-4c7e-a3d1-2fb779b91bf7` とランダムな ThingUUID `c3b24e1-5186-4828-82ee-82724a1d4c45` が割り当てられます。そして、ピボットをキットにバンドルします。同じシステムの別のユーザがこのキットを自分用にインストールすると、ピボットのインスタンスが **同じ** GUID (`e80293f0-5732-4c7e-a3d1-2fb779b91bf7`)、**ランダムな** ThingUUID (`f07373a8-ea85-415f-8dfd-61f7b9204ae0`)で作成されます。

このシステムは、[テンプレート](templates.md)で使用されているものと同じです。テンプレートはGUIDとThingUUIDを使用しているので、ダッシュボードはGUIDでテンプレートを参照することができますが、複数のユーザーが同じキット(サンプルテンプレート付き)を同時にインストールしても競合することはありません。ダッシュボードがテンプレートを参照するのと同じ方法でアクショナブルを参照するGravwellコンポーネントはありませんが、将来を見据えてこのような動作にしました。

### ピボットへのアクセス　GUID対ThingUUID

通常のユーザーは常に GUID でピボットにアクセスする必要があります。管理者ユーザーは代わりにThingUUIDでピボットを参照することができますが、リクエストURLに `?admin=true` パラメータを設定しなければなりません。

## ピボットを作成する

ピボットを作成するには、`/api/pivots`にPOSTを発行します。ボディは、'Contents'フィールドと、オプションでGUID、ラベル、名前、説明を持つJSON構造体でなければなりません。例えば、以下のようになります。

```
{
  "Name": "IP actions",
  "Description": "Actions for an IP address",
  "Contents": {
    "actions": [
      {
        "name": "Whois",
        "description": null,
        "placeholder": null,
        "start": {
          "type": "string",
          "format": null,
          "placeholder": null
        },
        "end": {
          "type": "string",
          "format": null,
          "placeholder": null
        },
        "command": {
          "type": "url",
          "reference": "https://www.whois.com/whois/_VALUE_",
          "options": {}
        }
      }
    ],
    "menuLabel": null,
    "triggers": [
      {
        "pattern": "/\\b(?:[0-9]{1,3}\\.){3}[0-9]{1,3}\\b/g",
        "hyperlink": true
      }
    ]
  }
}
```

API は、 新たに作成したピボットの GUID で応答します。リクエストに GUID が指定されている場合は、その GUID が使用されます。GUID を指定されていないときは、ランダムな GUID が生成されます。

注意: 現時点では、`UID`、`GIDs`、および `Global` フィールドはピボットの作成中に設定できません。代わりに、更新呼び出しで設定する必要があります (以下を参照)。

## ピボットの一覧表示

ユーザが利用できるすべてのピボットをリストアップするには、`/api/pivots`をGETします。結果はピボットの配列になります。

```
[
  {
    "GUID": "afba4f9b-f66a-4f9f-9c58-f45b3db6e474",
    "ThingUUID": "196a3cc3-ec9e-11ea-bfde-7085c2d881ce",
    "UID": 1,
    "GIDs": null,
    "Global": false,
    "Name": "IP actions",
    "Description": "Actions for an IP address",
    "Updated": "2020-09-01T15:57:23.416537696-06:00",
    "Contents": {
      "actions": [
        {
          "name": "Whois",
          "description": null,
          "placeholder": null,
          "start": {
            "type": "string",
            "format": null,
            "placeholder": null
          },
          "end": {
            "type": "string",
            "format": null,
            "placeholder": null
          },
          "command": {
            "type": "url",
            "reference": "https://www.whois.com/whois/_VALUE_",
            "options": {}
          }
        }
      ],
      "menuLabel": null,
      "triggers": [
        {
          "pattern": "/\\b(?:[0-9]{1,3}\\.){3}[0-9]{1,3}\\b/g",
          "hyperlink": true
        }
      ]
    },
    "Labels": null,
    "Disabled": false
  },
  {
    "GUID": "34ba8372-0314-460a-9742-5a65c18d6241",
    "ThingUUID": "e1bdf35a-de7b-11ea-9709-7085c2d881ce",
    "UID": 1,
    "GIDs": [
      0
    ],
    "Global": false,
    "Name": "Network Port",
    "Description": "Actions to take on a network port, e.g. 22",
    "Updated": "2020-08-14T16:17:03.790048874-06:00",
    "Contents": {
      "actions": [
        {
          "name": "Netflow - Most active hosts on this port",
          "description": null,
          "placeholder": null,
          "start": {
            "type": "string",
            "format": null,
            "placeholder": null
          },
          "end": {
            "type": "string",
            "format": null,
            "placeholder": null
          },
          "command": {
            "type": "query",
            "reference": "tag=netflow netflow Src Dst SrcPort DstPort Port==_VALUE_ Protocol Bytes | stats sum(Bytes) as ByteTotal by Port Src Dst | lookup -r network_services Protocol proto_number proto_name as Proto Port service_port service_name as Service | table Src Dst Port Service Proto ByteTotal",
            "options": {}
          }
        },
        {
          "name": "Netflow - Chart traffic",
          "description": "Traffic on this port over time",
          "placeholder": null,
          "start": {
            "type": "string",
            "format": null,
            "placeholder": null
          },
          "end": {
            "type": "string",
            "format": null,
            "placeholder": null
          },
          "command": {
            "type": "query",
            "reference": "tag=netflow netflow Src Dst SrcPort DstPort Port==_VALUE_ Protocol Bytes | lookup -r network_services Protocol proto_number proto_name as Proto Port service_port service_name as Service | stats sum(Bytes) by Service Port | chart sum by Service Port",
            "options": {}
          }
        },
        {
          "name": "Netflow - Internal IPs serving this port",
          "description": null,
          "placeholder": null,
          "start": {
            "type": "string",
            "format": null,
            "placeholder": null
          },
          "end": {
            "type": "string",
            "format": null,
            "placeholder": null
          },
          "command": {
            "type": "query",
            "reference": "tag=netflow netflow Dst ~ PRIVATE DstPort==_VALUE_ Bytes Protocol | lookup -r ip_protocols Protocol Number Name as ProtocolName | stats sum(Bytes) as TotalTraffic by Dst | table Dst DstPort Protocol ProtocolName TotalTraffic",
            "options": {}
          }
        }
      ],
      "menuLabel": null,
      "triggers": []
    },
    "Labels": [
      "kit/io.gravwell.netflowv5"
    ],
    "Disabled": false
  }
]
```

## 単一ピボットの取得

単一のピボットを取得するには、`/api/pivots/<guid>`にGETリクエストを発行します。例えば、`/api/pivots/afba4f9b-f66a-4f9f-9c58-f45b3db6e474`をGETすると、次の内容が返ってきます:

```
{
  "GUID": "afba4f9b-f66a-4f9f-9c58-f45b3db6e474",
  "ThingUUID": "196a3cc3-ec9e-11ea-bfde-7085c2d881ce",
  "UID": 1,
  "GIDs": null,
  "Global": false,
  "Name": "IP actions",
  "Description": "Actions for an IP address",
  "Updated": "2020-09-01T15:57:23.416537696-06:00",
  "Contents": {
    "actions": [
      {
        "name": "Whois",
        "description": null,
        "placeholder": null,
        "start": {
          "type": "string",
          "format": null,
          "placeholder": null
        },
        "end": {
          "type": "string",
          "format": null,
          "placeholder": null
        },
        "command": {
          "type": "url",
          "reference": "https://www.whois.com/whois/_VALUE_",
          "options": {}
        }
      }
    ],
    "menuLabel": null,
    "triggers": [
      {
        "pattern": "/\\b(?:[0-9]{1,3}\\.){3}[0-9]{1,3}\\b/g",
        "hyperlink": true
      }
    ]
  },
  "Labels": null,
  "Disabled": false
}

```

管理者は、ThingUUIDとadminパラメータを使用して、明示的にこの特定のピボットを取得することができることに留意してください。例. `/api/pivots/196a3cc3-ec9e-11ea-bfde-7085c2d881ce?admin=true`.

## ピボットの更新

ピボットを更新するには、`/api/pivots/<guid>`にPUTリクエストを発行します。リクエスト本文は、同じパスのGETによって返されたものと同じでなければならず、必要な要素は変更されていなければなりません。GUIDとThingUUIDは変更できないことに注意してください。次のフィールドのみが変更可能です:

* Contents: ピボットの実際の本体/コンテンツ
* Name: ピボットの名前を変更します。
* Description: ピボットの説明を変更します。
* GIDs. 32ビット整数のグループIDの配列を設定することができます。例. `"GIDs":[1,4]`
* UID: (管理者のみ) 32 ビット整数に設定します。
* Global: (管理者のみ) ブール値で true もしくは falseに設定します。グローバルピボットはすべてのユーザーに表示されます。

注意: これらのフィールドを空白にしておくと、そのフィールドがNULL値でピボットが更新されます！

## ピボットの削除

ピボットを削除するには、`/api/pivots/<guid>`にDELETEリクエストを発行します。

## 管理者のアクション

管理者ユーザーは、システム上のすべてのピボットを表示したり、変更したり、削除したりする必要がある場合があります。GUIDは必ずしも一意ではないので、管理者APIは、代わりにGravwellがアイテムを保存するために内部的に使用する一意のUUIDを参照しなければなりません。上記のピボットリストの例には、「ThingUUID」という名前のフィールドが含まれていることに注意してください。これは、そのピボットの内部的な一意の識別子です。

管理者ユーザーは、`/api/pivots?admin=true`のGETリクエストで、システム内のすべてのピボットのグローバルリストを取得することができます。

管理者は、特定のピボットを `/api/pivots/<ThingUUID>?admin=true` へのPUTで更新し、希望するピボットのThingUUID値を代入することができます。削除についても同じパターンが適用されます。

管理者は、`/api/pivots/<ThingUUID>?admin=true`に対してGETまたはDELETEリクエスト(それぞれ)で特定のピボットにアクセスしたり削除したりすることができます。
