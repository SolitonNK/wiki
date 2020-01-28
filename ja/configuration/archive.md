# クラウドアーカイブ

Gravwellは、Cloud Archiveと呼ばれるエイジアウトメカニズムをサポートしています。クラウドアーカイブは、削除する前にデータをリモートでアーカイブできるリモートサービスです。 Gravwell Cloud Archiveは、積極的に検索する必要はないが、保持する必要があるデータの長期アーカイブストレージの優れた方法です。 Cloud Archiveサービスは、さまざまなストレージプラットフォームでホストでき、リモートの低コストのストレージプラットフォームを提供するように設計されています。 Cloud Archiveの構成は、ウェルごとに有効にできます。つまり、どのデータセットが長期アーカイブを保証するかを決定できます。

アーカイブシステムは、通常のエージアウト中に削除される前に、データがアーカイブサーバーに正常にアップロードされるようにします。

重要：インデクサーは、データをアーカイブサーバーに正常にアップロードするまでデータを削除しません。接続の問題、構成の誤り、またはネットワークスループットの低下によりインデクサーがアップロードできない場合、データは削除されません。データを削除できないため、インデクサーのストレージが不足し、新しいデータの取り込みが停止する場合があります。クラウドアーカイブのアップロードが完了しなかった場合、ユーザーインターフェイスに失敗の通知が表示されます。

重要：Cloud Archiveシステムは、送信中にデータを圧縮します。これにより、アップロード時にCPUリソースが必要になります。データをリモートシステムにプッシュする場合も、利用可能な帯域幅とCPUに応じて時間がかかります。 Cloud Archiveで消費される追加の時間を考慮して、エージアウトパラメーターを定義するときは、少し余裕を持たせてください。

## インデクサーの構成

すべてのインデクサーには、リモートアーカイブサーバーと認証トークンを指定するグローバルクラウドアーカイブ構成ブロックがあります。 構成ブロックは、グローバルセクションの「[cloud-archive]」ヘッダーを使用して指定されます。 ウェルでCloud Archiveを有効にするには、ウェル内に「Archive-Deleted-Shards = true」ディレクティブを追加します。

3つのウェルを使用した構成の例を次に示します。

```
[global]
Web-Port=443
Control-Port=9404
Ingest-Port=4023

[Cloud-Archive]
	Archive-Server=test.archive.gravwell.io
	Archive-Shared-Secret="password"

[Default-Well]
	Location=/opt/gravwell/storage/default/
	Cold-Location=/opt/gravwell/cold_storage/default/
	Hot-Duration=1d
	Cold-Duration=30d
	Delete-Frozen-Data=true
	Archive-Deleted-Shards=true

[Storage-Well "netflow"]
	Location=/opt/gravwell/storage/netflow/
	Hot-Duration=7d
	Delete-Cold-Data=true
	Archive-Deleted-Shards=true
	Tags=netflow

[Storage-Well "raw"]
	Location=/opt/gravwell/storage/raw/
	Hot-Duration=7d
	Delete-Cold-Data=true
	Tags=pcap
```

上記の例には、3つのウェル（デフォルト、netflow、およびraw）が構成されています。 デフォルトのウェルでは、ホットストレージ層とコールドストレージ層の両方が使用されます。つまり、データは通常、コールドストレージ層からロールアウトされるときにアーカイブされます。 netflowウェルにはホットストレージ層のみが含まれており、そのデータは通常7日後に削除されるときにアップロードされます。 rawウェルではCloud Archiveが有効になっていないため（Archive-Deleted-Shards = false）、データはアップロードされません。

## ホスティングクラウドアーカイブ

Cloud Archiveサービスは、自己ホスト型であり、他の大規模なインフラストラクチャに統合される可能性があるように設計されたモジュールサービスです。 独自のCloud Archiveサービスをホストすることに興味がある場合、またはデータをリモートでアーカイブする場合は、sales @ gravwell.ioにお問い合わせください。
