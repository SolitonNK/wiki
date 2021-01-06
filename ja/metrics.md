# Gravwellのメトリクスとクラッシュレポート

Gravwellユーザーは、ネットワークで何が起こっているかを気にしています。私たちは、すべてのユーザーが、Gravwellに組み込まれた自動クラッシュレポートおよびメトリクスシステムを認識し、快適に利用できるようにしたいと考えています。このドキュメントでは、Gravwellサーバーに送信する内容の完全な例を示しながら、両方のシステムについて説明します。

## クラッシュレポート

Gravwellコンポーネントがクラッシュすると、自動クラッシュレポートがGravwellに送られます。これは、問題のコンポーネントからのコンソール出力で構成され、通常、ライセンスに関する簡単な情報(誰のシステムがクラッシュしたかを判断するため)とスタックトレースが含まれています。**ウェブサーバー、インデクサー、インジェスター、検索エージェントなど、すべての** Gravwellコンポーネントは、クラッシュレポートを送信するように設定されています。
 
注意: クラッシュレポートは、常にTLSで検証されたHTTPS経由でupdate.gravwell.ioに送信されます。リモート証明書を完全に検証できなかった場合、レポートは出力されません。

ここでは、Gravwellの社員のテストシステムのクラッシュレポートの例を紹介します。

```
Component:      webserver
Version:        3.3.5
Host:           X.X.X.X
Domain: c-X-X-X-X.hsd1.nm.comcast.net
Full Log Location:      /opt/gravwellCustomers/uploads/crash/webserver_3.3.5_X.X.X.X_2020-01-31T14:39:42


Log Snippet:
Version         3.3.10
API Version     0.1
Build Date      2020-Apr-30
Build ID        745dc6ca
Cmdline         /opt/gravwell/bin/gravwell_webserver -stderr gravwell_webserver.service
Executing user  gravwell
Parent PID      1
Parent cmdline  /sbin/init
Parent user     root
Total memory    4147781632
Memory used     5.781707651865122%
Process heap allocation 2005776
Process sys reserved    72112017
CPU count       4
Host hash       4224be94ae35247ed32013d9021f64bc40986c9fbbafac97787ab58b400f1666
Virtualization role     guest
Virtualization system   kvm
max_map_count   65530
RLIMIT_AS (address space)       18446744073709551615 / 18446744073709551615
RLIMIT_DATA (data seg)  18446744073709551615 / 18446744073709551615
RLIMIT_FSIZE (file size)        18446744073709551615 / 18446744073709551615
RLIMIT_NOFILE (file count)      1048576 / 1048576
RLIMIT_STACK (stack size)       8388608 / 18446744073709551615
SKU             2UX
Customer NUM    00000000
Customer GUID   ffffffff-ffff-ffff-ffff-ffffffffffff

panic: send on closed channel

goroutine 90 [running]:
gravwell/pkg/search.(*SearchManager).clientSystemStatsRoutine(0xc01414edc0,
0xc00017fd20, 0x16, 0xc000c300c0, 0xc000c1dc80)
        gravwell@/pkg/search/manager_handlers.go:284 +0x106
created by gravwell/pkg/search.(*SearchManager).GetSystemStats.func1
        gravwell@/pkg/search/manager_handlers.go:301 +0x65
```

メッセージはクラッシュした特定のコンポーネント(この場合はウェブサーバー)から始まる。その後、Gravwellのバージョン、クラッシュしたシステムのIPとホスト名、Gravwellのスタッフがクラッシュログの完全なコピーを見つけられるパスがリストアップされます。

メッセージの残りの部分は、クラッシュしたプログラムからのコンソール出力です。クラッシュレポーターはこれを`/dev/shm`にあるコンポーネントの出力ファイルから直接取得します。過去のクラッシュレポートは`/opt/gravwell/log/crash`で見ることができますが、クラッシュレポーターを*無効*にすると、クラッシュログはそのディレクトリに保存されなくなることに注意してください。

