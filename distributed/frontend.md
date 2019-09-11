# 分散Gravwell ウェブサーバー

Gravwellが複数のインデクサーを同時に動作するように設計されているように、同じインデクサーのセットを指す複数のWebサーバーを同時に動作させることもできます。 複数のWebサーバーを使用すると、負荷分散と高可用性が可能になります。 Webサーバーが1つしかない場合でも、データストアを使用して障害が発生したWebサーバーを復元したり、その逆を行うことができるため、データストアを展開すると有用な復元力が得られます。

構成が完了すると、分散Webサーバーはリソース、ユーザー、ダッシュボード、ユーザー設定、およびユーザー検索履歴を同期します。

## データストアサーバー

Gravwellは、データストアと呼ばれる別のサーバープロセスを使用してWebサーバーの同期を保ちます。 それはそれ自身のマシン上で走ることができるか、あるいはウェブサーバーとサーバーを共有することができます。 [ダウンロードページ](#!quickstart/downloads.md)からデータストアインストーラを取得し、データストアを含むマシン上でそれを実行します。

### データストアサーバーの設定

データストアサーバーは、gravwell.confに変更を加えずに実行できる状態になっているはずです。 起動時にデータストアサーバーを有効にしてサービスを開始するには、次のコマンドを実行します。

```
systemctl enable gravwell_datastore.service
systemctl start gravwell_datastore.service
```

#### 高度なデータストア設定

デフォルトでは、データストアサーバーはポート9405を介してすべてのインターフェースをlistenします。何らかの理由でこのコメント解除を変更して/opt/gravwell/etc/gravwell.confファイルに次の行を設定する必要がある場合。

```
Datastore-Listen-Address=10.0.0.5	# listen only on 10.0.0.5
Datastore-Port=9555					# listen on port 9555 instead of 9405
```

## 分散操作用のWebサーバーの構成

Webサーバにデータストアとの通信を開始するように指示するには、Webサーバの/opt/gravwell/etc/gravwell.confの「global」セクションにあるDatastoreフィールドとExternal-Addrフィールドを設定します。 たとえば、データストアサーバがIP 10.0.0.5およびデフォルトのデータストアポートを持つマシン上で実行されていて、設定中のWebサーバが10.0.0.1上で実行されていた場合、エントリは次のようになります。

```
Datastore=10.0.0.5:9405
External-Addr=10.0.0.1:443
```

External-Addrフィールドは、他のWebサーバーがこのWebサーバーと通信するために使用するIPアドレスとポートです。 これにより、あるWebサーバーのユーザーは別のWebサーバーで実行された検索の結果を見ることができます。

注：デフォルトでは、Webサーバは10秒ごとにデータストアにチェックインします。 これは、Datastore-Update-Intervalフィールドを希望の秒数に設定することで変更できます。 更新間の待ち時間が長すぎると、Webサーバ間での変更の伝播が非常に遅くなります。更新が頻繁に行われると、システムに過度の負荷がかかる可能性があります。 5〜10秒が良い選択です。

## 災害からの回復

データストアとWebサーバーで使用される同期技術により、データストアサーバーを再初期化するか交換する場合は注意が必要です。 Webサーバーがデータストアと同期すると、そのデータストアはすべてのトピックで真実であると見なされます。 リソースがデータストアに存在しないが、Webサーバーが以前にそのリソースをデータストアと同期していた場合、Webサーバーはそのリソースを削除します。

データストアは次の場所にデータを格納します。

* `/opt/gravwell/etc/datastore-users.db` (user database)
* `/opt/gravwell/etc/datastore-webstore.db` (dashboards, user preferences, search history)
* `/opt/gravwell/etc/resources/datastore/` (resources)

これらの場所のいずれかが誤って紛失または削除された場合は、データストアを再起動する前にいずれかのWebサーバーシステムから復元する必要があります。 データストアがWebサーバーの1つと同じマシン上にあると仮定して、以下のコマンドを使用します。

```
cp /opt/gravwell/etc/users.db /opt/gravwell/etc/datastore-users.db
cp /opt/gravwell/etc/webstore.db /opt/gravwell/etc/webstore.db
cp -r /opt/gravwell/resources/webserver/* /opt/gravwell/resources/datastore/
```

データストアが別のマシンにある場合は、scpまたは他のファイル転送方法を使用してそれらのファイルをWebサーバサーバからコピーします。

## 負荷分散

分散Webサーバーではユーザーの負荷分散が可能ですが、実際の負荷分散/リバースプロキシを実行するには外部ツールを使用する必要があります。ネットワークがすでにNginx +、Apache、または他のロードバランシングツールを使用している場合は、Webサーバ間でロードバランシングを行うように設定するだけで、持続セッションまたは「スティッキー」セッションが有効になります。

一方、ロードバランシング用にリバースプロキシを設定していない場合は、Gravwell Webサーバ用のスタンドアロンロードバランサとしてTræfikを設定するのが簡単です。

Traefikをそれ自身のマシンに置くか、少なくともWebサーバと同じサーバには置かないことをお勧めします。

まず、[Traefikのリリースページ](https://github.com/containous/traefik/releases)から最新のTraefikを入手するか、自分でコンパイルしてください。

SSL用の証明書も必要です。有効な証明書をインポートするか、次のコマンドで自己署名証明書を生成します。

```
openssl req -newkey rsa:4096 -nodes -sha512 -x509 -days 3650 -nodes -out traefik.crt -keyout traefik.key
```

次に、次の設定ファイルをtraefik.tomlとして保存します。

```
defaultEntryPoints = ["http", "https"]

InsecureSkipVerify = true

[file]

[entryPoints]
        [entryPoints.http]
        address = ":80"
        [entryPoints.https]
        address = ":443"
                [entryPoints.https.tls]
                        [[entryPoints.https.tls.certificates]]
                        certFile = "traefik.crt"
                        keyFile = "traefik.key"

[webservers]
        [webservers.webserver1]
                backend = "backend1"
        [webservers.webserver1.headers]
                SSLRedirect = true
                SSLTemporaryRedirect = true

[backends]
        [backends.backend1]
                [backends.backend1.loadbalancer.stickiness]
                [backends.backend1.servers.server1]
                        url="https://10.0.0.1"
                [backends.backend1.servers.server2]
                        url="https://10.0.0.2"
```

注：Traefikには、Gravwell Webサーバーに対応していない「Webサーバー」と「バックエンド」という独自の概念があります。 この設定ファイルでは、Traefikの「バックエンド」がGravwell Webサーバーサーバーを指しています。

必要に応じて設定ファイルを修正します。 特に[backends]セクションに注意してください。ここでは、10.0.0.1と10.0.0.2の2つのサーバーを定義しています。 これらはあなたのウェブサーバのIPアドレスであるべきです。 Gravwell Webサーバーが3つ以上ある場合は、サーバーセクションをさらに追加してください。

Webサーバーサーバーが有効なSSL証明書を使用している場合は、InsecureSkipVerify = trueという行を削除できます。

設定ファイルを変更したら、次のコマンドを実行してリバースプロキシを起動します。

```
./traefik -c traefik.toml -file traefik.toml
```

その後、Webブラウザを使用してTraefikサーバのホスト名またはIPにアクセスできます。 TraefikはGravwell Webサーバーの1つを選択してトラフィックをそこに誘導し、そのセッションのすべてのトラフィックが確実に同じサーバーに送信されるようにブラウザーにCookieを設定します。
