# Resources

Web APIはリソースへのアクセスを提供します。あいまいさを防ぐために、リソースはGUIDによって参照される必要があります。

## リソースメタデータ構造
リソースシステムは各リソースのメタデータ構造を保持します。Web APIは、この構造体のJSONエンコードバージョンを使用して通信します。これらのフィールドは大部分は一目瞭然ですが、ここでは正確さのために説明されています。

* UID: リソース所有者のUID
* GUID: リソースの一意の識別子
* LastModified: リソースが最後に更新された時刻
* VersionNumber: リソースの内容が変更されるたびに増加します
* GroupACL: リソースへのアクセスが許可されている整数グループIDのリスト
* Global: trueの場合、リソースはシステム上のすべてのユーザーから読み取り可能です。グローバルリソースを作成できるのは管理者だけです
* ResourceName: リソースの名前
* Description: リソースの詳細な説明
* Size: リソースの内容のサイズ（バイト）
* Hash: リソースの内容のsha1ハッシュ
* Synced: （内部使用のみ）

## リソースの一覧表示

すべてのリソースのリストを取得するには、GETを実行してください/api/resources。結果は次のようになります。

```
[{"UID":1,"GUID":"2332866c-9b8d-469f-bf40-de9fad828362","LastModified":"2018-03-07T15:19:10.945117816-07:00","VersionNumber":0,"GroupACL":[3,7],"Global":false,"ResourceName":"newresource","Description":"Description of the resource","Size":0,"Hash":"","Synced":true},{"UID":1,"GUID":"66f7be7d-893b-4dc4-b0ad-3609b348385d","LastModified":"2018-02-12T11:06:44.215431364-07:00","VersionNumber":1,"GroupACL":[1],"Global":false,"ResourceName":"test","Description":"test resource","Size":543,"Hash":"zkTmUEV+AR6JZdqhobIeYw==","Synced":true}]
```

この例では、 "newresource"（GUID 2332866c-9b8d-469f-bf40-de9fad828362）と "test"（GUID 66f7be7d-893b-4dc4-b0ad-3609b348385d）の2つのリソースを示しています。

## リソースを作成する

リソースを作成するには、上でPOSTリクエストを実行/api/resourcesし、次の形式でJSON構造を送信します。

```
{
	"GroupACL": [3,7],
	"Global": false,
	"ResourceName": "newresource",
	"Description": "Description of the resource"
}
```

注：この構造は、ユーザーが設定できるフィールドを含むメタデータ構造のサブセットです。

サーバーは新しく作成されたリソースのリソースメタデータ構造で応答します。

```
{"UID":1,"GUID":"2332866c-9b8d-469f-bf40-de9fad828362","LastModified":"2018-03-07T15:19:10.945117816-07:00","VersionNumber":0,"GroupACL":[3,7],"Global":false,"ResourceName":"newresource","Description":"Description of the resource","Size":0,"Hash":"","Synced":false}
```

## リソース内容の設定

新しく作成されたリソースにはデータが含まれていません。リソースの内容を変更するには、にPUT要求を発行し、リソースの適切なGUIDに/api/resources/{guid}/raw置き換え{guid}ます。したがって、上記で作成したリソースの内容を設定するには、PUT onを実行し/api/resources/2332866c-9b8d-469f-bf40-de9fad828362/rawます。サーバーは新しい変更時刻、サイズ、およびハッシュを示す更新されたメタ構造で応答します。

## リソースの内容を読む

リソースの内容を読み取るには、単にリソースの適切なGUIDに/api/resources/{guid}/raw置き換え{guid}てGETリクエストを実行します。

## リソースメタデータの閲覧と更新

単一のリソースのメタデータを読み取るには、GETリクエストを実行し/api/resources/{guid}ます。たとえば、GET on /api/resources/2332866c-9b8d-469f-bf40-de9fad828362は次のようになります。

```
{"UID":1,"GUID":"2332866c-9b8d-469f-bf40-de9fad828362","LastModified":"2018-03-07T15:29:10.557490321-07:00","VersionNumber":1,"GroupACL":[3,7],"Global":false,"ResourceName":"newresource","Description":"Description of the resource","Size":6,"Hash":"QInZ92Blt3TopFBeBTD0Cw==","Synced":true}
```

PUTリクエストを実行してメタデータを変更できます/api/resources/{guid}。要求の内容は、GET要求で読み取られた構造で、必要なフィールドが変更されている必要があります。たとえば、 "newresource"の説明を変更するには、次の内容でPUTを実行します。

```
{"UID":1,"GUID":"2332866c-9b8d-469f-bf40-de9fad828362","LastModified":"2018-03-07T15:29:10.557490321-07:00","VersionNumber":1,"GroupACL":[3,7],"Global":false,"ResourceName":"newresource","Description":"A new description for the resource!","Size":6,"Hash":"QInZ92Blt3TopFBeBTD0Cw==","Synced":true}
```

<span style="color: red; ">注：GroupACL、ResourceName、およびDescriptionの各フィールドだけが、通常のユーザーが変更できます。管理者ユーザーはグローバルフィールドも変更できます。その他の変更されたフィールドはサーバーによって無視されます。</span>

## リソースを削除する

リソースを削除するには、単純にDELETE要求を発行し、通常どおりリソースの適切なGUIDに/api/resources/{guid}置き換え{guid}ます。

## 管理者アクション

管理者ユーザーはシステム上のすべてのリソースを表示する必要があるかもしれません。管理者ユーザーは、GET要求を使用してシステム内のすべてのリソースのグローバルリストを取得できます/api/resources?admin=true。

リソースGUIDはシステム全体で一意であるため?admin=true、パラメータを追加してもエラーになることはありませんが、管理者は指定しなくてもリソースを変更、削除、または取得できます。