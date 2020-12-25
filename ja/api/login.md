# ログインサブシステム

## ログイン

ログインするには、以下の構造で `/api/login` に JSON を POST します。

```
{
    "User": "username",
    "Pass": "password"
}
```

入力すると、サーバはログインが成功したかどうかを示すために以下のような応答をします。

```
{
  "LoginStatus":true,
  "JWT":"reallylongjsonwebtokenstringishere"
}

```

ログインに失敗した場合、サーバは "reason"プロパティを持つ構造体を返します。
```
{
  "LoginStatus":false,
  "Reason":"Invalid username or password"
}
```

JSONを送信する代わりに、ログインPOSTリクエストでフォームフィールド「User」と「Pass」を設定することもできます。

## ログアウト

* PUT /api/logout - 現在のインスタンスをログアウトします。
* DELETE /api/logout - すべてのユーザのインスタンスをログアウトします。

## すべての POST に JWT プロテクションが適用される
ログインAPIから受信したJWTは、他のすべてのAPIリクエストでAuthorization Bearerヘッダーとして含まれている必要があります。

```Authorization: Bearer reallylongjsonwebtokenstringishere```

## アクティブなセッションを見る
GETを `/api/users/{id}/sessions` に送ると、JSONの塊を返します。 管理者は任意のユーザーのセッションをリクエストすることができますが、ユーザーは自分のセッションのみをリクエストすることができます。

```
{
    "Sessions": [
        {
            "LastHit": "2020-08-04T15:28:12.601899275-06:00",
            "Origin": "127.0.0.1",
            "Synced": false,
            "TempSession": false
        },
        {
            "LastHit": "2020-08-03T23:59:53.807610997-06:00",
            "Origin": "127.0.0.1",
            "Synced": false,
            "TempSession": false
        },
        {
            "LastHit": "2020-08-04T09:45:48.291770859-06:00",
            "Origin": "127.0.0.1",
            "Synced": false,
            "TempSession": false
        }
    ],
    "UID": 1,
    "User": "admin"
}
```
