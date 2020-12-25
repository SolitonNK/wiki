# テンプレートAPI

テンプレートは、*変数*を含むGravwellクエリを定義する特別なオブジェクトです。 同じ変数を使用する複数のテンプレートをダッシュボードに含めて、強力な調査ツールを作成できます。たとえば、変数としてIPアドレスを期待するテンプレートを使用して、IPアドレス調査ダッシュボードを作成できます。

## データ構造

テンプレート構造には、次のフィールドが含まれています：

* GUID：テンプレートのグローバルリファレンス。キットのインストール後も持続します（次のセクションを参照）
* ThingUUID：この特定のテンプレートインスタンスの一意のID（次のセクションを参照）
* UID：テンプレートの所有者の数値ID
* GID：このテンプレートが共有される数値グループIDの配列
* グローバル：ブール値。テンプレートをすべてのユーザーに表示する必要がある場合はtrueに設定します（管理者のみ）
* 名前：テンプレートの名前
* 説明：テンプレートのより詳細な説明
* 更新：テンプレートの最終更新時刻を表すタイムスタンプ
* ラベル： [labels](#!gui/labels/labels.md)を含む文字列の配列
* 内容：テンプレート自体の実際の定義（以下を参照）

Webサーバーは`Contents`フィールドに何が入力されるかを気にしませんが（有効なJSONでなければならないことを除いて）、**GUI**が使用する特定の形式があります。GUIで使用できるようにするには、[コンテンツ]フィールドがこの構造に準拠している必要があります。
```
Contents: {
  query: string;
  variable: string;
  variableLabel: string;
  variableDescription: string | null;
  required: boolean;
  testValue: string | null;
};
```

以下は、テンプレートデータ型の完全なTypescript定義です:


```
interface RawTemplate {
    GUID: RawUUID;
    ThingUUID: RawUUID;
    UID: RawNumericID;
    GIDs: null | Array<RawNumericID>;
    Global: boolean;
    Name: string;
    Description: string; // Empty string is null
    Updated: string; // Timestamp
    Contents: {
        query: string;
        variable: string;
        variableLabel: string;
        variableDescription: string | null;
        required: boolean;
        testValue: string | null;
    };
    Labels: null | Array<string>;
}
```

## ネーミング：GUIDとThingUUID

テンプレートには、GUIDとThingUUIDの2つの異なるIDが添付されています。 これらは両方ともUUIDであり、混乱を招く可能性があります。1つのオブジェクトに2つの識別子があるのはなぜですか。このセクションで明確にしようとします。

Gravwellでは、ダッシュボードが特定のテンプレートを参照する場合があります。 このダッシュボードと対応するテンプレートは、他のユーザーに配布するためにキットにパックすることもできます。 ダッシュボードには、キットにパックされて他の場所にインストールされたときに**永続**するテンプレートを参照する方法が必要です。そのため、テンプレートの「グローバル」名としてGUIDを導入します。キットがインストールされる場所に関係なく、そのテンプレートは 同じGUIDを持っています。ただし、複数のユーザーが同じキットをインストールできるため、テンプレートの*個々のインスタンス化*ごとに異なる識別子も必要です。この役割は、ThingUUIDフィールドによって満たされます。

例を考えてみましょう。ダッシュボードとテンプレートを含むキットを作成します。テンプレートを最初から作成するので、ランダムなGUID`e80293f0-5732-4c7e-a3d1-2fb779b91bf7`とランダムなThingUUID`c3b24e1e-5186-4828-82ee-82724a1d4c45`が割り当てられます。次に、ダッシュボードにGUID（`e80293f0-5732-4c7e-a3d1-2fb779b91bf7`）でテンプレートを参照するタイルを作成し、テンプレートとダッシュボードの両方をキットにバンドルします。次に、同じシステム上の別のユーザーがこのキットを自分でインストールします。これにより、**同じ** GUID（`e80293f0-5732-4c7e-a3d1-2fb779b91bf7`）でテンプレートがインスタンス化されますが、**ランダム** ThingUUID（`f07373a8-ea85-415f-8dfd-61f7b9204ae0`）。ユーザーがダッシュボードを開くと、ダッシュボードはGUID ==  `e80293f0-5732-4c7e-a3d1-2fb779b91bf7`のテンプレートを要求します。 Webサーバーは、ThingUUID == `f07373a8-ea85-415f-8dfd-61f7b9204ae0`を使用してそのユーザーのテンプレートのインスタンスを返します。

ユーザーがグローバルテンプレートと同じGUIDを使用してテンプレートをインストールすると、ユーザーはグローバルテンプレートを透過的にオーバーライドしますが、それは自分自身に対してのみであることに注意してください。同じGUIDを持つ複数のテンプレートが存在する場合、それらは次の順序で優先されます。

* ユーザーが所有
* ユーザーがメンバーであるグループと共有
* グローバル

これは、ユーザーがグローバルダッシュボードにアクセスしている場合、同じGUIDを使用してテンプレートの独自のコピーを作成することにより、ダッシュボードによって参照される特定のテンプレートをオーバーライドできることを意味します。実際には、これはまれなはずです。

### GUIDとThingUUIDを介したテンプレートへのアクセス

通常のユーザーは、常にGUIDでテンプレートにアクセスする必要があります。管理ユーザーは、代わりにThingUUIDでテンプレートを参照できますが、リクエストURLに `？admin = true`パラメータを設定する必要があります。

##テンプレートを作成します

テンプレートを作成するには、`/api/templates`にPOSTを発行します。本文は、有効なJSONと、オプションでGUID、ラベル、名前、説明を含む「コンテンツ」フィールドを持つJSON構造である必要があります。以下はすべて有効です。

```
{
  "Contents": {
    "query": "tag=json json ip==%%IP%% | table",
    "required": true,
    "testValue": "\"10.0.0.1\"",
    "variable": "%%IP%%",
    "variableDescription": "the IP to investigate",
    "variableLabel": "IP address"
  }
}
```

```
{
  "GUID": "ce95b152-d47f-443f-884b-e0b506a215be",
  "Contents": {
    "query": "tag=json json ip==%%IP%% | table",
    "required": true,
    "testValue": "\"10.0.0.1\"",
    "variable": "%%IP%%",
    "variableDescription": "the IP to investigate",
    "variableLabel": "IP address"
  }
}
```

```
{
  "Contents": {
    "query": "tag=json json ip==%%IP%% | table",
    "required": true,
    "testValue": "\"10.0.0.1\"",
    "variable": "%%IP%%",
    "variableDescription": "the IP to investigate",
    "variableLabel": "IP address"
  },
  "Name": "mytemplate"
}
```

```
{
  "GUID": "ce95b152-d47f-443f-884b-e0b506a215be",
  "Contents": {
    "query": "tag=json json ip==%%IP%% | table",
    "required": true,
    "testValue": "\"10.0.0.1\"",
    "variable": "%%IP%%",
    "variableDescription": "the IP to investigate",
    "variableLabel": "IP address"
  },
  "Name": "mytemplate"
}
```

```
{
  "Contents": {
    "query": "tag=json json ip==%%IP%% | table",
    "required": true,
    "testValue": "\"10.0.0.1\"",
    "variable": "%%IP%%",
    "variableDescription": "the IP to investigate",
    "variableLabel": "IP address"
  },
  "Name": "mytemplate",
  "Labels": [
    "suits",
    "ladders"
  ]
}
```

APIは、新しく作成されたテンプレートのGUIDで応答します。リクエストでGUIDが指定されている場合、そのGUIDが使用されます。GUIDが指定されていない場合、ランダムなGUIDが生成されます。

注：現時点では、テンプレートの作成中に`UID`、`GIDs`、および`Global`フィールドを設定することはできません。代わりに、更新呼び出しを介して設定する必要があります（以下を参照）

##リストテンプレート

ユーザーが利用できるすべてのテンプレートを一覧表示するには、`/api/templates`でGETを実行します。結果は、テンプレートの配列になります。

```
[
  {
    "ThingUUID": "1b36a1d7-a5ac-11ea-b07e-7085c2d881ce",
    "UID": 1,
    "GIDs": [
      6,
      8
    ],
    "Global": false,
    "GUID": "780b1d31-e46b-4460-ad83-2fc11c34a162",
    "Name": "json ip",
    "Description": "JSON tag, filter by IP",
    "Contents": {
      "variable": "%%IP%%",
      "query": "tag=json* json ip==%%IP%% | table",
      "variableLabel": "IP address",
      "variableDescription": "the IP to investigate!",
      "required": true,
      "testValue": "\"10.0.0.1\""
    },
    "Updated": "2020-09-01T15:01:18.354750806-06:00",
    "Labels": [
      "test"
    ]
  }
]

```

## 単一のテンプレートを取得しする

単一のテンプレートをフェッチするには、`/api/templates/<guid>`にGETリクエストを発行します。 サーバーはそのテンプレートの内容で応答します。たとえば、`/api/templates/780b1d31-e46b-4460-ad83-2fc11c34a162`のGETは次を返す場合があります。

```
{
  "ThingUUID": "1b36a1d7-a5ac-11ea-b07e-7085c2d881ce",
  "UID": 1,
  "GIDs": [
    6,
    8
  ],
  "Global": false,
  "GUID": "780b1d31-e46b-4460-ad83-2fc11c34a162",
  "Name": "json ip",
  "Description": "JSON tag, filter by IP",
  "Contents": {
    "variable": "%%IP%%",
    "query": "tag=json* json ip==%%IP%% | table",
    "variableLabel": "IP address",
    "variableDescription": "the IP to investigate!",
    "required": true,
    "testValue": "\"10.0.0.1\""
  },
  "Updated": "2020-09-01T15:01:18.354750806-06:00",
  "Labels": [
    "test"
  ]
}
```

管理者は、ThingUUIDとadminパラメータを使用して、この特定のテンプレートを明示的にフェッチできることに注意してください。`/api/templates/1b36a1d7-a5ac-11ea-b07e-7085c2d881ce?admin=true`。

## テンプレートを更新します

テンプレートを更新するには、`/api/templates/<guid>`にPUTリクエストを発行します。リクエストの本文は、必要な要素を変更して、同じパスのGETによって返されるものと同じである必要があります。GUIDとThingUUIDは変更できないことに注意してください。次のフィールドのみを変更できます。

* Contents：テンプレートの実際の本文/内容
* Name：テンプレートの名前を変更します
* Description：テンプレートの説明を変更します
* GIDs：32ビット整数グループIDの配列に設定できます。`"GIDs":[1,4]`
* UID :(管理者のみ）32ビット整数に設定
* Global:(管理者のみ）ブール値のtrueまたはfalseに設定します。グローバルテンプレートはすべてのユーザーに表示されます。

注：これらのフィールドのいずれかを空白のままにすると、テンプレートがそのフィールドのnull値で更新されます。

## テンプレートを削除します

テンプレートを削除するには、`/api/templates/<guid>`にDELETEリクエストを発行します。

## 管理者のアクション

管理ユーザーは、システム上のすべてのテンプレートを表示、変更、または削除する必要がある場合があります。GUIDは必ずしも一意ではないため、管理APIは、代わりにGravwellがアイテムを格納するために内部で使用する一意のUUIDを参照する必要があります。上記のテンプレートリストの例には、「ThingUUID」という名前のフィールドが含まれていることに注意してください。これは、そのテンプレートの内部の一意の識別子です。

管理者ユーザーは、`/api/templates?admin=true`のGETリクエストを使用して、システム内のすべてのテンプレートのグローバルリストを取得できます。

次に、管理者は、PUTを使用して特定のテンプレートを`/api/templates/<ThingUUID>?admin=true`に更新し、目的のテンプレートをThingUUID値に置き換えることができます。同じパターンが削除にも当てはまります。

管理者は、`/api/templates/<ThingUUID>?admin=true`でGETまたはDELETEリクエストを使用して特定のテンプレートにアクセスまたは削除できます（それぞれ）。