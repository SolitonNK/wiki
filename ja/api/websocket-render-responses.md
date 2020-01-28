# Response formats

すべてのモジュールは同じコマンドに応答しますが、関連するデータ型の性質が異なるため、エントリを返す形式は異なります。 ここで説明する応答は、RESP_GET_ENTRIES、RESP_STREAMING、RESP_TS_RANGEに共通です。 このセクションの例では、これらのリクエスト/レスポンスIDを無差別に使用しています。

## textおよびRawのモジュール応答

「text」および「raw」レンダリングモジュールは、「エントリ」というラベルのフィールドに配列としてエントリを返します。

```
{
	"ID": 16,
	"EntryCount": 1575,
	"AdditionalEntries": false,
	"Finished": true,
	"Entries": [
		{
			"TS":"2018-04-02T16:16:39-06:00",
			"SRC":"",
			"Tag":0,"Data"<elided>",
			"Enumerated": [
				{"Name":"count","Value":{"Type":9,"Data":1}},
				{"Name":"SrcIP","Value":{"Type":13,"Data":"10.177.98.189"}}
			]
		},
<entries elided>
		{
			"TS":"2018-04-02T16:16:39-06:00",
			"SRC":"",
			"Tag":0,
			"Data":"<elided>",
			"Enumerated": [
				{"Name":"count","Value":{"Type":9,"Data":1}},
				{"Name":"SrcIP","Value":{"Type":13,"Data":"35.174.22.108"}}
			]
		}
	]
}
```

## テーブルモジュールの応答

テーブルモジュールは、行と列を定義する構造を含む「エントリ」というフィールドにエントリを返します。

```
{
	"ID": 16,
	"EntryCount": 1575,
	"AdditionalEntries": false,
	"Finished": true,
	"Entries": {
		"Rows": [
			{
				"TS": "2018-04-02T10:30:29-06:00",
				"Row": [
					"10.144.162.236",
					"9410"
				]
			},
<67 similar entries elided>
			{
				"TS": "2018-04-02T10:28:51-06:00",
				"Row": [
					"192.168.1.1",
					"2"
				]
			}
		],
		"Columns": [
			"SrcIP",
			"count"
		]
	}
}
```

## ゲージモジュールの応答

ゲージモジュールは、ゲージの名前、大きさ、および（オプションで）このゲージに定義された最小値と最大値を含む構造体の配列としてエントリを返します。

```
{
	"ID": 16,
	"EntryCount": 1,
	"AdditionalEntries": true,
	"Finished": true,
	"Entries": [
		{
			"Name": "mean",
			"Magnitude": "31691.213",
			"Min": "0",
			"Max": "64000"
		}
	]
}
```

## Point-to-Pointモジュールの応答

point2pointモジュールは、DstLocation、SrcLocation、およびMagnitudeフィールドを含むエントリの配列を返します。 オプションで、エントリにはレンダラーへの引数として指定された追加の列挙値を含む「値」配列も含まれる場合があります。 これらの列挙値の名前は、「ValueNames」配列で指定されます。

クエリ

```
tag=pcap packet tcp.Port ipv4.SrcIP ipv4.DstIP ipv4.Length | geoip SrcIP.Location as srcloc DstIP.Location as dstloc | sum Length by srcloc dstloc | point2point -srcloc srcloc -dstloc dstloc -mag sum SrcIP DstIP
```

次のような結果が生成されます。

```
{
    "AdditionalEntries": true,
    "Entries": [
        {
            "DstLocation": "33.381516 -108.391164",
            "Magnitude": 420471,
            "SrcLocation": "34.054400 -118.244000",
            "Values": [
                "151.11.24.133",
                "192.168.2.60"
            ]
        },
        {
            "DstLocation": "33.381516 -108.391164",
            "Magnitude": 373204,
            "SrcLocation": "52.382400 5.899500",
            "Values": [
                "185.19.10.154",
                "192.168.2.60"
            ]
        },
        {
            "DstLocation": "33.381516 -108.391164",
            "Magnitude": 246593,
            "SrcLocation": "39.048100 -76.472800",
            "Values": [
                "53.1.11.28",
                "192.168.2.60"
            ]
        },
[...]
        {
            "DstLocation": "32.769700 -122.393300",
            "Magnitude": 8662,
            "SrcLocation": "33.381516 -108.391164",
            "Values": [
                "192.168.2.60",
                "192.33.23.124"
            ]
        }
    ],
    "EntryCount": 16,
    "Finished": true,
    "ID": 18,
    "ValueNames": [
        "SrcIP",
        "DstIP"
    ]
}
```

各エントリの「Values」配列は、「ValueNames」配列のタイトルに対応していることに注意してください。 最初のエントリの151.11.24.133の「SrcIP」があります。

## チャートモジュールの応答

