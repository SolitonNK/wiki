# 圧縮

ストレージはデフォルトで圧縮されており、ストレージからCPUへの負荷の一部を移行するのに役立ちます。 Gravwellは、規模の広いパラダイムで最新のハードウェア向けに構築された高度に非同期なシステムです。 現代のシステムは通常、CPUで過剰にプロビジョニングされ、大容量ストレージが遅れています。 データを圧縮することにより、Grabwellはストレージリンクへのストレスを軽減しながら、過剰なCPUサイクルを使用して非同期的にデータを圧縮および圧縮解除できます。 その結果、低速の大容量記憶装置で圧縮を使用すると、検索が高速になります。 圧縮は、ホットデータとコールドデータに対して異なる圧縮設定を使用して、ウェルごとに個別に構成できます。

注目に値する例外は、データをあまり圧縮しない場合（まったく圧縮しない場合）です。 この状況では、データを圧縮しようとするとCPU時間を消費しますが、ストレージスペースや速度は実際には改善されません。 生のネットワークトラフィックは、暗号化と高いエントロピーが効果的な圧縮を妨げる良い例です。 ウェルの圧縮を無効にするには、"Disable-Compression=true"ディレクティブを追加します。

## 圧縮設定

縮と透過圧縮の2種類の圧縮をサポートしています。 デフォルトの圧縮では、[snappy](https://en.wikipedia.org/wiki/Snappy_%28compression%29)圧縮システムを使用して、ユーザースペースで圧縮と圧縮解除を実行します。 デフォルトの圧縮システムは、すべてのファイルシステムと互換性があります。 透過的な圧縮システムは、基礎となるファイルシステムを使用して透過的なブロックレベルの圧縮を提供します。

透過的な圧縮により、圧縮されていないページキャッシュを維持しながら、ホストカーネルに圧縮/解凍作業をオフロードできます。 透過的圧縮は、非常に高速で効率的な圧縮/解凍を可能にしますが、基礎となるファイルシステムが透過的圧縮をサポートする必要があります。 現在、[BTRFS](https://btrfs.wiki.kernel.org/index.php/Main_Page)および[ZFS](https://wiki.archlinux.org/index.php/ZFS)ファイルシステムがサポートされています。

**圧縮を無効にする**
デフォルト値： `false`
例： `Disable-Compression = true`
ウェル全体の圧縮は無効になっています。ホットストレージとコールドストレージの両方で圧縮を使用しません

**ホット圧縮を無効にする**
デフォルト値： `false`
例： `Disable-Hot-Compression = true`
ホットストレージの場所の圧縮は無効になっています。

**コールド圧縮を無効にする**
デフォルト値： `false`
例： `Disable-Cold-Compression = true`
コールドストレージの場所の圧縮は無効になります。コールドストレージの場所が指定されていない場合、設定は効果がありません。

**Enable-Transparent-Compression**
デフォルト値： `false`
例： `Enable-Transparent-Compression = true`
Gravwellは、ストレージデータを圧縮可能としてマークし、圧縮操作を実行するためにカーネルに依存します。

**Enable-Hot-Transparent-Compression**
デフォルト値： `false`
例： `Enable-Hot-Transparent-Compression = true`
Gravwellは、ホットストレージデータを圧縮可能としてマークし、圧縮操作を実行するためにカーネルに依存します。

**Enable-Cold-Transparent-Compression**
デフォルト値： `false`
例： `Enable-Cold-Transparent-Compression = true`
Gravwellは、コールドストレージデータを圧縮可能としてマークし、圧縮操作を実行するためにカーネルに依存します。

注：透過的な圧縮が有効になっており、基礎となるファイルシステムが透過的な圧縮と互換性がないと検出された場合、データは非圧縮になり、Gravwellはユーザーに通知を送信します。

警告：ホットストレージとコールドストレージの場所が圧縮に関して互換性がない場合、Gravwellはデータをホットからコールドにエージングアウトするための追加作業を実行する必要があります。 アクセラレーションが有効になっている場合、Gravwellはエージアウトを実行するときにデータのインデックスを再作成します。 互換性のない圧縮設定は、エージアウト中に大きなオーバーヘッドを招く可能性があります。 非圧縮データは透過的に圧縮されたデータと互換性がありますが、デフォルトの圧縮は非圧縮または透過的に圧縮されたデータと互換性がありません。 Gravwellは、互換性のない圧縮でも完全に正常に機能しますが、インデクサーはエージングアウト中にさらに激しく動作します。


## 圧縮と複製

[レプリケーションシステム] [replication.md]は、通常のウェルストレージと同じルールに従います。 レプリケートされたデータは、透過的な圧縮、デフォルトの圧縮、または圧縮なしを使用するように構成できます。 ウェル内のホットストレージとコールドストレージの場所と圧縮の互換性に関する同じルールは、複製されたデータと複製ピアにも適用されます。 レプリケーションピアが互換性のない形式の圧縮インデクサーを構成している場合、障害後に復元する際の作業が大幅に増えます。 最高のパフォーマンスを得るために、ホット、コールド、およびレプリケーションストアは同じ圧縮スキームを使用することをお勧めします。

レプリケーションストレージの場所の圧縮は、 `Disable-Compression`および` Enable-Transparent-Compression`ディレクティブによって制御されます。 snappy compression systmeはデフォルトの圧縮スキームです。

## 圧縮の例

ウェル全体の圧縮が無効になっているストレージウェルの例：

```
[Storage-Well "network"]
	Location=/opt/gravwell/storage/network
	Cold-Location=/mnt/storage/gravwell_cold/network
	Tags=pcap
	Max-Hot-Data-GB=100
	Max-Cold-Data-GB=1000
	Delete-Frozen-Data=true
	Disable-Compression=true
```

ホットストレージの場所に対して圧縮が無効になり、コールドの場所に対して透過的な圧縮が有効になっているストレージウェルの例。 この構成は互換性があると見なされ、エージングアウト中に追加の作業を必要としません。

```
[Storage-Well "syslog"]
	Location=/opt/gravwell/storage/syslog
	Cold-Location=/mnt/storage/gravwell_cold/syslog
	Tags=syslog
	Max-Hot-Data-GB=100
	Max-Cold-Data-GB=1000
	Delete-Frozen-Data=true
	Disable-Hot-Compression=true
	Enable-Cold-Transparent-Compression=true
```

ホットストレージの場所でトランスペアレント圧縮が有効になっているストレージウェルの例と、コールドウェルのデフォルトのユーザースペース圧縮。 この構成は互換性がないと見なされ、データのエージアウト中に追加のオーバーヘッドが発生します。

```
[Storage-Well "windows"]
	Location=/opt/gravwell/storage/windows
	Cold-Location=/mnt/storage/gravwell_cold/windows
	Tags=windows
	Max-Hot-Data-GB=100
	Max-Cold-Data-GB=1000
	Delete-Frozen-Data=true
	Enable-Hot-Transparent-Compression=true
	Disable-Cold-Compression=true
```

レプリケーションストレージに透過的な圧縮を使用するレプリケーション構成の例。

```
[Replication]
	Peer=indexer1
	Peer=indexer2
	Peer=indexer3
	Peer=indexer4
	Storage-Location=/mnt/storage/replication
	Enable-Transparent-Compression=true
```
