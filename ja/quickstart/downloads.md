# ダウンロード

注意 : debianリポジトリは、これらのスタンドアロンインストーラよりも保守が容易で、推奨されるインストール方法です。[クイックスタートの手順](#!quickstart/quickstart.md) を参照してください。

## Gravwell Core

Gravwell Coreインストーラーには、インデクサーとウェブサーバーフロントエンドが含まれています。ライセンスが必要です。Community Editionのフリーライセンスを取得するか、商用オプションについては info@gravwell.io に連絡してください。

[Gravwell Core インストーラーのダウンロード](https://update.gravwell.io/files/gravwell_4.0.2-3.sh) (SHA256: 55dd5b5941afe046a70c5b9e46133699ad409f14703e88d56594ad023beafa9f)

## インジェスター

インジェスターのコアスイートは、インストール可能なパッケージとしてダウンロードできます。 Linux マシン上で動作するように設計されたインジェスターは、通常、ホストのパッケージ管理システムに依存しない (NetworkCapture インゲスターを除いて) セルフ・インクルードで、静的にリンクされた実行可能ファイルです。 Windows ベースのインジェスターは、実行可能な MSI パッケージとして配布されます。 多くのインジェスターのソースコードは [Github](https://github.com/gravwell/ingesters) にあります。

### 現在のインジェスターのリリース
| インジェスター | 説明 | SHA256 | 詳細情報 |
|:--------:|-------------|:------:|----------:|
| [Simple Relay](#!ingesters/ingesters.md#Simple_Relay) | ネットワーク経由で送信された syslog や回線ブロークエンドのデータを受け付けることができるインジェスター。 |dc7c77fc3b3efbaf4c39260d70d34e8885c1fafb004a58db7235d06c4e9d1e50| [ダウンロード](https://update.gravwell.io/files/gravwell_simple_relay_installer_4.0.2.sh)|
| [File Follower](#!ingesters/ingesters.md#File_Follower) | ファイル内の改行されたログエントリを探すために設計された標準的なファイル追跡インジェスターです。 ファイルにしかログを記録できないシステムからログをインジェストするのに便利です。 |f451fd11bbc6a950d9df13e2f17ea4d8dae9db6348526395701c20cbe634cb50| [ダウンロード](https://update.gravwell.io/files/gravwell_file_follow_installer_4.0.2.sh) |
| [HTTP Ingester](#!ingesters/ingesters.md#HTTP_POST) | HTTP インジェスターは、HTTP リクエストをイベントとして受け取るシンプルなウェブサーバをホストすることができます。 SOHO や IoT デバイスはしばしば webhook 機能をサポートしていますが、これは HTTP インジェスターがサポートするのに完全に適しています。 |05fd887865d59c98924326d90ea7d04e9a81835b27174c4a96727e7a51d483ad| [ダウンロード](https://update.gravwell.io/files/gravwell_http_ingester_installer_4.0.2.sh) |
| [Netflow Capture](#!ingesters/ingesters.md#Netflow_Ingester) | Netflow Capture インジェスターは、Netflow v5、v9、ipfix コレクタとして動作し、Netflow レコードを Gravwell エントリとしてインジェストします。 |3aacb7deaf6c955b4988d80cbdf2a04b20d2610899daf4eb658b1a9bddbc60d6| [ダウンロード](http://update.gravwell.io/files/gravwell_netflow_capture_installer_4.0.2.sh) |
| [Network Capture](#!ingesters/ingesters.md#Network_Ingester) | Network Captureインジェスターは、複数のネットワークタップにバインドして、未加工のネットワークトラフィックをGravwellに送ることができるパッシブなネットワークスニッフィングインジェスターです。 |11069d989e1d9023f9396575279395410de307531b204755afce1a309c0b23f2| [ダウンロード](https://update.gravwell.io/files/gravwell_network_capture_installer_4.0.2.sh) |
| [Collectd Collector](#!ingesters/ingesters.md#collectd) | Collectdインジェスターは、スタンドアロンのcollectd collectorとして機能します。  |4af078b2bab39caf28c460bb439b3bae9e49e09564787765141cd7d82da34f15| [ダウンロード](https://update.gravwell.io/files/gravwell_collectd_installer_4.0.2.sh) |
| [Ingest Federator](#!ingesters/ingesters.md#Federator_Ingester) | Federatorインジェスターは、複数のダウンストリーム・インジェスターを集約し、エントリをアップストリームのインジェクショ ン・ポイントにリレーするように設計されています。 Federatorは、信頼の境界を越え、エントリフローを集約し、信頼されない可能性のあるダウンストリームのエントリジェネレータからGravwellインデクサーを絶縁するのに便利です。 |4c4c231f62ba9585f11c1710703143d0c7e2f987c81bfd47e05a96047ccc0b36| [ダウンロード](https://update.gravwell.io/files/gravwell_federator_installer_4.0.2.sh) |
| [Windows Events](#!ingesters/ingesters.md#Windows_Event_Service) | Winevent インジェスターは、Windows イベントサブシステムを使用して、Windows イベントを取得して Gravwell に送信します。 Winevent インジェスターは、ログコレクタとして動作する単一の Windows マシン上に配置することも、複数のエンドポイントに配置することもできます。 |e92965d04e76b3496fcfde4a90d52a9f8b80227a730f35ad2353db9c604cbf70| [ダウンロード](https://update.gravwell.io/files/gravwell_win_events_4.0.0.msi) |
| [Windows File Follower](#!ingesters/ingesters.md#File_Follower) | Windows用のファイルフォロワーは、ファイルフォロワーインジェスターと同じですが、Windows用です。 |570ff54992c8adac171bc4ca0dd724571d2e91d4af0c1a8b8e1f627f7c19ef2f| [ダウンロード](https://update.gravwell.io/files/gravwell_file_follow_4.0.2.msi) |
| [Apache Kafka](#!ingesters/ingesters.md#Kafka) | Apache Kafka インジェスターは、1つまたは複数の Kafka クラスタにアタッチしてトピックを読むことができます。大規模なデプロイを簡素化することができます。 |182bf634afc248ca0870c367d25b9a90d58924d8c07bcd25bcd2ca52a4c85ae0| [ダウンロード](https://update.gravwell.io/files/gravwell_kafka_installer_4.0.2.sh)|
| [Amazon Kinesis](#!ingesters/ingesters.md#Kinesis_Ingester) | Amazon Web Services Kinesis インジェスターは、Kinesis ストリームにアタッチして、クラウド展開のロギングを劇的に簡素化することができます。 |4a3822a9386030fcae1a58df0ec39d2071cb2348e2f3cdc5a8ae30babe5a6727| [ダウンロード](https://update.gravwell.io/files/gravwell_kinesis_ingest_installer_4.0.2.sh)|
| [Google PubSub](#!ingesters/ingesters.md#GCP_PubSub) | Google Cloud Platform PubSubインジェスターは、GCP PubSubシステム上の排気をサブスクライブすることができ、GCPとの統合を容易にします。 |257b300a04cd5fe565052ac417eb317cda68e2955040f320a6162f70c5c26bde| [ダウンロード](https://update.gravwell.io/files/gravwell_pubsub_ingest_installer_4.0.2.sh)|
| [Office 365 Logs](#!ingesters/ingesters.md#Office_365_Log_Ingester) | Office 365ログインジェスターは、Microsoft Office 365からログイベントを取得することができます。 |9db7a2cee8d2e5b99e7b0df7819e47f144ef14556551ac29c4b6f4336d0d8969| [ダウンロード](https://update.gravwell.io/files/gravwell_o365_installer_4.0.2.sh)|
| [Microsoft Graph API](#!ingesters/ingesters.md#Microsoft_Graph_API_Ingester) | MS Graph APIインジェスターは、Microsoft Graph APIからセキュリティ情報を取得することができます。 |e9ba56cb59c180e8908809b14909484c15c019fbcc6b1876bad7c1b6030b408a| [ダウンロード](https://update.gravwell.io/files/gravwell_msgraph_installer_4.0.2.sh)|
[//]: <> (| [](#!ingesters/ingesters.md#) | | | [ダウンロード](https://update.gravwell.io/files/) |)

## その他のダウンロード

Gravwellコンポーネントの中には、検索エージェントやデータストアなど、オプションの追加インストーラーとして配布されているものもあります。

| コンポーネント | 説明 | SHA256 | 詳細情報 |
|:---------:|-------------|:------:|----------:|
| [Datastore](#!distributed/frontend.md) | データストアは複数のGravwellウェブサーバーを同期させ、ロードバランシングを可能にします。 |5f3d7646e316a2f672bbd22d48c9bdfacaf288805e8cf3b88484945ed3f03fe5| [ダウンロード](https://update.gravwell.io/files/gravwell_datastore_installer_4.0.2.sh) |
| [Offline Replicator](#!configuration/replication.md) | オフラインレプリケーションサーバーはスタンドアロンのレプリケーションピアとして動作し、クエリには参加せず、シングルインデクサーのGravwellインストールとペアにするのが最適です。 |aa7936c718de5f49f9df1cd0d21093103b03eae6a6e4e07c59b3b2e36559308d| [ダウンロード](https://update.gravwell.io/files/gravwell_offline_replication_installer_4.0.2.sh) |
| Load Balancer | ロードバランサーは、複数のGravwellウェブサーバーをフロントするために、 Gravwell固有のHTTPロードバランシングソリューションを提供する。Gravwell ウェブサーバーのリストを取得するためにデータストアに接続します。 |0b6a2103dff851c520d4b6f4a90dd721b5fba91a2553906929b9b40836226128| [ダウンロード](https://update.gravwell.io/files/gravwell_loadbalancer_installer_4.0.2.sh) |
