# 

![](logo-name.png)

# Gravwell

このサイトでは、Gravwellのドキュメントと、Changelogsなどの他のリソースが含まれています。

Gravwellを使い始めたばかりの方は、まず[クイックスタート](quickstart/quickstart.md)を読んでから、[パイプラインを検索](search/search.md)のドキュメントを読んで詳細を知ることをお勧めします。

Gravwellは、無料の[コミュニティ版](https://www.gravwell.io/download)を発表しました。

## クイックスタートとダウンロード

  * [クイックスタート](quickstart/quickstart.md)

  * [ダウンロード](quickstart/downloads.md)

## Gravwellで検索

  * [検索概要](search/search.md)

  * [抽出モジュールの検索](search/extractionmodules.md)

  * [検索処理モジュール](search/processingmodules.md)

  * [レンダーモジュールの検索](search/rendermodules.md)

  * [全パイプラインモジュールのアルファベット順リスト](search/complete-module-list.md)

## システムアーキテクチャ

  * [Gravwellシステムアーキテクチャ](architecture/architecture.md)

    * [Gravwell が使用するネットワークポート](configuration/networking.md)


  * [リソースシステム](resources/resources.md)

## インジェスターの設定 : Gravwellへのデータ取り込み

  * [インジェスターの設定](ingesters/ingesters.md)

    * [File Follower インジェスター](ingesters/file_follow.md)

    * [Simple Relay インジェスター](ingesters/simple_relay.md)
    
    * [Windows Events インジェスター](ingesters/ingesters.md#Windows_Event_Service)

    * [Netflow/IPFIX インジェスター](ingesters/ingesters.md#Netflow_Ingester)

    * [Collectd](ingesters/ingesters.md#collectd_Ingester)

  * [インジェスター前処理](ingesters/preprocessors/preprocessors.md)

  * [サービスの統合](ingesters/integrations.md)

## 高度なGravwellのインストールと設定

  * [Gravwellのインストールと設定](configuration/configuration.md)

  * [Dockerデプロイメント](configuration/docker.md)

  * [TLS/HTTPSの設定](configuration/certificates.md)

  * [Gravwellクラスタの構築](distributed/cluster.md)

  * [分散フロントエンド](distributed/frontend.md)

    * [監視](distributed/overwatch.md)


  * [環境変数](configuration/environment-variables.md)

  * [詳細設定パラメータ](configuration/parameters.md)

  * [シングルサインオン](configuration/sso.md)

  * [Gravwellの堅牢化](configuration/hardening.md)

## クエリの高速化、自動抽出、データ管理
  
  * [自動抽出器の設定](configuration/autoextractors.md)
  
  * [クエリの高速化（インデックス化とブルームフィルタ）](configuration/accelerators.md)

  * [データ複製](configuration/replication.md)

  * [データエイジアウト](configuration/ageout.md)

  * [データ圧縮](configuration/compression.md)

  * [データアーカイブ](configuration/archive.md)

## 自動化

  * [スケジュールされた検索とスクリプト](scripting/scheduledsearch.md)

  * [スクリプトの概要](scripting/scripting.md)

    * [オートメーションスクリプトのAPIと例](scripting/scriptingsearch.md)

## ユーザーインターフェース

  * [Gravwell Web GUI](gui/gui.md)

    * [検索インターフェイス](gui/queries/queries.md)

    * [ラベルとフィルタリング](gui/labels/labels.md)

		* [キット](kits/kits.md)

  * [コマンドラインクライアント](cli/cli.md)

## API

  * [API](api/api.md)

## その他

  * [ライセンス](license/license.md)

  * [メトリクスとクラッシュレポート](metrics.md)

  * [Changelogs](changelog/list.md)

  * [Gravwell EULA](eula.md)

  * [オープンソースライセンス](open_source.md)

Documentation version 2.0
