# システム管理

このページでは、Gravwellのプロセスと設定を管理するための管理者専用APIについて説明します。

## Gravwellの再起動

Gravwellのプロセスを再起動するために2つのAPIが提供されています。一つはウェブサーバの再起動、もう一つはインデクサーの再起動です。どちらの場合も、プロセスをシャットダウンし、systemd (または使用中のinitシステム) が再起動できるようにすることで再起動が可能になります。

### ウェブサーバの再起動

ウェブサーバを再起動するには、`/api/restart/webserver`に空のボディでPOSTリクエストを送信してください。これにより、ウェブサーバがシャットダウンしてすぐに再起動するようになります。

### インデクサーの再起動

ウェブサーバが現在接続しているすべてのインデクサを再起動するには、`/api/restart/indexers`に空のボディでPOSTリクエストを送信します。ウェブサーバは各インデクサに対して、自分自身をシャットダウンして再起動するようにシグナルを送ります。個々のインデクサーが再起動すると、ウェブサーバは自動的に再接続します。

### 分散型フロントエンドのチェック

Gravwellクラスタが分散型フロントエンドモードで動作しているかどうかを確認するには、`/api/distributed`でGETを実行してください。 ウェブサーバはフロントエンドが分散モードで設定されているかどうかを示すJSONオブジェクトを返します。

次は、分散モードでない場合の応答例です:

```
{
	"Distributed": false
}
```