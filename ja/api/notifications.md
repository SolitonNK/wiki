# Notifications REST API
通知は基本的にメッセージであり、潜在的にはタイプです。このタイプは、何かを複製することなく継続的に通知を送信できるようにするために使用されます。たとえば、Webサーバーはタイプ0xDEADBEEFで通知を追加し続けることができます（インデクサXは停止しています）。通知エンジンは、同じTypeを持つ古い通知を置き換え続けるだけです。

通知には、ターゲット型とブロードキャスト型の2つの形式があります。ブロードキャストは、適切なUIDまたはGIDを持つユーザーのみを対象とし、全員に配信されます。ブロードキャスト通知には、UID / GIDが本質的にありません（これらは常にゼロになります）。

通知は有効期限日に期限切れ（削除）になります。

将来のIgnoreUntil日付の通知はリクエストに戻ってこないでしょう（隠されています）

空白の日付（Sent、Expires、IgnoreUntil）を使用して通知が追加されると、それらにはデフォルト値が入力されます。

次の通知を検討してください。

```
{"UID":7,"GID":0,"Sender":7,"Type":1,"Broadcast":false,"Sent":"2019-04-22T21:44:01.776942432Z","Expires":"2019-04-23T03:44:01.776918756-06:00","IgnoreUntil":"0001-01-01T00:00:00Z","Msg":"foobar","Origin":"00000000-0000-0000-0000-000000000000"}
```

この通知はUID 7のユーザーを対象としており、特定のグループを対象としていません。同じユーザーによって作成されました。タイプは '1'です。ブロードキャスト通知ではありません。それは21:44 UTCに送信され、12時間後に期限切れになります。メッセージは「foobar」です。

すべての通知は一時的です。Webサーバ/フロントエンドが再起動すると、それらは失われます。

すべての通知にはIDがあり、各IDは単調に増加し、常に10進数のuint64として表されます。

基本的なREST APIのURLは次のとおりです。
```
/api/notifications/all/{id:[0-9]+}
/api/notifications/{id:[0-9]+}
/api/notifications/broadcast
/api/notifications/targeted
/api/notifications/targeted/user/{id:[0-9]+}
/api/notifications/targeted/group/{id:[0-9]+}
/api/notifications/targeted/self
```

リクエストのメソッド
GET - 通知を引き戻す
PUT - 特定の通知を更新する
POST - 新しい通知を追加する
DELETE - 特定の通知を削除する

### 通知ルール
* /api/notifications/targeted/selfAPI を介して以外は管理者だけが新しい通知を追加できます。
* ユーザーは、自分が明示的に所有している（UIDが一致する）通知のみを更新できます。GIDの一致が十分ではありません
* ユーザーは通知に関連付けられたUIDまたはGIDを変更できません
* 自分が所有していない通知をユーザーが削除することはできません（UIDが一致しません）

## 新しい通知を追加する

管理者は、通知構造をPOSTすることによってブロードキャスト通知を作成し/api/notifications/broadcastたり、POSTによってターゲット通知を作成することができます/api/notifications/targeted。

管理者以外のユーザーはPOSTを使って通知を作成できます/api/notifications/targeted/self。Type、Msg、およびExpiresフィールドのみが優先されることに注意してください。これらの通知はデフォルトで12時間の有効期限があります。[有効期限]フィールドが過去または過去24時間以上の場合は、現在時刻から12時間に設定されます。

## 通知を受け取る
ユーザーは、/ api / notifications / {id}をIDなしで押すことで、すべての通知を受け取ることができます（例：/ api / notifications /）。特定のIDの後にすべての通知を受け取るには、/ api / notifications / {id}にGETを発行します。たとえば、通知を引き戻して最大のIDが10の場合、/ api / notifications / 10を要求すると、ユーザーがアクセスできる新しい通知のみが取得されます。

### すべての通知を要求している管理者
管理者は、/ api / notifications / all /にIDなしでGETを発行することで、すべての通知を取得できます（UIDまたはGIDに関係なく）。