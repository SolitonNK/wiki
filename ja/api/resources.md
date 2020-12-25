# API Webリソース

Web API は、リソースへのアクセスを提供します。リソースは、曖昧さを防ぐために GUID で参照する必要があります。

## リソースのメタデータ構造
リソースシステムは、各リソースのメタデータ構造体を保持しています。Web API はこの構造体をJSONでエンコードしたものを通信に使用します。フィールドの大部分は自明ですが、ここでは精度のために説明します:

* UID：リソース所有者のUID
* GUID：リソースの一意の識別子
* LastModified：リソースが最後に更新された時刻
* VersionNumber：リソースの内容が変更されるたびに増分されます。
* GroupACL：リソースへのアクセスが許可されている整数のグループIDのリスト
* Global：trueの場合、リソースはシステム上のすべてのユーザーが読み取ることができます。管理者のみがグローバルリソースを作成できます。
* ResourceName：リソースの名前
* Description：リソースの詳細な説明
* Size：リソースコンテンツのサイズ（バイト単位）
* Hash：リソースコンテンツのsha1ハッシュ
* Synced:(内部使用のみ）

## リソースの一覧表示

すべてのリソースのリストを取得するには、`/api/resources`でGETを実行します。結果は次のようになります。

```
[{"UID":1,"GUID":"2332866c-9b8d-469f-bf40-de9fad828362","LastModified":"2018-03-07T15:19:10.945117816-07:00","VersionNumber":0,"GroupACL":[3,7],"Global":false,"ResourceName":"newresource","Description":"Description of the resource","Size":0,"Hash":"","Synced":true},{"UID":1,"GUID":"66f7be7d-893b-4dc4-b0ad-3609b348385d","LastModified":"2018-02-12T11:06:44.215431364-07:00","VersionNumber":1,"GroupACL":[1],"Global":false,"ResourceName":"test","Description":"test resource","Size":543,"Hash":"zkTmUEV+AR6JZdqhobIeYw==","Synced":true}]
```

この例は、「newresource」（GUID 2332866c-9b8d-469f-bf40-de9fad828362）と「test」（GUID 66f7be7d-893b-4dc4-b0ad-3609b348385d）の2つのリソースを示しています。

## リソースの作成

リソースを作成するには、`/api/resources`でPOSTリクエストを実行し、次の形式でJSON構造を送信します。

```
{
	"GroupACL": [3,7],
	"Global": false,
	"ResourceName": "newresource",
	"Description": "Description of the resource"
}
```

注：構造はメタデータ構造のサブセットであり、ユーザーが設定できるフィールドが含まれています。

サーバーは、新しく作成されたリソースのリソースメタデータ構造で応答します。

```
{"UID":1,"GUID":"2332866c-9b8d-469f-bf40-de9fad828362","LastModified":"2018-03-07T15:19:10.945117816-07:00","VersionNumber":0,"GroupACL":[3,7],"Global":false,"ResourceName":"newresource","Description":"Description of the resource","Size":0,"Hash":"","Synced":false}
```

## リソースコンテンツの設定

新しく作成されたリソースにはデータが含まれていません。リソースのコンテンツを変更するには、マルチパートPUTリクエストを`/api/resources/{guid}/raw`に発行し、`{guid}`をリソースの適切なGUIDに置き換えます。 リクエストには、リソースに保存する必要のあるデータを含む、`file`という名前の1つの部分のみが必要です。 したがって、上記で作成したリソースの内容を設定するには、`/api/resources/2332866c-9b8d-469f-bf40-de9fad828362/raw`でマルチパートPUTを実行します。サーバーは、新しい変更時間、サイズ、およびハッシュを示す更新されたメタ構造で応答します。「maxmind.db」という名前のファイルをリソースにアップロードするcurl呼び出しの例を以下に示します（Bearerトークンはユーザーセッションに適切に設定する必要があることに注意してください。これは単なる例です）。

```
curl 'http://gravwell.example.com/api/resources/2332866c-9b8d-469f-bf40-de9fad828362/raw' -X PUT -H 'Authorization: Bearer 7b22616c676f223a35323733382c22747970223a226a3774227d.7b22756964223a312c2265787069726573223a22323031392d31302d30395431333a33333a32352e3231343632203131352d30363a3030222c22696174223a5b33392c32323c2c35382c36362c3231372c32362c3131392c33362c3234312c33352c39302c312c39312c3138312c3234322c33362c3137342c3139342c3130382c37342c3133382c32362c3133392c3234362c37362c3132352c3136342c38382c39322c39302c3231312c36365d7d.ef9ca1e0ac7f012adcd796d8cca0746a6fabecd7e787c025d754e54a072be5c89dc7bac5f648ae26b422f0bbe6b69a806e8de4a0fe2b7d06d3293ed4c1323daf' -H 'Content-Type: multipart/form-data' -H 'Accept: */*' --form file=@maxmind.db
```

## リソースの内容を読む

リソースのコンテンツを読み取るには、`/api/resources/{guid}/raw`でGETリクエストを実行し、`{guid}`をリソースの適切なGUIDに置き換えます。

## リソースメタデータの読み取りと更新

単一のリソースのメタデータを読み取るには、`/api/resources/{guid}`でGETリクエストを実行します。 たとえば、`/api/resources/2332866c-9b8d-469f-bf40-de9fad828362`のGETは次のようになります。

```
{"UID":1,"GUID":"2332866c-9b8d-469f-bf40-de9fad828362","LastModified":"2018-03-07T15:29:10.557490321-07:00","VersionNumber":1,"GroupACL":[3,7],"Global":false,"ResourceName":"newresource","Description":"Description of the resource","Size":6,"Hash":"QInZ92Blt3TopFBeBTD0Cw==","Synced":true}
```

メタデータは、`/api/resources/{guid}`でPUTリクエストを実行することで変更できます。リクエストの内容は、GETリクエストで読み取られた構造であり、必要なフィールドが変更されている必要があります。たとえば、「newresource」の説明を変更するには、次の内容でPUTを実行します。

```
{"UID":1,"GUID":"2332866c-9b8d-469f-bf40-de9fad828362","LastModified":"2018-03-07T15:29:10.557490321-07:00","VersionNumber":1,"GroupACL":[3,7],"Global":false,"ResourceName":"newresource","Description":"A new description for the resource!","Size":6,"Hash":"QInZ92Blt3TopFBeBTD0Cw==","Synced":true}
```

注：通常のユーザーが変更できるのは、GroupACL、ResourceName、およびDescriptionフィールドのみです。管理者ユーザーは、グローバルフィールドを変更することもできます。その他の変更されたフィールドは、サーバーによって無視されます。

## リソースコンテンツタイプの取得

 `/api/resources/{guid}/contenttype`へのGETリクエストは、リソースの検出されたコンテンツタイプとリソース本体の最初の512バイトを含む構造を返します。

```
{"ContentType":"text/plain; charset=utf-8","Body":"IyBEdW1wcyB0aGUgcm93cyBvZiB0aGUgc3BlY2lmaWVkIENTViByZXNvdXJjZSBhcyBlbnRyaWVzLCB3aXRoCiMgZW51bWVyYXRlZCB2YWx1ZXMgY29udGFpbmluZyB0aGUgY29sdW1ucy4KIyBlLmcuIGdpdmVuIGEgcmVzb3VyY2UgbmFtZWQgImZvbyIgY29udGFpbmluZyB0aGUgZm9sbG93aW5nOgojCWhvc3RuYW1lLGRlcHQKIwl3czEsc2FsZXMKIwl3czIsbWFya2V0aW5nCiMJbWFpbHNlcnZlcjEsSVQKIyBydW5uaW5nIHRoZSBmb2xsb3dpbmcgcXVlcnk6CiMJdGFnPWRlZmF1bHQgYW5rbyBkdW1wIGZvbwojIHdpbGwgb3V0cHV0IDQgZW50cmllcyB3aXRoIHRoZSB0YWcgImRlZmF1bHQiLCBjb250YWluaW5nIGVudW1lcmF0ZWQKIyB2YWx1ZXMgbmFtZWQgImhvc3RuYW1lIiBhbmQgImRlcHQiIHdob3NlIGNvbnRlbnRzIG1hdGNoIHRoZSByb3dzCiMgb2YgdGhlIHJlc291cmNlLgojIEZsYWdzOgojICAtZDogc3BlY2lmaWVzIHRoYXQgaW5jb21pbmcgZW50cmllcyBzaG91bGQgYmUgZHJvcHBlZCA="}
```

バイトパラメータを追加します。例：`/api/resources/{guid}/contenttype?bytes=1024`は、返されるバイト数を変更します。コンテンツタイプの検出を成功させるために、APIは常に少なくとも128バイトを読み取ることに注意してください。指定されたバイト数が128バイト未満の場合、APIはデフォルトで512バイトを読み取ります。

## リソースの削除

リソースを削除するには、`/api/resources/{guid}`でDELETEリクエストを発行し、通常どおり`{guid}`をリソースの適切なGUIDに置き換えます。

## リソースのクローン作成

`/api/resources/{guid}/clone`でPOSTリクエストを発行し、`{guid}`を元のリソースのGUIDに置き換えることで、既存のリソースのクローンを作成できます。リクエストの本文は、新しく作成されたクローンの名前を含むJSON構造である必要があります。

```
{
	"Name": "Copy of Foo"
}
```

サーバーは、新しく複製されたリソースのメタデータで応答します:

```
{
  "UID": 1,
  "GUID": "bbf682f8-363f-4245-8182-d7f6286022ff",
  "Domain": 0,
  "LastModified": "2020-08-19T20:31:11.258257937Z",
  "VersionNumber": 1,
  "GroupACL": null,
  "Global": false,
  "ResourceName": "Copy of Foo",
  "Description": "foobar",
  "Size": 207,
  "Hash": "xfV5dQG4eRe75ULzdb2e2A==",
  "Synced": false,
  "Labels": [
    "blah"
  ]
}
```

## 管理者のアクション

管理ユーザーは、システム上のすべてのリソースを表示する必要がある場合があります。管理者ユーザーは、`/api/resources?admin=true`のGETリクエストを使用して、システム内のすべてのリソースのグローバルリストを取得できます。

リソースのGUIDはシステム全体で一意なので、管理者は`?admin=true`を指定しなくてもリソースを変更/削除/検索することができますが、不必要にパラメータを追加してもエラーにはなりません。