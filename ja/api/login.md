# Login

## ログイン

このページのGETは/api/login/login.htmlにリダイレクトします。

ログインするには、JSONを/ api / login /に次のような構造でPOSTします。
```
{
    "User": "username",
    "Pass": "password"
}
```

そして、サーバーはログインが成功したかどうかを示すために以下で応答します。

```
{
  "LoginStatus":true,
  "JWT":"reallylongjsonwebtokenstringishere"
}

```

ログインが失敗した場合、サーバーは "reason"プロパティを持つ構造体を返します。
```
{
  "LoginStatus":false,
  "Reason":"Invalid username or password"
}
```

## ログアウト

* PUT / api / logout - 現在のインスタンスをログアウトします
* DELETE / api / logout - すべてのユーザーインスタンスをログアウトします

## JWT保護はすべてのPOSTに適用されます
ログインAPIから受け取ったJWTは、他のすべてのAPI要求でAuthorization Bearerヘッダーとして含める必要があります。

```Authorization: Bearer reallylongjsonwebtokenstringishere```

## アクティブセッションを表示する
/ api / account / {id} / sessionsを取得すると、大量のJSONが返されます。管理者は任意のユーザーのセッションを要求でき、ユーザーは自分のセッションのみを要求できます。

```
{
     User:  "user",
     [
          {
              Origin: "192.168.4.2",
              LastHit: "2014-12-1 12:32:02.2332422Z07:00"
          },
          {
              Origin: "1.35.2.2",
              LastHit: "2014-12-1 12:05:02.2332422Z07:00"
          },
          {
              Origin: "10.0.0.22",
              LastHit: "2014-11-1 12:05:02.2332422Z07:00"
          }
     ]
}
```
