# Scheduled Searches

このAPIはスケジュール検索の作成と管理を可能にします。検索は、ランダムに生成されたIDによって参照されます。

## スケジュール検索構造

スケジュール検索には、次の関心分野が含まれています。

* ID: スケジュール検索のID
* GUID: この特定の検索のためのユニークなID。作成時に空白のままにすると、ランダムなGUIDが割り当てられます（これは標準的な使用例です）。
* Owner: 検索の所有者のUID
* Groups: この検索の結果を見ることを許可されているグループのリスト
* Name: この予約検索の名前
* Description: スケジュールされた検索の説明文
* Schedule: いつ実行するかを指定するcron互換の文字列
* Permissions: パーミッションビットを格納するために使用される64ビット整数
* Disabled: trueに設定されている場合、スケジュールされた検索の実行を妨げるブール値。
* OneShot: ブール値。trueに設定すると、無効にしない限り、スケジュール検索をできるだけ早く実行します。
* LastRun: このスケジュール検索が最後に実行された時刻
* LastRunDuration: 最後の実行にかかった時間
* LastSearchIDs: このスケジュール検索から最後に実行された検索の検索IDを含むストリングの配列
* LastError: この検索の最後の実行から発生したエラー

検索が「標準」スケジュール検索の場合、これらのフィールドも設定されます。

* SearchString: 実行するGravwellクエリ
* Duration: 検索を実行するまでの時間を指定する秒単位の値。これは負の値でなければなりません。
* SearchSinceLastRun: ブール値。設定されていると、Durationフィールドは無視され、代わりにLastRunから現在までの間に検索が実行されます。

一方、検索がスクリプトの場合は、次のフィールドが設定されます。

* Script: ankoスクリプトを含む文字列

## ユーザーコマンド

このセクションのAPIコマンドはどのユーザでも実行できます。

### スケジュール検索の一覧表示

ユーザーに表示されている（ユーザーが所有しているか、ユーザーのグループの1つにアクセス可能とマークされている）すべてのスケジュールされた検索のリストを取得するには、GETを実行し/api/scheduledsearchesます。結果は次のようになります。

```
[{"ID":1439174790,"GUID":"efd1813d-283f-447a-a056-729768326e7b","Groups":null,"Name":"count","Description":"count all entries","Owner":1,"Schedule":"* * * * *","Permissions":0,"Updated":"2019-05-21T16:01:01.036703243-06:00","Disabled":false,"OneShot":false,"Synced":true,"SearchString":"tag=* count","Duration":-3600,"SearchSinceLastRun":false,"Script":"","PersistentMaps":{},"LastRun":"2019-05-21T16:01:00.013062447-06:00","LastRunDuration":1015958622,"LastSearchIDs":["672586805"],"LastError":""}]

```

この例は、UID 1（admin）が所有する「count」という名前の単一のスケジュール検索を示しています。毎分実行さtag=* countれ、過去1時間の検索が実行されます。

### スケジュール検索の作成

新しいスケジュール検索を作成するに/api/scheduledsearchesは、スケジュール検索に関する情報を含むJSON構造でPOSTリクエストを実行します。標準検索を作成するには、この例のようにSearchStringフィールドとDurationフィールドを必ず入力します。この例では、毎日午前8時に過去24時間にわたって検索が実行されます。

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

あるいは、SearchSinceLastRunフィールドがtrueに設定されている場合、検索エージェントはDuration（この新しい検索の最初の実行を除く）を無視し、代わりに最後の実行から現在までの時間にわたって検索を実行します。

スクリプトを使用してスケジュール検索を作成するには、[SearchString]および[Duration]フィールドの代わりに[Script]フィールドを入力します。両方が設定されている場合は、スクリプトが優先されます。

サーバーは新しいスケジュール検索のIDで応答します。

### 特定のスケジュール検索を取得する

単一のスケジュールされた検索に関する情報は、GETをオンにしてアクセスできます/api/scheduledsearches/{id}。たとえば、1353491046というスケジュールされた検索IDがあるとする/api/scheduledsearches/1353491046と、次のクエリを実行して受信します。

```
{"ID":1439174790,"GUID":"efd1813d-283f-447a-a056-729768326e7b","Groups":null,"Name":"count","Description":"count all entries","Owner":1,"Schedule":"* * * * *","Permissions":0,"Updated":"2019-05-21T16:01:01.036703243-06:00","Disabled":false,"OneShot":false,"Synced":true,"SearchString":"tag=* count","Duration":-3600,"SearchSinceLastRun":false,"Script":"","PersistentMaps":{},"LastRun":"2019-05-21T16:01:00.013062447-06:00","LastRunDuration":1015958622,"LastSearchIDs":["672586805"],"LastError":""}
```

スケジュールされた検索はGUIDによっても取得できます。 これはWebサーバのためにより多くの作業を必要とし、必要なときにだけ使用されるべきであることに注意してください。 上記のスケジュール検索を取得するには、/ api / schedulesearches / cdf011ae-7e60-46ec-827e-9d9fcb0ae66dでGETを実行します。

### 既存の検索を更新する

スケジュール検索を変更するには、HTTP PUTを実行/api/scheduledsearches/{id}して、必要な変更を含む更新された構造を含めるようにします。変更されていないフィールドもプッシュするように注意してください。そうしないと、空の値で上書きされます。

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

スクリプトフィールドを空にしてSearchStringとDurationを押すことで、スクリプトスケジュール検索を標準のスケジュール検索に変更できます。同様に、標準のスケジュール検索は、スクリプトフィールドをプッシュしてSearchStringを空に設定することでスクリプトスケジュール検索に変換できます。

### スケジュールされた検索エラーを解決する

スケジュールされた検索構造体のLastErrorフィールドは、エラーが発生した場合に設定され、その後の正常な実行によってクリアされることはありません。それはのDELETEによって手動でクリアすることができます/api/scheduledsearches/{id}/error

### スケジュール検索の削除

既存のスケジュール検索は、DELETEを実行することで削除できます/api/scheduledsearches/{id}。

## 管理コマンド

次のコマンドは管理者ユーザーだけが利用できます。

### すべての検索を一覧表示する

管理ユーザーは、システム上のすべてのスケジュール検索を表示する必要がある場合があります。管理者ユーザーは、GETリクエストを使用して、システム内のすべてのスケジュールされた検索のグローバルリストを取得できます/api/scheduledsearches?admin=true。

スケジュールされた検索IDはシステム全体で一意であるため、管理者は指定しなくても検索を変更、削除、取得できます?admin=trueが、パラメータを追加してもエラーにはなりません。

### 特定のユーザーの検索を取得する

上/api/scheduledsearches/user/{uid}でGETを実行するuidと、数字のユーザーIDはそのユーザーに属するすべての検索の配列を取得します。

### 特定のユーザーの検索をすべて削除する

DELETEを実行する/api/scheduledsearches/user/{uid}と、指定したユーザーに属するすべてのスケジュール検索が削除されます。
