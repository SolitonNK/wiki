# スケジュールされた検索API

本APIは、スケジュールされた検索の作成と管理を可能にします。検索はランダムに生成されたIDで参照されます。

## スケジュールされた検索の構造

スケジュール検索には、以下の重要なフィールドが含まれています。

* ID: 予定されている検索のID
* GUID: この特定の検索のための一意の ID です。作成時に空白のままにしておくと、ランダムな GUID が割り当てられます (これは標準的な使用例です)
* Owner: 検索の所有者の uid
* Groups: この検索の結果を見ることができるグループのリスト
* Name: この予定されている検索の名前
* Description: 予定されている検索のテキストによる説明
* Schedule: いつ実行するかを指定する cron 互換の文字列
* Permissions: パーミッションビットを格納するために使用される 64 ビット整数
* Disabled: trueに設定されている場合、スケジュールされた検索が実行されないようにします。
* OneShot: ブール値で、trueに設定すると、スケジュールされた検索を可能な限り早く一度だけ実行します。
* LastRun: このスケジュールされた検索が最後に実行された時刻
* LastRunDuration: 最後の実行にかかった時間
* LastSearchIDs: このスケジュールされた検索から最近実行された検索の検索IDを含む文字列の配列
* LastError: この検索の最後の実行の結果として発生したエラー

検索が「標準」スケジュール検索の場合、これらのフィールドも設定されます:

* SearchString: 実行するためのGravwellクエリ
* Duration: 検索を実行するまでの距離を指定する秒単位の値。これは負の値でなければなりません。
* SearchSinceLastRun: ブール値です。設定されている場合、Durationフィールドは無視され、代わりにLastRun時間から現在まで検索が実行されます。

一方、検索がスクリプトの場合は、以下のフィールドを設定します。

* Script: ankoスクリプトを含む文字列

## ユーザーコマンド

このセクションのAPIコマンドは、任意のユーザが実行することができます。

### 予定されている検索のリストアップ

ユーザーが閲覧可能なすべてのスケジュール検索のリストを取得するには、`/api/scheduledsearches`でGETを実行します(ユーザーが所有しているか、ユーザーのグループの1つにアクセス可能とマークされているかのいずれか)。結果は次のようになります。

```
[{"ID":1439174790,"GUID":"efd1813d-283f-447a-a056-729768326e7b","Groups":null,"Name":"count","Description":"count all entries","Owner":1,"Schedule":"* * * * *","Permissions":0,"Updated":"2019-05-21T16:01:01.036703243-06:00","Disabled":false,"OneShot":false,"Synced":true,"SearchString":"tag=* count","Duration":-3600,"SearchSinceLastRun":false,"Script":"","PersistentMaps":{},"LastRun":"2019-05-21T16:01:00.013062447-06:00","LastRunDuration":1015958622,"LastSearchIDs":["672586805"],"LastError":""}]
```

この例では、UID 1 (admin)が所有する "count "という名前の単一のスケジュール検索を示しています。これは1分ごとに実行され、過去1時間の間に検索 `tag=* count` を実行します。

### スケジュールされた検索の作成

新しいスケジュール検索を作成するには、スケジュール検索に関する情報を含むJSON構造体で `/api/scheduledsearches` にPOSTリクエストを実行します。標準の検索を作成するには、SearchStringフィールドと Durationフィールドを必ず入力してください。

```
{
	"Name": "myscheduledsearch",
	"Description": "a scheduled search",
	"Groups": [2],
	"Schedule": "0 8 * * *",
	"SearchString": "tag=default grep foo",
	"Duration": -86400,
	"SearchSinceLastRun": false
}
```

別の方法として、SearchSinceLastRunフィールドがtrueに設定されている場合、検索エージェントはDurationを無視し（この新しい検索の最初の実行を除く）、代わりに最後の実行から現在の時刻までの時間に渡って検索を実行します。

スクリプトを使用してスケジュール検索を作成するには、「SearchString」と「Duration」フィールドの代わりに「Script」フィールドを入力します。両方が入力されている場合、スクリプトが優先されます。

スケジュールされた検索は、ユーザーの準備が整うまで実行されないように、Disabledフラグをtrueに設定して作成することができます。また、OneShotフラグをtrueに設定して作成することもできます。

サーバは、新たにスケジュールされた検索のIDで応答します。

### 特定のスケジュールされた検索の取得

単一のスケジュール検索に関する情報は、`/api/scheduledsearches/{id}`をGETすることでアクセスできます。例えば、スケジュールされた検索IDが1439174790であるとすると、`/api/scheduledsearches/1439174790`に問い合わせて、次のような情報を受け取ることになります。

```
{"ID":1439174790,"GUID":"efd1813d-283f-447a-a056-729768326e7b","Groups":null,"Name":"count","Description":"count all entries","Owner":1,"Schedule":"* * * * *","Permissions":0,"Updated":"2019-05-21T16:01:01.036703243-06:00","Disabled":false,"OneShot":false,"Synced":true,"SearchString":"tag=* count","Duration":-3600,"SearchSinceLastRun":false,"Script":"","PersistentMaps":{},"LastRun":"2019-05-21T16:01:00.013062447-06:00","LastRunDuration":1015958622,"LastSearchIDs":["672586805"],"LastError":""}
```

