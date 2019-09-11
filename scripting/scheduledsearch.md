# 検索エージェントを使用した検索のスケジューリング

たとえば、毎朝検索を実行して前夜からの悪意のある動作を検出するなど、検索を自動的に実行することはしばしば有利です。 Gravwellの検索エージェントを使用して、カスタマイズされたスケジュールで検索を実行できます。

スケジューリング機能により、ユーザーは通常の検索と[検索スクリプト](scriptingsearch.md)の両方をスケジュールできます。

## 検索エージェントのセットアップ

Gravwell Search AgentはメインのGravwellインストールパッケージに含まれるようになり、デフォルトでインストールされます。 `--no-webserver`フラグでwebserverコンポーネントを無効にするか、` --no-searchagent`フラグを設定すると、検索エージェントのインストールが無効になります。 検索エージェントはGravwell Debianパッケージによって自動的にインストールされます。

次のコマンドを使用して、検索エージェントが実行されていることを確認します。

```
$ ps aux | grep gravwell_searchagent
```

## 検索エージェントを無効にする

検索エージェントはデフォルトでインストールされますが、必要に応じて次を実行して無効にすることができます。

```
systemctl stop gravwell_searchagent.service
systemctl disable gravwell_searchagent.service
```

## 検索エージェントスクリプトでネットワーク機能を無効にする

デフォルトでは、検索エージェントによって実行されるスケジュールされたスクリプトは、httpライブラリ、sftp、sshなどのネットワークユーティリティを使用できます。 `/ opt / gravwell / etc / searchagent.conf`でオプション` Disable-Network-Script-Functions = true 'を設定すると、これが無効になります。