最初の数行(Version, API Version, Build Date, Build ID)は、Gravwellが実行されていたバージョンを正確に判断するのに役立ちます。"Cmdline"、"Executing user"、"Parent PID"、"Parent cmdline"、"Parent user "はすべて、Gravwellプロセスがどのように実行されているかを把握し、そこにある潜在的な問題を特定するのに役立ちます - この例では、親プロセスがPID 1で "manager "という名前であることから、GravwellがDockerコンテナで実行されていると推測できます。ある環境(例: "manager "によって起動されるDocker)では問題が発生することがありますが、他の環境(Ubuntu、systemd経由での起動)では問題が発生しないことがありますが、これはそれを追いかけるのに役立ちます。

また、システムのメモリ量とrlimitsの設定についての情報も含めています。これは、特定のクラスのクラッシュを追跡するのに役立つからです。ホストハッシュ "フィールドはプロセスを実行しているホストの一意の識別子ですが、ハッシュなので、"これは他のクラッシュレポートと同じマシンである "という意味でのみ使用でき、他の情報は含まれません。

「SKU」、「Customer NUM」、および「Customer GUID」フィールドは、使用中のライセンスを記述します。この場合、Gravwellの従業員は無制限（「UX」）ライセンスを使用しています。顧客番号と顧客GUIDフィールドは、顧客データベースを参照し、問題を抱えている人を確認することを可能にします。

これらの情報の下に、Gravwell のプロセスからのバックトレースがあります。この場合、アルファビルドのバグが GUI のハードウェア統計ページで使用するためにウェブサーバがインデクサーの CPU/メモリ情報をチェックするために使用するルーチンでクラッシュを引き起こしたことがわかります。スタックトレースには決してユーザデータは含まれず、ソースコードの行番号だけが含まれていることを明確にしておきます。

### クラッシュレポートの無効化

何らかの理由でクラッシュレポートを送信したくないと判断した場合、レポートシステムを無効にするには複数のオプションがあります。

* スタンドアロンシェルインストーラを使用している場合は、インストール時に `--no-crash-report` フラグで無効化できます。
* Debian リポジトリから Gravwell をインストールした場合は、`systemctl disable gravwell_crash_report` で無効化できます。
* Gravwell Docker イメージを使用している場合は、Docker コマンドに `-e DISABLE_ERROR_HANDLING=true` を渡すことでクラッシュレポーターを無効にすることができます。

ただし、クラッシュレポートを有効にしていただけると本当にありがたいです。このようなクラッシュレポートのおかげで、ユーザーが気づかないような問題を特定して修正することができます。これは、当社のソフトウェアを改善するための最高のフィードバックメカニズムの一つです。

お客様のシステムが送信した過去のクラッシュレポートの削除を希望される場合は、support@gravwell.ioまで電子メールをお送りください。

## メトリクスのレポート

Gravwell ウェブサーバコンポーネント(ウェブサーバ*のみ*)は、一般的な使用統計情報を含むHTTPS POST要求をGravwell企業サーバに送信することがあります。この情報は、どの機能が最もよく使われ、どの機能がより多くの作業を必要とするかを把握するのに役立ちます。ガベージコレクションを最適化する必要があるのか、デフォルトの構成でより保守的にする必要があるのかなど、GravwellがどのくらいのRAMを消費しているかについての統計を生成することができます。また、有料ライセンスが不適切にデプロイされていないかどうかを確認することもできます。

これらのメトリクスを収集する上で最も重要な目標は、あなたのデータの匿名性を保護することです。これらのメトリクスレポートには、Gravwellに保存されているデータの実際の内容が含まれることはありませんし、実際の検索クエリやシステム上のタグのリストすら送信されることはありません。

注:メトリクスレポートは、常にTLSで検証されたHTTPS経由でupdate.gravwell.ioに送信されます。リモート証明書を完全に検証できなかった場合、レポートは出力されません。

メトリクスレポートが送信されると、サーバーは最新バージョンのGravwellで応答します。これにより、新しいバージョンが利用可能になったときに、Gravwell UIに通知を表示することができます(これらの通知は、gravwell.confの`Disable-Update-Notification`パラメータで無効にすることができます)。

ここでは、Gravwellの社員の自宅システムから送られてきた例を紹介します。

