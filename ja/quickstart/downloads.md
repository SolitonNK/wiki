# ダウンロード

注 : debianリポジトリは、これらのスタンドアロンインストーラよりも保守が容易で、推奨されるインストール方法です。[クイックスタートの手順](#!quickstart/quickstart.md) を参照してください。

## Gravwellコア

Gravwell Coreインストーラーには、インデクサーとウェブサーバーフロントエンドが含まれています。ライセンスが必要です。Community Editionのフリーライセンスを取得するか、商用オプションについては info@gravwell.io に連絡してください。

[Gravwell Core インストーラーのダウンロード](https://update.gravwell.io/archive/4.1.3/installers/gravwell_4.1.3.sh) (SHA256: 41ada3842d9ab9a09e4841f2d3a15c8d7adaa0c89775347e95f3cb9885c035d9)

## インジェスター

インジェスターのコアスイートは、インストール可能なパッケージとしてダウンロードできます。 Linux マシン上で動作するように設計されたインジェスターは、通常、ホストのパッケージ管理システムに依存しない (NetworkCapture インゲスターを除いて) セルフ・インクルードで、静的にリンクされた実行可能ファイルです。 Windows ベースのインジェスターは、実行可能な MSI パッケージとして配布されます。 多くのインジェスターのソースコードは [Github](https://github.com/gravwell/ingesters) にあります。

### 現在リリースされているインジェスター
| インジェスター | 説明 | SHA256 | 詳細情報 |
|:--------:|-------------|:------:|----------:|
| [Simple Relay](#!ingesters/ingesters.md#Simple_Relay) | ネットワーク経由で送信された syslog や回線ブロークエンドのデータを受け付けることができるインジェスター。 |a5699012807f3775b5f0ca533a6306959b9871a88bc67e532d59ee09a7ef7e6d| [Download](https://update.gravwell.io/archive/4.1.3/installers/gravwell_simple_relay_installer_4.1.3.sh)|
| [File Follower](#!ingesters/ingesters.md#File_Follower) | ファイル内の改行されたログエントリを探すために設計された標準的なファイル追跡インジェスター。ファイルにしかログを記録できないシステムからログをインジェストするのに便利です。 |a33903077842979a250f9644abd184709e1c0c1180624c2a28b4238bcd9fe9dc| [Download](https://update.gravwell.io/archive/4.1.3/installers/gravwell_file_follow_installer_4.1.3.sh) |
| [HTTP Ingester](#!ingesters/ingesters.md#HTTP_POST) | HTTPインジェスターは、HTTPリクエストをイベントとして受け取るシンプルなウェブサーバをホストすることを可能にします。SOHOや IoTデバイスはしばしば webhook機能をサポートしていますが、これは HTTPインジェスターが完全にサポートするのに適しています。 |cf9c420bedaf7f310393d55330d67dde8d4a75adcdac21338b4aa73faeff3700| [Download](https://update.gravwell.io/archive/4.1.3/installers/gravwell_http_ingester_installer_4.1.3.sh) |
| [Netflow Capture](#!ingesters/ingesters.md#Netflow_Ingester) | Netflow Captureインジェスターは、Netflow v5、v9、ipfixコレクタとして動作し、NetflowレコードをGravwellエントリとしてインジェストします。 |908fa597473b9adf6a8b19818b906dcfc7d32791851c158564499395a77917ca| [Download](http://update.gravwell.io/archive/4.1.3/installers/gravwell_netflow_capture_installer_4.1.3.sh) |
| [Network Capture](#!ingesters/ingesters.md#Network_Ingester) | Network Captureインジェスターは、複数のネットワークタップにバインドして、未加工のネットワークトラフィックをGravwellに送ることができるパッシブなネットワークスニッフィングインジェスターです。 |08a52091fa6660aca987075924648cd662a1a1593bc9df2baa0c15cfabdab593| [Download](https://update.gravwell.io/archive/4.1.3/installers/gravwell_network_capture_installer_4.1.3.sh) |
| [Collectd Collector](#!ingesters/ingesters.md#collectd) | Collectdインジェスターは、スタンドアロンのcollectd collectorとして機能します。  |20884e8c26b98830679f838969509963497c876b6797146bc9ea9883c5442062| [Download](https://update.gravwell.io/archive/4.1.3/installers/gravwell_collectd_installer_4.1.3.sh) |
| [Ingest Federator](#!ingesters/ingesters.md#Federator_Ingester) |  Federatorインジェスターは、複数のダウンストリーム・インジェスターを集約し、エントリをアップストリームのインジェクショ ン・ポイントにリレーするように設計されています。 Federatorは、信頼の境界を越え、エントリフローを集約し、信頼されない可能性のあるダウンストリームのエントリジェネレータからGravwellインデクサーを絶縁するのに便利です。 |e58e3fec759e869cc8e4ea2b0dc9d280436d214554652a8bdd9eee7a36e06be4| [Download](https://update.gravwell.io/archive/4.1.3/installers/gravwell_federator_installer_4.1.3.sh) |
| [Windows Events](#!ingesters/ingesters.md#Windows_Event_Service) | Wineventインジェスターは、Windowsイベントサブシステムを使用して、Windowsイベントを取得してGravwellに送信します。 Wineventインジェスターは、ログコレクタとして動作する単一の Windowsマシン上に配置することも、複数のエンドポイントに配置することもできます。 |26ba3363ead1f91fc5721fd088a36c4b21a9de17923aedc1fbecdde8356a6cc4| [Download](https://update.gravwell.io/archive/4.1.3/installers/gravwell_win_events_4.1.3.msi) |
| [Windows File Follower](#!ingesters/ingesters.md#File_Follower) | Windows用のファイルフォロワーは、ファイルフォロワーインジェスターと同じですが、Windows用です。 |8fc5fdc1f0a7dd7a1ec87d625d508cd4329931c53934016f757e0ac2a0af936a| [Download](https://update.gravwell.io/archive/4.1.3/installers/gravwell_file_follow_4.1.3.msi) |
| [Apache Kafka](#!ingesters/ingesters.md#Kafka) | Apache Kafkaインジェスターは、1つまたは複数のKafkaクラスタにアタッチしてトピックを読むことができます。大規模なデプロイを簡素化することができます。 |610bac8ceb7f69a017621fe5eb2956d3b1beda54dcff220f1b56b032b8f20bef| [Download](https://update.gravwell.io/archive/4.1.3/installers/gravwell_kafka_installer_4.1.3.sh)|
| [Amazon Kinesis](#!ingesters/ingesters.md#Kinesis_Ingester) | Amazon Web Services Kinesis インジェスターは、Kinesis ストリームにアタッチして、クラウド展開のロギングを劇的に簡素化することができます。 |03c964e52695dbb893b36e3395a69cf09265d684522746679ae524652d30f73e| [Download](https://update.gravwell.io/archive/4.1.3/installers/gravwell_kinesis_ingest_installer_4.1.3.sh)|
| [Google PubSub](#!ingesters/ingesters.md#GCP_PubSub) | Google Cloud Platform PubSubインジェスターは、GCP PubSubシステム上の排気をサブスクライブすることができ、GCPとの統合を容易にします。 |3038047c833844532743c4442d9a3b7d1eec931218080c4cb627f5e9d0fead5a| [Download](https://update.gravwell.io/archive/4.1.3/installers/gravwell_pubsub_ingest_installer_4.1.3.sh)|
| [Office 365 Logs](#!ingesters/ingesters.md#Office_365_Log_Ingester) | Office 365ログインジェスターは、Microsoft Office 365からログイベントを取得することができます。 |fbb9b989ebf605c040193b6a5db02df7424b9dec2002560b0a133f951a8f65d3| [Download](https://update.gravwell.io/archive/4.1.3/installers/gravwell_o365_installer_4.1.3.sh)|
| [Microsoft Graph API](#!ingesters/ingesters.md#Microsoft_Graph_API_Ingester) | MS Graph APIインジェスターは、Microsoft Graph APIからセキュリティ情報を取得することができます。 |f01819094.1.24acb19c120f2f42062953ddc96a7c26968d29460662e427712e| [Download](https://update.gravwell.io/archive/4.1.3/installers/gravwell_msgraph_installer_4.1.3.sh)|

[//]: <> (| [](#!ingesters/ingesters.md#) | | | [Download](https://update.gravwell.io/archive/4.1.3/installers/) |)

## その他のダウンロード

Gravwellコンポーネントの中には、検索エージェントやデータストアなど、オプションの追加インストーラーとして配布されているものもあります。

| コンポーネント | 説明 | SHA256 | 詳細情報 |
|:---------:|-------------|:------:|----------:|
| [Datastore](#!distributed/frontend.md) | データストアは複数のGravwellウェブサーバーを同期させ、ロードバランシングを可能にします。 |4f6666d63534e2c42c48f48b68214b1b22b70126ce7e6f6435b35e2367564075| [Download](https://update.gravwell.io/archive/4.1.3/installers/gravwell_datastore_installer_4.1.3.sh) |
| [Offline Replicator](#!configuration/replication.md) | オフラインレプリケーションサーバーはスタンドアロンのレプリケーションピアとして動作し、クエリには参加せず、シングルインデクサーのGravwellインストールとペアにするのが最適です。 |5c15d1e08e59e2cbabc128118e7ff77081ba1828021c12899a8538268d04bb6f| [Download](https://update.gravwell.io/archive/4.1.3/installers/gravwell_offline_replication_installer_4.1.3.sh) |
| Load Balancer | ロードバランサーは、複数のGravwellウェブサーバーをフロントするために、 Gravwell固有のHTTPロードバランシングソリューションを提供する。Gravwell ウェブサーバーのリストを取得するためにデータストアに接続します。 |66dc89ab3a6f1903f40b8abe191a3e67b6627107b2648452bb911fc634d3e7f4| [Download](https://update.gravwell.io/archive/4.1.3/installers/gravwell_loadbalancer_installer_4.1.3.sh) |
