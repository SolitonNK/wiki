# 一般的なトラブルと警告

Gravwellの設定と使用中に発生する可能性のある問題がいくつかあります。このドキュメントでは、最も一般的な問題のいくつかを取り上げようとしています。

## クロックソースについての警告

KVMなどの仮想化環境でGravwellを実行していると、以下のような通知が表示されることがあります。:

```
Detected potentially slow clock source acpi_pm. Consider changing webservers and indexers to one of the following: tsc, kvm-clock
```
（遅い可能性のあるクロックソース acpi_pm を検出しました。ウェブサーバとインデクサを以下のいずれかに変更することを検討してください: tsc, kvm-clock）

Gravwellは時刻に深く根ざしたシステムなので、現在時刻を頻繁にチェックする必要があります。いくつかのクロックソース（基準用時計）は、他のものよりも著しく遅いため、Gravwell クエリで顕著な速度低下を引き起こす可能性があります。

クロックリソースを変更するには, [この指導書](https://aws.amazon.com/premiumsupport/knowledge-center/manage-ec2-linux-clock-source/)を参照してください。

注意: クロックソースを変更できなくても、この通知はデフォルトの'admin'ユーザーにのみ表示され、他のユーザーには表示されないので、実情に応じて無視することができます。

## Gravwell のインターフェースに到達できない

Gravwellをインストールした後、WebブラウザがWebサーバに到達できなかったり、Webサーバがどのインデクサーにも接続していなかったりすることが分かることがあるかもしれません。これはファイアウォールのポートが閉じていることが原因であることが多いです。Gravwell を適切な運用するためには、いくつかの TCP ポートを開いて通すようにしておかなければれなりません。どのポートを開いておくべきかという詳細は、[ネットワークの考慮事項](networking.md)のページを参照してください。


## HTTPS とセキュアリスナーの設定

デフォルトでは、Gravwell には TLS 証明書は含まれていませんし、生成もされていません。インターネットやその他の信頼されていないネットワーク上で Gravwell を使用する場合は、火急的に証明書をインストールすることを強くお勧めします。手順については、[証明書](certificates.md)のページを参照してください。

注：Gravwellは、TLS 1.2以降と互換性のある証明書を必要とします。

## Gravwellプロセスが起動しない

Gravwellコンポーネント(ウェブサーバー、インデクサー、サーチエージェント、インジェスターなど)が起動を拒否する場合について、いくつかありがちな原因があります。

### 設定が不適正

不適正な設定ファイルは、大抵その設定に関連するコンポーネントの不具合動作につながります。原因について詳しい情報が得られる場所がいくつかあります。

* `/dev/shm/` には通常、問題のプロセスの標準エラー出力が置かれています。例えば、ウェブサーバーコンポーネントは `/dev/shm/gravwell_webserver` に出力します。
* `/opt/gravwell/log` には、いくつかのコンポーネントのログファイルが置かれてます。
* 設定の誤り内容によっては、インジェスターが `gravwell` タグでその誤りのエラーや警告を記録していることがあります。

### 管理権限の問題

Gravwell コンポーネントは、ユーザー "gravwell "として実行されるものとして設計されています。ルートユーザーが手動でGravwellコンポーネントを実行すると、重要なファイルを作成したり修正したりして、それをルートユーザーに属するものとしてマークすることがあります。後で "gravwell "アカウントで実行すると、プロセスはファイルにアクセスできなくなります。これらのファイルを`chown` を使って gravwell ユーザに再割り当てすることができるとはいうものの、`/opt/gravwell/bin` の中は何も変更しないように注意してください。というのも、その変更内容は SELinux フラグの設定と衝突する可能性があるからです!

警告：Gravwell のサポートから明示的に指示がない限り、`/opt/gravwell/bin`内のファイルのパーミッションや所有権を変更しないでください。

### SELinux の問題

GravwellはSELinux互換性のために`/opt/gravwell`内のファイルを適切なフラグに設定するようになっていますが、`/opt/gravwell/bin`内のファイルに`chown`や`chmod`コマンドを不注意に使用すると、これらのフラグがクリアされてしまう可能性があります。詳細は [SELinuxの強化](hardening.md)のページを参照してください。

## GravwellがメモリやCPUを過剰に消費する問題

Gravwellはしばしば膨大な量のデータを処理しなければならないので、Gravwellが消費するメモリやCPUの時間は制限設定はなされていません。しかし、Gravwell を他の重要なソフトウェアと同じシステム上で実行しなければならない場合などには、リソースへのアクセスを制限したいことあるかもしれません。その場合は、[システム強化](hardening.md)のページの「Systemd Unit Files」の項を参照してください。

## Gravwellと仮想記憶領域

Gravwellのインデクサは、保存されたデータにアクセスできるようにメモリマップされた(mmap)ファイルを使用します。マップされたファイルは、インデクサーのメモリマップがファイルされる毎に増え、Linux カーネルによって許容されるファイル最大数まで増加します。システムによっては、またインデクサのデータ量や粒度によっては、メモリマップされたファイルの許容数をクラッシュ回避のために増やす必要があるかもしれません。利用可能なメモリマップされたファイル数を使い果たすと、通常 `malloc` の失敗となってクラッシュします。ほとんどの最新の Linux ディストリビューションでは、デフォルトのメモリマップされたファイルの許容数は 65536 です。


メモリマップファイルの許容数を増やすには、`sysctl`コマンドを使います:

```
sysctl -w vm.max_map_count=262144
```

メモリマップされるファイルの数の変更を永続的に保つには、`sysctl` パラメータを `sysctl.conf` に追加します:

```
echo vm.max_map_count=262144 >> /etc/sysctl.conf
```