```
{
    "ApiVer": {
        "Major": 0,
        "Minor": 1
    },
    "AutomatedSearchCount": 1,
    "BuildVer": {
        "BuildDate": "2020-04-02T00:00:00Z",
        "BuildID": "e755ee13",
        "GUIBuildID": "87e5e523",
        "Major": 3,
        "Minor": 3,
        "Point": 8
    },
    "CustomerNumber": 000000000,
    "CustomerUUID": "ffffffff-ffff-ffff-ffff-ffffffffffff",
    "DashboardCount": 5,
    "DashboardLoadCount": 13,
    "DistributedFrontends": false,
    "ForeignDashboardLoadCount": 0,
    "ForeignSearchLoadCount": 0,
    "Groups": 2,
    "IndexerCount": 4,
    "IndexerNodeInfo": [
        {
            "CPUCount": 12,
            "HostHash": "90578d2dcc5bea54614528e1b2c5a25c261cdd7c945f763d2387f309bdd38816",
            "ProcessHeapAllocation": 47899944,
            "ProcessSysReserved": 282423040,
            "TotalMemory": 67479150592,
            "VirtRole": "guest",
            "VirtSystem": "docker"
        },
        {
            "CPUCount": 12,
            "HostHash": "90578d2dcc5bea54614528e1b2c5a25c261cdd7c945f763d2387f309bdd38816",
            "ProcessHeapAllocation": 66157568,
            "ProcessSysReserved": 282554112,
            "TotalMemory": 67479150592,
            "VirtRole": "guest",
            "VirtSystem": "docker"
        },
        {
            "CPUCount": 12,
            "HostHash": "90578d2dcc5bea54614528e1b2c5a25c261cdd7c945f763d2387f309bdd38816",
            "ProcessHeapAllocation": 58577296,
            "ProcessSysReserved": 351827712,
            "TotalMemory": 67479150592,
            "VirtRole": "guest",
            "VirtSystem": "docker"
        },
        {
            "CPUCount": 12,
            "HostHash": "90578d2dcc5bea54614528e1b2c5a25c261cdd7c945f763d2387f309bdd38816",
            "ProcessHeapAllocation": 58304584,
            "ProcessSysReserved": 282226432,
            "TotalMemory": 67479150592,
            "VirtRole": "guest",
            "VirtSystem": "docker"
        }
    ],
    "IndexerStats": [
        {
            "WellStats": [
                {
                    "Cold": false,
                    "Data": 658757162,
                    "Entries": 2770447
                },
                {
                    "Cold": false,
                    "Data": 12681258,
                    "Entries": 9882
                },
                {
                    "Cold": false,
                    "Data": 325462303,
                    "Entries": 1344586
                },
                {
                    "Cold": false,
                    "Data": 0,
                    "Entries": 0
                },
                {
                    "Cold": false,
                    "Data": 45312907669,
                    "Entries": 119150365
                },
                {
                    "Cold": false,
                    "Data": 0,
                    "Entries": 0
                },
                {
                    "Cold": false,
                    "Data": 50161444,
                    "Entries": 297743
                }
            ]
        },
        {
            "WellStats": [
                {
                    "Cold": false,
                    "Data": 669469662,
                    "Entries": 2931573
                },
                {
                    "Cold": false,
                    "Data": 0,
                    "Entries": 0
                },
                {
                    "Cold": false,
                    "Data": 325986097,
                    "Entries": 1348645
                },
                {
                    "Cold": false,
                    "Data": 50301788,
                    "Entries": 298556
                },
                {
                    "Cold": false,
                    "Data": 45316008062,
                    "Entries": 119174395
                },
                {
                    "Cold": false,
                    "Data": 12341038,
                    "Entries": 9559
                },
                {
                    "Cold": false,
                    "Data": 0,
                    "Entries": 0
                }
            ]
        },
        {
            "WellStats": [
                {
                    "Cold": false,
                    "Data": 663669955,
                    "Entries": 2782081
                },
                {
                    "Cold": false,
                    "Data": 326449600,
                    "Entries": 1350525
                },
                {
                    "Cold": false,
                    "Data": 50427080,
                    "Entries": 299538
                },
                {
                    "Cold": false,
                    "Data": 12552734,
                    "Entries": 9759
                },
                {
                    "Cold": false,
                    "Data": 0,
                    "Entries": 0
                },
                {
                    "Cold": false,
                    "Data": 45445347364,
                    "Entries": 119473828
                },
                {
                    "Cold": false,
                    "Data": 0,
                    "Entries": 0
                }
            ]
        },
        {
            "WellStats": [
                {
                    "Cold": false,
                    "Data": 660249138,
                    "Entries": 2794164
                },
                {
                    "Cold": false,
                    "Data": 45332590720,
                    "Entries": 119204014
                },
                {
                    "Cold": false,
                    "Data": 50572152,
                    "Entries": 300251
                },
                {
                    "Cold": false,
                    "Data": 12608944,
                    "Entries": 9730
                },
                {
                    "Cold": false,
                    "Data": 0,
                    "Entries": 0
                },
                {
                    "Cold": false,
                    "Data": 0,
                    "Entries": 0
                },
                {
                    "Cold": false,
                    "Data": 325899751,
                    "Entries": 1347670
                }
            ]
        }
    ],
    "IngesterCount": 8,
    "LicenseHash": "kH3R+R4AdTCnXFYDi3L4nZ==",
    "LicenseTimeLeft": 23550517079782204,
    "ManualSearchCount": 330,
    "ResourceUpdates": 13356,
    "ResourcesCount": 4,
    "SKU": "2UX",
    "ScheduledSearchCount": 4,
    "SearchCount": 331,
    "Source": "X.X.X.X",
    "SystemMemory": 67479150592,
    "SystemProcs": 3,
    "SystemUptime": 1920449,
    "TimeStamp": "2020-04-02T22:11:23Z",
    "TotalData": 185614443921,
    "TotalEntries": 494907311,
    "Uptime": 300,
    "UserLoginCount": 27,
    "Users": 2,
    "WebserverNodeInfo": {
        "CPUCount": 12,
        "HostHash": "90578d2dcc5bea54614528e1b2c5a25c261cdd7c945f763d2387f309bdd38816",
        "ProcessHeapAllocation": 311618224,
        "ProcessSysReserved": 420052881,
        "TotalMemory": 67479150592,
        "VirtRole": "guest",
        "VirtSystem": "docker"
    },
    "WebserverUUID": "17405830-3ac4-4b75-a639-6a265e6718a4",
    "WellCount": 28
}
```

