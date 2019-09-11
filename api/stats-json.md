# REST統計API

## Ping

ping統計APIは、Webサーバーとインデクサーのステータスを返します。 これは/ api / stats / pingへのGETで取得されます。 すべてのシステムが起動している場合、それらは "OK"として報告されます。

```
{"192.168.2.60:9404":"OK","webserver":"OK"}
```

インデクサーが停止している場合は、「切断」とマークされます。

```
{"192.168.2.60:9404":"Disconnected","webserver":"OK"}
```

## インデックス統計

インデクサー統計APIは、各インデクサーの索引に関する情報を提供します。 / api / stats / idxStatsへのGETを介してアクセスされます。

```
{
    "192.168.2.60:9404": {
        "IndexStats": [
            {
                "Name": "default",
                "Stats": [
                    {
                        "Cold": false,
                        "Data": 461610307,
                        "Entries": 4327438,
                        "Path": "/opt/gravwell/storage/default"
                    },
                    {
                        "Cold": true,
                        "Data": 3637724726,
                        "Entries": 33315409,
                        "Path": "/opt/gravwell/cold-storage/default"
                    }
                ]
            },
            {
                "Name": "csv",
                "Stats": [
                    {
                        "Accelerator": "index",
                        "Cold": false,
                        "Data": 69904,
                        "Entries": 0,
                        "Extractor": "csvAcceleratorV1",
                        "Path": "/opt/gravwell/storage/csv"
                    }
                ]
            },
[...]
            {
                "Name": "test",
                "Stats": [
                    {
                        "Accelerator": "index",
                        "Cold": false,
                        "Data": 775980546,
                        "Entries": 2000000,
                        "Extractor": "jsonAcceleratorV1",
                        "Path": "/opt/gravwell/storage/test2"
                    }
                ]
            }
        ]
    }
}

```

## インジェスター統計

GET要求を/api/stats/igstStatsに送信すると、各インデクサーに接続されているインジェスターを記述する構造体が返されます。 以下の例は、2つのインジェスターが接続された単一のインデクサー（192.168.2.60）を示しています。SimpleRelayインジェスターとNetwork Captureインジェスターです。

```
{
    "192.168.2.60:9404": {
        "Ingesters": [
            {
                "Count": 6,
                "RemoteAddress": "unix://@",
                "Size": 659,
                "Tags": [
                    "bro",
                    "default",
                    "syslog"
                ],
                "Uptime": 5639681950
            },
            {
                "Count": 3,
                "RemoteAddress": "tcp://192.168.2.60:43684",
                "Size": 229,
                "Tags": [
                    "pcap"
                ],
                "Uptime": 2846761051
            }
        ],
        "QuotaMax": 0,
        "QuotaUsed": 0,
        "TotalCount": 9,
        "TotalSize": 888
    }
}
```

各インジェスターの説明で、「カウント」フィールドは取り込まれたエントリーの数を示し、「サイズ」フィールドは取り込まれたバイト数を示します。 「稼働時間」とは、摂取者が接続されている時間（ナノ秒単位）です。

"QuotaMax"と "QuotaUsed"フィールドに注意してください。 コミュニティライセンスは毎日一定量しか摂取できません。 「QuotaMax」は、指定されたインデクサーが1日に取り込めるバイト数を指定します。 "QuotaUsed"は、今日までに何バイトが取り込まれたかを示します。

## システム統計

/api/stats/sysStatsにGETリクエストを送信すると、CPUの数、CPU使用率、メモリとネットワークの使用率など、インデクサーとWebサーバーシステムに関する情報を含む構造体が返されます。

```
{
    "192.168.2.60:9404": {
        "Stats": {
            "CPUCount": 4,
            "CPUUsage": 28.717948741951247,
            "Disks": [
                {
                    "Mount": "/",
                    "Partition": "/dev/mapper/foo--vg-root",
                    "Total": 233377820672,
                    "Used": 170719322112
                }
            ],
            "HostHash": "bef3ac74c4bd31fc15d37bbbd927ea7213d7ea0d922126ed07c34e2c41a9ca12",
            "IO": [
                {
                    "Device": "nvme0n1p2",
                    "Read": 0,
                    "Write": 0
                },
                {
                    "Device": "foo--vg-root",
                    "Read": 0,
                    "Write": 0
                },
                {
                    "Device": "nvme0n1p1",
                    "Read": 0,
                    "Write": 0
                },
                {
                    "Device": "sda1",
                    "Read": 0,
                    "Write": 0
                }
            ],
            "MemoryUsedPercent": 39.42591390055748,
            "Net": {
                "Down": 1160,
                "Up": 310
            },
            "TotalMemory": 16721588224,
            "Uptime": 15178980
        }
    },
    "webserver": {
        "Stats": {
            "CPUCount": 4,
            "CPUUsage": 28.589743582518338,
            "Disks": [
                {
                    "Mount": "/boot",
                    "Partition": "/dev/nvme0n1p2",
                    "Total": 247772160,
                    "Used": 108133376
                },
                {
                    "Mount": "/",
                    "Partition": "/dev/mapper/foo--vg-root",
                    "Total": 233377820672,
                    "Used": 170719322112
                }
            ],
            "HostHash": "bef3ac74c4bd31fc15d37bbbd927ea7213d7ea0d922126ed07c34e2c41a9ca12",
            "IO": [
                {
                    "Device": "nvme0n1p1",
                    "Read": 0,
                    "Write": 0
                },
                {
                    "Device": "foo--vg-root",
                    "Read": 0,
                    "Write": 0
                },
                {
                    "Device": "sda1",
                    "Read": 0,
                    "Write": 0
                },
                {
                    "Device": "nvme0n1p2",
                    "Read": 0,
                    "Write": 0
                }
            ],
            "MemoryUsedPercent": 39.42591390055748,
            "Net": {
                "Down": 747,
                "Up": 255
            },
            "TotalMemory": 16721588224,
            "Uptime": 15178980
        }
    }
}
```

ほとんどのフィールドは一目瞭然です。 "IO"配列はディスクに関する情報を提供し、 "Read"および "Write"フィールドは1秒あたりのバイト数で読み書き速度を指定します。 同様に、「Net」コンポーネントは、ネットワーク使用率を1秒あたりのバイト数で表します。

## システムの説明

/api/stats/sysDescにGETリクエストを送信すると、Webサーバーとインデクサーのホストシステムに関する追加情報を示す構造体が返されます。

```
{
    "192.168.2.60:9404": {
        "CPUCache": "4096",
        "CPUCount": 4,
        "CPUMhz": "3500",
        "CPUModel": "Intel(R) Core(TM) i7-7500U CPU @ 2.70GHz",
        "SystemVersion": "4.9.0-8-amd64",
        "TotalMemoryMB": 15946
    },
    "webserver": {
        "CPUCache": "4096",
        "CPUCount": 4,
        "CPUMhz": "3500",
        "CPUModel": "Intel(R) Core(TM) i7-7500U CPU @ 2.70GHz",
        "SystemVersion": "4.9.0-8-amd64",
        "TotalMemoryMB": 15946
    }
}
```