スケジュールされた検索はGUIDでも取得できます。これはウェブサーバに多くの負荷がかかるので、必要な場合にのみ使用すべきであることに注意してください。上記のスケジュール検索を取得するには、`/api/scheduledsearches/cdf011ae-7e60-46ec-827e-9d9fcb0ae66d`でGETしてください。

### 既存の検索の更新

スケジュール検索を変更するには、`/api/scheduledsearches/{id}`にHTTP PUTを行い、変更したい内容を含む構造体を更新してください。変更されていないフィールドもプッシュするように注意してください。

以下のフィールドを更新することができます。

* Name
* Description
* Schedule
* SearchString
* Duration
* SearchSinceLastRun
* Script
* Groups
* Disabled
* OneShot

スクリプト・フィールドを空にしてSearchStringとDurationを押すことで、スクリプト・スケジュール検索を標準のスケジュール検索に変更できます。同様に、標準スケジュール検索は、スクリプト・フィールドをプッシュしてSearchStringを空に設定することで、スクリプト・スケジュール検索に変換できます。

### スケジュール検索のエラーのクリア

スケジュールされた検索構造体のLastErrorフィールドは、エラーが発生した場合に設定され、その後の成功した実行ではクリアされません。これは手動で `/api/scheduledsearches/{id}/error` でDELETEすることでクリアできます。

### スケジュールされた検索の永続的な状態のクリア

`/api/scheduledsearches/{id}/state` を削除すると、LastErrorフィールドとスケジュールされた検索の永続マップの両方がクリアされます。これにより、不正なスクリプトが原因で状態が壊れてしまった場合に、スケジュール検索をリセットすることができます。

### スケジュールされた検索の削除

既存のスケジュール検索は、`/api/scheduledsearches/{id}`でDELETEを実行することで削除することができます。

## 管理者コマンド

以下のコマンドは管理者のみが使用できます。

### すべての検索結果のリストアップ

管理者ユーザーは、システム上のすべてのスケジュールされた検索を表示する必要がある場合があります。管理者ユーザーは、`/api/scheduledsearches?admin=true`のGETリクエストで、システム上のすべてのスケジュールされた検索のグローバルリストを取得することができます。

スケジュールされた検索IDはシステム全体で一意であるため、管理者は `?admin=true` を指定しなくても検索の変更/削除/検索を行うことができますが、不必要にパラメータを追加してもエラーにはなりません。

### 特定のユーザーの検索結果の取得

`uid` が数値のユーザIDである`/api/scheduledsearches/user/{uid}`をGETすると、そのユーザに属するすべての検索結果の配列を取得します。

### 特定のユーザーの検索をすべて削除する

DELETEを実行すると、`/api/scheduledsearches/user/{uid}` に属するすべてのスケジュール検索が削除されます。

### スケジュールされた検索のテスト解析の実行

Scheduledsearches APIは、スケジュールされた検索を保存する前にテストするためのAPIを提供します。 Parse APIは `/api/scheduledsearches/parse` にあり、PUTリクエストでアクセスすることができます。 認証されたユーザは、既存のスケジュールスクリプトを保存したり変更したりすることなく、 解析やチェックを行うスケジュールスクリプトを送信することができます。

解析を実行するには、次のJSON構造体をPUTリクエスト `/api/scheduledsearches/parse` の本文で送信します。

```
{
	Script string
}
```

APIは次のJSON構造で応答します。
```
{
	OK bool
	Error string
	ErrorLine int
	ErrorColumn int
}
```

スクリプトが解析テストに合格した場合、レスポンスのOKフィールドには `true` が格納されます。 エラーフィールドは省略され、ErrorLineフィールドとErrorColumnフィールドはともに `-1` となります。 提供されたスクリプトが正しく解析できなかった場合、OKフィールドは `false` となり、Errorフィールドには失敗の理由が、ErrorLineとErrorColumnにはスクリプトのどこでエラーが発生したかが示されます。

ErrorLine フィールドと ErrorColumn フィールドは、常に入力されているとは限りません。 値 -1 は、スクリプト解析システムがスクリプトのどこにエラーがあるのかわからないことを示しています。

以下にリクエストとレスポンスの例を示します。

#### 有効なスクリプト
リクエスト
```
{
	"Script":"fmt = import(\"fmt\")\nfmt.Println(\"Hello\")\nfmt.Sstuff(\"Goodbye\")\n"
}
```

レスポンス
```
{
	"OK":true,
	"ErrorLine":-1,
	"ErrorColumn":-1
}
```

#### 無効なスクリプト
リクエスト
```
{
	"Script":"fmt = import(\"fmt\")\nfmt.Println(\"Hello\")\nfmt.Sstuff(\"Goodbye)\n"
}
```

レスポンス
```
{
	"OK":false,
	"Error":"syntax error",
	"ErrorLine":3,
	"ErrorColumn":21
}
```
