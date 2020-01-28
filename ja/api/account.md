## Account controls
このページでは、ユーザーやグループとのやり取りに使用されるAPIについて説明します。

## 接続テスト
### バックエンドとの接続を確認することができます
基本的に/api/testに対するGETは、内容のない200を返します。ものすごく単純。

## バージョンAPI
### パスは/api/version/です。
GETを実行するとバージョン情報が表示されます。認証は必要ありません。
```
{"Version":0.1,"Date":"0001-01-01T00:00:00Z"}
```
## アカウントと活動を一覧表示する
 /api/user/サブディレクトリは、管理者がアカウント情報を取得したり、アカウントの利用状況を確認したりできるようにするためのものです。

##すべてのアカウントを一覧表示する
管理者は /api/user/に対してGET要求を実行でき、バックエンドはすべてのアカウントの要約を含むJSONパケットを返します。返されるJSONは次のとおりです。
```
{[
    {
        UID: 1
        User: "Administrator" 
        Name: "I am the law" 
        Email: "admin@example.com" 
        Admin: true
        Locked: false
        TS: "2009-11-01 17:45:02.1000 +0000 UTC" 
    },
    {
        UID: 2
        User: "paco" 
        Name: "Paco Chingato" 
        Email: "paco@gmail.com" 
        Admin: false
        Locked: true
        TS: "2014-05-01 01:31:33.1234 +0000 UTC" 
    }
]}
```
## アカウント管理
管理者はアカウントを変更するために/api/user/を操作できます。バックエンドは常にコマンドが成功したかどうかを示すJSONパケットと可能な情報エラーメッセージで応答します。失敗するとフロントエンドはエラーメッセージをユーザーに表示します（人間向けにフォーマットされています）。適切なエラーメッセージが応答の本文に表示されます。StatusOK（200）は良いリクエストで送信され、400-500ステータスはエラーで送信されます（どのエラーとなぜ）。

管理者アカウントのユーザー名を変更または削除することはできません。また、管理者アカウントを管理者ステータスから降格することもできません。

## 新しいユーザーを追加する
新しいユーザーを追加するには、フロントエンドは次のstruct/JSONを /api/user/にPOSTする必要があります。
```

{
     User: "chuck",
     Pass: "chuckTesta4Eva",
     Name: "Chuck Testa",
     Email: "chuck@testa.net" 
     Admin: true,
}
```
すべてのフィールドに入力する必要があります。バックエンドは上記のように標準レスポンスJSONでレスポンスします。

ユーザーアカウントをロックする

アカウントをロックするには、フロントエンドはロックされるユーザーのUIDの{id}とともに空のPOSTを /api/user/{id}/lockに送信する必要があります。ユーザーアカウントをロックすると、セッションがアクティブであっても、バックエンドへの新しい接続は即座に停止されます。

ユーザーアカウントのロック解除
アカウントのロックを解除するには、フロントエンドはロックを解除するユーザーのUIDの{id}を付けて/api/user/{id}/lock with the {id}に空のDELETEを送信する必要があります。アクションが許可されている場合、アカウントが実際にロック解除されたかどうかにかかわらず、バックエンドは正常に応答します。これは、ロックされたアカウントをロックすると、そのアカウントがロックされた状態で終了するためです。だからそれはすべて良いです。ロック解除と同じです。

## ユーザー情報を変更する
ユーザ情報を変更するには、フロントエンドはPUT /api/user/{id}/をアカウントJSONに置き換えて送信します。
```
{
     User: "chuck",
     Name: "Chuck Testa",
     Email: "chuck@testa.net" 
}
```
入力されていないフィールドは無視されます。バックエンドは上記のように標準レスポンスJSONでレスポンスします。現在のユーザーが管理者ではなく、自分のアカウントを変更していない場合、リクエストは拒否されます。管理者は任意のアカウントの情報を変更できます。プライマリ管理者アカウント（UIDゼロ）はその管理ステータスを変更できません。バックエンドは、誰がエラーの原因となったのかに応じて、成功時に200、エラー時に400〜500で応答します。エラーメッセージはレスポンスの本文に返され、人間が表示できます。

## ユーザーのパスワードを変更する
パスワードを変更するには、フロントエンドはJSONをurl /api/user/{id}/pwdに設定する必要があります。ユーザーが管理者でアカウントのパスワードを変更しない場合、OrigPassフィールドは必要ありません。
```
{
     OrigPass: "my old password was bad",
     NewPass: "thisis mynewpassword",
}
```
## ユーザーを削除する
ユーザを削除するには、フロントエンドは削除するUIDのIDを付けてDELETEを/api/user/{id}/に送信する必要があります。この機能を使用できるのは管理者だけです。ユーザーは自分のアカウントを削除することはできません。1次管理者（UIDゼロ）は削除できません。

## 管理ステータスを変更する
ユーザの管理ステータスを変更したり、管理ステータスを問い合わせたりするには、フロントエンドは/api/user/{id}/adminに空のボディを付けてメソッドにアクションを指定する必要があります。バックエンドは、成功した場合は200、エラーが発生した場合は400から500で応答を返します。
```
GET - returns current admin status
PUT - sets the user as an admin and returns the new status
DELETE - removes admin status for the user

//example JSON on success
{
     UID: 1,
     Admin: true,
}
```
## シングルユーザーアカウント情報の取得
シングルユーザーアカウント情報を取得するには、フロントエンドは/api/user/{id}/にGETを送信する必要があります。管理者は任意のアカウントを取得でき、管理者以外は自分のアカウント情報のみを取得できます。200のレスポンスはボディに有効なJSONを含み、400-500は誰かがリクエストをボーンしたことを意味し、ボディはエラーメッセージを含みます。バックエンドは次のようにアカウント情報を含むJSONパケットで応答します。
```
//the JSON returned on success
{
    Status: bool,
    Reason: string
    Info: {
        UID: int,
        User: string,
        Name: string,
        Email: string,
        Admin: bool,
        Locked: bool,
        TS: string
    }
}

//an example successful response
{
    Status: true,
    Reason: "",
    Info: {
        UID: 5,
        User: "ChuckTesta",
        Name: "Chuck Testa",
        Email: "ChuckTesta@gmail.com",
        Admin: false,
        Locked: false,
        TS: "05-12-2014 13:05:01.2242 +0000 UTC" 
    }
}
```
## アカウントとグループの情報
情報URLは、認証されたユーザーなら誰でも攻撃できます。URLを押すと現在のユーザー情報が得られます。

## アカウントユーザー情報のベースURL /api/info/whoami
/api/info/whoamiでGETを実行すると、現在のアカウント情報が返されます。
```
{
        "UID": 1,
        "User": "admin",
        "Name": "Sir changeme of change my password the third",
        "Email": "admin@admin.admin",
        "Admin": true,
        "Locked": false,
        "TS": "2016-12-05T17:05:33.121180268-07:00",
        "DefaultGID": 0,
        "Groups": [
                {
                        "GID": 1,
                        "Name": "TheNinos",
                        "Desc": "This is the Ninos group" 
                }
        ]
}
```
