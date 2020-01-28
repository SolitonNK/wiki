# 環境変数

インデクサー、Webサーバー、およびインジェスターコンポーネントは、構成ファイルではなく環境変数を介していくつかのパラメーターを構成することをサポートします。 これは、より大規模な展開向けに、より一般的な設定ファイルを作成するのに役立ちます。 複数のディレクティブを含めることができる構成変数は、コンマ区切りリストを使用して環境変数で構成されます。 例えば、フェデレーターを起動するときに取り込みシークレットを指定するには、以下のようにします。

```
GRAVWELL_INGEST_SECRET=MyIngestSecret /opt/gravwell/bin/gravwell_federator
```
"_FILE"が環境変数名の末尾に追加された場合、Gravwellは変数にファイルへのパスが含まれていると想定し、ファイルには希望のデータが含まれています。 これは[Dockerの「秘密」機能](https://docs.docker.com/engine/swarm/secrets/)と組み合わせると特に便利です。

```
GRAVWELL_INGEST_AUTH_FILE=/run/secrets/ingest_secret /opt/gravwell/bin/gravwell_indexer
```
注：環境変数値は、対応するフィールドが適切な構成ファイル（gravwell.confまたはingesterの構成ファイル）で明示的に設定されていない場合にのみ使用されます。

## インデクサーとWebサーバー
以下の表は、インデクサおよびWebサーバ用の環境変数を介してどの `gravwell.conf`パラメータを設定できるかを示しています。 これらの変数は、パラメータが `gravwell.conf`で設定されていない場合にのみ使用されることに注意してください。

| gravwell.conf変数 | 環境変数 | 例 |
|:------|:----|:---|----:|
| Ingest-Auth | GRAVWELL_INGEST_AUTH | GRAVWELL_INGEST_AUTH=CE58DD3F22422C2E348FCE56FABA131A |
| Control-Auth | GRAVWELL_CONTROL_AUTH | GRAVWELL_CONTROL_AUTH=C2018569D613932A6BBD62A03A101E84 |
| Indexer-UUID | GRAVWELL_INDEXER_UUID | GRAVWELL_INDEXER_UUID=a6bb4386-3433-11e8-bc0b-b7a5a01a3120 |
| Webserver-UUID | GRAVWELL_WEBSERVER_UUID | GRAVWELL_WEBSERVER_UUID=b3191f54-3433-11e8-a0c2-afbff4695836 |
| Remote-Indexers | GRAVWELL_REMOTE_INDEXERS | GRAVWELL_REMOTE_INDEXERS=172.20.0.1:9404,172.20.0.2:9404|
| Replication-Peers | GRAVWELL_REPLICATION_PEERS | GRAVWELL_REPLICATION_PEERS=172.20.0.1:9406,172.20.0.2:9406 |
| Datastore | GRAVWELL_DATASTORE | GRAVWELL_DATASTORE=172.20.0.10:9405 |

## インジェスター

インジェスターは、構成ファイルで明示的に設定するのではなく、いくつかのパラメーターを環境変数として受け入れることもできます。

| 設定ファイル変数 | 環境変数 | 例 |
|:------|:----|:---|
| Ingest-Secret | GRAVWELL_INGEST_SECRET | GRAVWELL_INGEST_SECRET=CE58DD3F22422C2E348FCE56FABA131A |
| Log-Level | GRAVWELL_LOG_LEVEL | GRAVWELL_LOG_LEVEL=DEBUG |
| Cleartext-Backend-target | GRAVWELL_CLEARTEXT_TARGETS | GRAVWELL_CLEARTEXT_TARGETS=172.20.0.1:4023,172.20.0.2:4023 |
| Encrypted-Backend-target | GRAVWELL_ENCRYPTED_TARGETS | GRAVWELL_ENCRYPTED_TARGETS=172.20.0.1:4024,172.20.0.2:4024 |
| Pipe-Backend-target | GRAVWELL_PIPE_TARGETS | GRAVWELL_PIPE_TARGETS=/opt/gravwell/comms/pipe |


### フェデレーター固有の変数
フェデレーターは、それぞれに関連付けられたさまざまな取り込み秘密を持つ多数のリスナーを実行する可能性があるため、実行時にそれらのリスナー秘密を構成するための特別な環境変数のセットを認識します。

各リスナーは名前を持ちます。 以下の例では、リスナーは "base"という名前です。

```
[IngestListener "base"]
	Cleartext-Bind = 0.0.0.0:4023
	Tags=syslog
```

実行時にそのリスナーの取り込みシークレットを指定するために、変数`FEDERATOR_base_INGEST_SECRET`を使用します。

```
FEDERATOR_base_INGEST_SECRET=SuperSecret /opt/gravwell/bin/gravwell_federator
```
あるいは、他の環境変数のようにファイルを指定することもできます。
```
FEDERATOR_base_INGEST_SECRET_FILE=/run/secrets/federator_base_secret /opt/gravwell/bin/gravwell_federator
```

### データストア固有の変数

[データストア](#!distributed/frontend.md)は、実行時に環境変数によって設定できます。

| gravwell.conf変数 | 環境変数 | 例 |
|------------------------|----------------------|---------|
| Datastore-Listen-Address | GRAVWELL_DATASTORE_LISTEN_ADDRESS | GRAVWELL_DATASTORE_LISTEN_ADDRESS=192.168.1.100 |
| Datastore-Port | GRAVWELL_DATASTORE_LISTEN_PORT | GRAVWELL_DATASTORE_LISTEN_PORT=9995 |
