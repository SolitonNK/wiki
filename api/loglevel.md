# Logging

このAPIは、管理ユーザーがロギングを管理したり、Gravwellのオンディスクログファイルにログエントリを挿入したりするためのユーティリティを提供します。

## Webサーバーのログレベルを表示/設定する

このAPIにより、管理者は現在のログレベルと利用可能なログレベルを表示できます。
管理者はこのAPIを介してログレベルを自由に変更できます。

現在のログレベルと利用可能なログレベルを取得するには、/ api / loggingに対してGETリクエストを実行します。
このリクエストは以下を返します。

```
{
     Levels: ["Error", "Warn", "Off"],
     Current: "Error"
}
```

ログレベルを設定するには、次のように/ api / logging /にPUTを実行します。

```
{
     Level: "Error"
}
```

## ログを記録する

管理ユーザーはPOST要求を適切なURLに送信することでログエントリを挿入できます。これらのログはGravwell Webサーバーのディスク上のログファイルに書き出されます。

URLは以下のとおりです。

```
/api/logging/access
/api/logging/info
/api/logging/warn
/api/logging/error
```

POSTリクエストの本文には、希望するログメッセージを含む 'Body'という名前のフィールドを持つJSON構造体を含める必要があります。

```
{
	"Body": "This is my error message"
}
```

成功した場合、サーバーはブール値 'true'を返します。
