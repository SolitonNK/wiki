# ダウンロード

重要：debianリポジトリは、これらのスタンドアロンインストーラーよりも簡単に管理でき、推奨されるインストール方法です。 [クイックスタート手順](#!quickstart/quickstart.md)を参照してください。

## Gravwell Core

Gravwellコアインストーラーには、インデクサーとWebサーバーフロントエンドが含まれています。 ライセンスが必要です。 Community Editionの無料ライセンスを取得するか、info @ gravwell.ioに連絡して商用オプションを入手してください。

[Gravwell Coreインストーラーをダウンロードする](https://update.gravwell.io/files/gravwell_3.2.3-5.sh) (MD5: 10244502cd90ee27deb4e3d9a94f10d2)

## インジェスター

インジェスターのコアスイートは、インストール可能なパッケージとしてダウンロードできます。 Linuxマシンで動作するように設計されたインジェスターは通常、自己完結型の静的にリンクされた実行可能ファイルであり、ホストパッケージ管理システムにとらわれません（NetworkCapture ingesterを除く）。 Windowsベースのインジェスターは、実行可能なMSIパッケージとして配布されます。 多くのインジェスターのソースコードは[Github](https://github.com/gravwell/ingesters)にあります。

### バージョン3.2.3 Ingesterリリース

| Ingester | Description | MD5 | More Info |
|:--------:|-------------|:---:|----------:|
| [Simple Relay](#!ingesters/ingesters.md#Simple_Relay) | An ingester capable of accepting syslog or line brokend data sent over the network. |9a96b7a4653c7d41a9cbed2025e3fd91| [Download](https://update.gravwell.io/files/gravwell_simple_relay_installer_3.2.3-5.sh)|
| [File Follower](#!ingesters/ingesters.md#File_Follower) | The standard file following ingester designed to look for line broken log entries in files.  Useful for ingesting logs from systems that can only log to files. |e4278acd7621409ace7707b01f4585fc| [Download](https://update.gravwell.io/files/gravwell_file_follow_installer_3.2.3-5.sh) |
| [HTTP Ingester](#!ingesters/ingesters.md#HTTP_POST) | The HTTP ingester allows for hosting a simple webserver that takes HTTP requests in as events.  SOHO and IOT devices often support webhook functionality which the HTTP ingester is perfectly suited to support. |cc929f31df4f164b7e777bd102803453| [Download](https://update.gravwell.io/files/gravwell_http_ingester_installer_3.2.3-5.sh) |
| [Netflow Capture](#!ingesters/ingesters.md#Netflow_Ingester) | The Netflow Capture ingester acts as a Netflow v5 collector, ingesting Netflow records as Gravwell entries. |fbed698cccaf4121492821e4e2305886| [Download](http://update.gravwell.io/files/gravwell_netflow_capture_installer_3.2.3-5.sh) |
| [Network Capture](#!ingesters/ingesters.md#Network_Ingester) | The Network Capture ingester is a passive network sniffing ingester which can bind to multiple network taps and send raw network traffic to Gravwell. |ec02fea008f07ba2ade47f48377e4805| [Download](https://update.gravwell.io/files/gravwell_network_capture_installer_3.2.3-5.sh) |
| [Collectd Collector](#!ingesters/ingesters.md#collectd) | The collectd ingester acts as a standalone collectd collector.  |e1d7f485063e485250d2d3246b94cb5c| [Download](https://update.gravwell.io/files/gravwell_collectd_installer_3.2.3-5.sh) |
| [Ingest Federator](#!ingesters/ingesters.md#Federator_Ingester) | The Federator ingester is designed to aggregate multiple downstream ingesters and relay entries to upstream ingestion points.  The Federator is useful for crossing trust boundaries, aggregating entry flows, and insulating Gravwell indexers from potentially untrusted downstream entry generators. |ada4b595dab097595d1d5fc0cf14fe2d| [Download](https://update.gravwell.io/files/gravwell_federator_installer_3.2.3-5.sh) |
| [Windows Events](#!ingesters/ingesters.md#Windows_Event_Service) | The Winevent ingester uses the Windows events subsystem to acquire windows events and ship them to gravwell.  The Winevent ingester can be placed on a single Windows machine acting as a log collector, or on multiple endpoints. |1b21e815bb1c35243e7b2c5a036caead| [Download](https://update.gravwell.io/files/gravwell_win_events_3.2.2.msi) |
| [Windows File Follower](#!ingesters/ingesters.md#File_Follower) | The Windows file follower is identical to the File Follower ingester, but for Windows. |aa8bd348e847fd593c41ca4e0ff679a6| [Download](https://update.gravwell.io/files/gravwell_file_follow_3.2.2.msi) |
| [Apache Kafka](#!ingesters/ingesters.md#Kafka) | The Apache Kafka ingester can attach to one or many Kafka clusters and read topics. It can simplify massive deployments. |e0f13328d2bc033e8ecdc9994029ced7| [Download](https://update.gravwell.io/files/gravwell_kafka_installer_3.2.3-5.sh)|
| [Amazon Kinesis](#!ingesters/ingesters.md#Kinesis_Ingester) | The Amazon Web Services Kinesis ingester can attach to the Kinesis stream and dramatically simplify logging a cloud deployment |508342a1caf9b111e72419ea6bdecfb4| [Download](https://update.gravwell.io/files/gravwell_kinesis_ingest_installer_3.2.3-5.sh)|
| [Google PubSub](#!ingesters/ingesters.md#GCP_PubSub) | The Google Cloud Platform PubSub Ingester can subscribe to exhausts on the GCP PubSub system, easing integration with GCP. |ad1d6e0f8bba9b53ab1601b7376a1870| [Download](https://update.gravwell.io/files/gravwell_pubsub_ingest_installer_3.2.3-5.sh)|

[//]: <> (| [](#!ingesters/ingesters.md#) | | | [Download](https://update.gravwell.io/files/) |)
[//]: <> (| [](#!ingesters/ingesters.md#) | | | [Download](https://update.gravwell.io/files/) |)

## その他のダウンロード

一部のGravwellコンポーネントは、検索エージェントやデータストアなどのオプションの追加インストーラーとして配布されます。

| コンポーネント | 説明 | MD5 | 詳細情報 |
|:---------:|-------------|:---:|----------:|
| [データストア](#!distributed/frontend.md) | データストアは複数のGravwell Webサーバーの同期を維持し、負荷分散を可能にします| 4cdbb5bc4c138158efeaa80ecef6dd58 | [ダウンロード](https://update.gravwell.io/files/gravwell_datastore_installer_3.2.3-5.sh)|