チャートモジュールは、「名前」と「値」を定義する構造を含む「エントリ」というフィールドにエントリを返します。 「名前」コンポーネントは、プロットされる線の名前の配列です。 この例の場合、IPアドレスが含まれています。 「値」コンポーネントには、タイムスタンプと「データ」配列が含まれています。 Data配列の要素は、指定されたタイムスタンプでの「Names」配列の名前に対応する値です。
```
{
    "EntryCount": 5,
    "Finished": true,
    "ID": 18
    "AdditionalEntries": false,
    "Entries": {
        "Names": [
            "10.177.98.189",
            "192.168.1.101",
            "192.168.1.50",
            "10.174.22.108",
            "10.203.116.250",
            "10.21.206.134"
        ],
        "Values": [
            {
                "Data": [
                    0,
                    0,
                    0,
                    0,
                    0,
                    0
                ],
                "TS": "2018-04-02T15:22:13.815-06:00"
            },
<elided>
            {
                "Data": [
                    1,
                    7,
                    2,
                    1,
                    1,
                    1
                ],
                "TS": "2018-04-02T16:16:13.815-06:00"
            },
<elided>
            {
                "Data": [
                    0,
                    0,
                    0,
                    0,
                    0,
                    0
                ],
                "TS": "2018-04-02T16:21:28.815-06:00"
            }
        ]
    }
}
```

## 有向グラフ応答の強制

fdgモジュールの応答は最も複雑です。 返されるデータには、グループ、リンク、ノードの3つのセクションがあります。

ノードとリンクはグラフを表します。 各ノードには、名前と所属するグループがあります。 リンクは方向性があるため、ノードの配列へのインデックスとしてソースノードとターゲットノードを指定し、さらにリンクの「重み」を表す「値」を指定します（この例では、値は常に1ですが、 しばしば大きくなります）。

グループは検索で定義され、fdg表示でノードを色付けするために使用されます。 この例には、「オペレーション」、「IT」、および名前のないグループの3つのグループがあります。 各ノードは、グループ配列のインデックスによって参照されるグループのいずれかに属します。

```
{
    "AdditionalEntries": false,
    "Entries": {
        "groups": [
            "",
            "operations",
            "IT"
        ],
        "links": [
            {
                "source": 0,
                "target": 1,
                "value": 1
            },
            {
                "source": 2,
                "target": 1,
                "value": 1
            },
            {
                "source": 3,
                "target": 1,
                "value": 1
            },
            {
                "source": 2,
                "target": 4,
                "value": 1
            },
            {
                "source": 4,
                "target": 5,
                "value": 1
            },
            {
                "source": 0,
                "target": 6,
                "value": 1
            },
            {
                "source": 2,
                "target": 7,
                "value": 1
            },
            {
                "source": 2,
                "target": 8,
                "value": 1
            },
            {
                "source": 9,
                "target": 8,
                "value": 1
            },
            {
                "source": 10,
                "target": 5,
                "value": 1
            },
            {
                "source": 11,
                "target": 8,
                "value": 1
            },
            {
                "source": 2,
                "target": 12,
                "value": 1
            },
            {
                "source": 2,
                "target": 13,
                "value": 1
            },
            {
                "source": 14,
                "target": 12,
                "value": 1
            },
            {
                "source": 15,
                "target": 12,
                "value": 1
            },
            {
                "source": 16,
                "target": 12,
                "value": 1
            },
            {
                "source": 17,
                "target": 13,
                "value": 1
            },
            {
                "source": 18,
                "target": 13,
                "value": 1
            },
            {
                "source": 13,
                "target": 19,
                "value": 1
            }
        ],
        "nodes": [
            {
                "group": 0,
                "name": "bbd307455de9"
            },
            {
                "group": 1,
                "name": "operations-5 OPERATIONS-5$"
            },
            {
                "group": 0,
                "name": "9b10deadbeef"
            },
            {
                "group": 0,
                "name": "db48a5920a82"
            },
            {
                "group": 2,
                "name": "desktop-2 DESKTOP-2$"
            },
            {
                "group": 2,
                "name": "e758bb7d2630"
            },
            {
                "group": 1,
                "name": "operations-2 OPERATIONS-2$"
            },
            {
                "group": 2,
                "name": "desktop-3 DESKTOP-3$"
            },
            {
                "group": 2,
                "name": "desktop-4 DESKTOP-4$"
            },
            {
                "group": 0,
                "name": "4f194d5cf71a"
            },
            {
                "group": 0,
                "name": "desktop-1 DESKTOP-1$"
            },
            {
                "group": 0,
                "name": "6"
            },
            {
                "group": 1,
                "name": "operation-desktop OPERATION-DESKT$"
            },
            {
                "group": 2,
                "name": "DESKTOP-67T38GD DESKTOP-67T38GD$"
            },
            {
                "group": 0,
                "name": "2fd6276575c7"
            },
            {
                "group": 0,
                "name": "dfc56224743c"
            },
            {
                "group": 0,
                "name": "cb7b71a72272"
            },
            {
                "group": 0,
                "name": "2f01fbc81c46"
            },
            {
                "group": 0,
                "name": "379bd32ecec6"
            },
            {
                "group": 2,
                "name": "foobar"
            }
        ]
    },
    "EntryCount": 19,
    "Finished": true,
    "ID": 18
}
```