このウェブサーバは、それぞれが独自の情報セットを取得する4つのインデクサに接続されているため、構造が大きくなっています。以下にフィールドの詳細な内訳を示します。

* `ApiVer`: Gravwell API の内部バージョン番号。
* `AutomatedSearchCount`: 自動的に実行された検索数（検索エージェントによる、またはダッシュボードの読み込みによる）。
* `BuildVer`: このシステム上でのGravwellの特定のビルドを記述した構造。
* `CustomerNumber`: このシステムのライセンスに関連付けられた顧客番号。
* `CustomerUUID`: このシステム上のライセンスのUUID。
* `DashboardCount`: 存在するダッシュボードの数。
* `DashboardLoadCount`: 任意のユーザが開いたダッシュボードの種類の数。
* `DistributedFrontends`: [分散型ウェブサーバ](#!distributed/frontend.md)が有効な場合にtrueを設定します。
* `ForeignDashboardLoadCount`: 他のユーザーが所有するダッシュボードをユーザーが閲覧した回数（ダッシュボードの共有オプションが十分に柔軟であるかどうかを判断するのに役立ちます。
* `ForeignSearchLoadCount`: 他のユーザーが所有する検索をユーザーが閲覧した回数（検索の共有オプションが十分に柔軟であるかどうかを判断するのに役立ちます。
* `Groups`: システム上のユーザーグループの数です。
* `IndexerCount`: このウェブサーバが接続されているインデクサの数。
* `IndexerNodeInfo`: 構造体の配列で，各インデクサーごとに1つずつ，各インデクサーの統計情報を簡単に記述します。
	- `CPUCount`:  インデクサのCPUコア数。
	- `HostHash`: インデクサを実行しているホストマシンを一意に識別する非可逆ハッシュ（[github.com/denisbrodbeck/machineid](https://github.com/denisbrodbeck/machineid)を参照）。この例では、すべてのインデクサが単一のDockerホスト上で動作しているので、すべて同じHostHashを持っていることに注意してください。
	- `ProcessHeapAllocation`: インデクサプロセスが割り当てたヒープメモリの量。
	- `ProcessSysReserved`: インデクサプロセスがOSから予約しているメモリの総量。
	- `TotalMemory`: システムのメインメモリのサイズ。
	- `VirtRole`: "host" または "guest" で、インデクサが仮想マシン上で動作しているかどうかによって異なります。
	- `VirtSystem`: 仮想化システムがあれば、"xen", "kvm", "vbox", "vmware", "docker", "openvz", "lxc" など。
* `IndexerStats`: 各インデクサの統計構造の配列。
	* `WellStats`: インデクサ上の各ウェルに関する匿名化された情報の配列。
		* `Cold`: これが"冷たい"ウェルかどうか。
		* `Data`: このウェル内のデータのバイト数。
		* `Entries`: このウェルのエントリー数。
* `IngesterCount`: システムに添付されているユニークなインゲスターの数。
* `LicenseHash`: 使用しているライセンスのMD5の合計。
* `LicenseTimeLeft`: ライセンスの残り数です。
* `ManualSearchCount`: 手動で実行した検索回数。
* `ResourceUpdates`: いずれかのリソースが変更された回数。
* `ResourcesCount`: システム上のリソースの数。
* `SKU`: 使用中のライセンスのSKUです。
* `ScheduledSearchCount`: システムにインストールされている定期検索の数です。
* `SearchCount`: 非推奨のフィールドで、`ManualSearchCount` + `AutomatedSearchCount` の合計。
* `Source`: このレポートが発信されたIPです。
* `SystemMemory`: ウェブサーバのホストシステムにインストールされているメモリのバイト数。
* `SystemProcs`: ホストシステム上で実行されているプロセスの数です。
* `SystemUptime`:ホストシステムが実行されている秒数です。
* `TimeStamp`: このレポートが生成された時刻。
* `TotalData`: すべてのインデクサ上のすべてのウェルにまたがるバイト数です。
* `TotalEntries`: すべてのインデクサー上のすべてのウェルに渡るエントリ数です。
* `Uptime`: ウェブサーバのプロセスが開始されてからの秒数です。
* `UserLoginCount`:  ユーザーがログインした回数です。
* `Users`: システムの利用者数です。
* `WebserverNodeInfo`: ウェブサーバプロセスを実行しているシステムの簡単な説明：
	- `CPUCount`: ウェブサーバのCPUコア数です。
	- `HostHash`: ウェブサーバを実行しているホストマシンを一意に識別する非可逆ハッシュ ( [github.com/denisbrodbeck/machineid](https://github.com/denisbrodbeck/machineid) を参照)。
	- `ProcessHeapAllocation`: ウェブサーバプロセスが割り当てたヒープメモリの量です。
	- `ProcessSysReserved`: ウェブサーバプロセスがOSから予約しているメモリの総量です。
	- `TotalMemory`: システムのメインメモリのサイズです。
	- `VirtRole`: "host" または "guest" は、ウェブサーバが仮想マシンで動作しているかどうかによって異なります。
	- `VirtSystem`: 仮想化システムがあれば、"xen", "kvm", "vbox", "vmware", "docker", "openvz", "lxc" など。
* `WebserverUUID`: すべてのGravwellウェブサーバーはインストール時にUUIDを生成します。
* `WellCount`: すべてのインデクサーにまたがるウェルの総数です。

私たちは、Gravwellで取得したデータの種類や内容に関する情報を得ることができないように配慮しながら、報告する情報を慎重に検討しています。当社が収集した情報の背後にある理由については、いつでも喜んでご相談に応じます。

### メトリックレポートの制限

カスタマはgravwell.confで`Disable-Stats-Report=true`を設定することができます。これにより、メトリクスレポートは、CustomerUUID、CustomerNumber、BuildVer、ApiVer、LicenseTimeLeft、およびLicenseHashフィールドを含む最小限のものにまで削除されます。

しかし、完全な統計レポートを有効にしていただけると本当にありがたいです。上で述べたように、これらの統計レポートは、どの機能が最も使用されているか、Gravwellがどのようなシステム上で実行されているか、どのくらいのRAMを使用しているかを把握するのに役立ちます。