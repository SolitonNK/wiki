# Named Fields

namedfieldsモジュールは、後で使用するために検索エントリから列挙値にデータを抽出してフィルタ処理するために使用されます。  [fieldsモジュール](#!search/fields/fields.md)と同じように、バイトのシーケンスで区切られたレコードからフィールドを抽出します。  ただし、fieldsモジュールがインデックスを使って要素を参照する場合（たとえば`fields -d "\t" [5] as foo`、 "6番目の要素を抽出してfooと呼ぶ"など）、namedfieldsは特別にフォーマットされた[リソース](#!resources/resources.md)を使って特定のデータフォーマットにわかりやすい名前を付けます。  これは、[Bro](https://www.bro.org)ログやCSVファイルなどを解析しようとするときに特に便利です。

非常に多くの人がBroを使用しているので、Broフィールドをデコードするためのリソースファイルを提供しています。[https://github.com/gravwell/resources](https://github.com/gravwell/resources)で、bro/namedfieldsサブディレクトリ。  namedfields.jsonGravwellにリソースとしてアップロードするだけです。このドキュメントの例では、`brofields`という名前のリソースにアップロードされたと仮定しています。

## 主な概念

namedfieldsモジュールは、その核となる部分で、行内の数値インデックスをユーザーフレンドリな名前にマッピングします。  一連の名前とインデックスのマッピングはグループと呼ばれます。  ネットワークフローのテキスト表現を解析するためのグループは、次のようなマッピングがあります:

| 索引 | 名 |
|-------|------|
| 0 | start_time |
| 1 | duration |
| 2 | protocol |
| 3 | src_ip |
| 4 | src_port |
| 5 | dst_ip |
| 6 | dst_port |
| 7 | packets |
| 8 | bytes |

次に、1つ以上のグループが、このドキュメントの他の場所で指定されている形式でGravwellリソースにまとめられます。  namedfieldsが実行されると、ユーザーはロードするリソースとそのリソース内のどのグループを使用してユーザー指定の名前をインデックスにマッピングするかを指定します。

## サポートされているオプション

* `-r <arg>`: "-r"オプションは必須です。  名前とインデックスのマッピングを含むリソースの名前またはGUIDを指定しました。
* `-g <arg>`: "-g"オプションは必須です。  指定されたリソース内で使用するグループを指定します。
* `-e <arg>`: “ -e”オプションは、レコード全体ではなく列挙値に作用します。
* `-s` : “ -s”オプションはnamedfieldsモジュールが厳密モードで動作することを指定します。  ファイルされた指定を満たすことができない場合は、そのエントリは削除されます。  たとえば、0番目、1番目、2番目のフィールドが必要で、エントリに2つのフィールドしかない場合は、strictフラグによってエントリが削除されます。

## フィルタリング演算子

namedfieldsモジュールは等式に基づくフィルタリングを可能にします。  等価（ "等しい"、 "等しくない"、 "含む"、 "含まない"）を指定するフィルタが有効になっている場合、フィルタの指定に失敗したエントリはすべて削除されます。  フィールドが等しくない "！="として指定され、そのフィールドが存在しない場合、そのフィールドは抽出されませんが、エントリは完全にはドロップされません。

| オペレーター | 名 | 説明 |
|----------|------|-------------|
| == | 等しい | フィールドは等しくなければなりません
| != | 等しくない | フィールドは等しくてはいけません
| ~ | サブセット | フィールドに値が含まれています
| !~ | サブセットではない | フィールドに値が含まれていません

## 例

Broのconn.logファイルに "broconn"フラグが含まれていると仮定すると、以下は各エントリから "service"、 "dst"、および "resp_bytes"フィールドを抽出します。  「service」フィールドが文字列「dns」と一致しないすべてのエントリを削除し、抽出された「dst」フィールドの名前を「server」に変更します。  その後、各サーバーのDNS応答の平均長を計算してグラフ化します。  "brofields"という名前のリソースと "conn"という名前のグループを指定します。  これは "brofields"リソース内で定義されています。

```
tag=broconn namedfields -r brofields -g Conn service==dns dst as server resp_bytes  | mean resp_bytes by server | chart mean by server
```

T次の例では、異なるBroファイルintel.logを解析します。  リソースは同じですが、異なるグループを指定していることに注意してください:

```
tag=brointel namedfields -r brofields -g Intel source | count source | table source count
```

## Named fieldsリソースフォーマット

namedfieldsモジュールを使用する前に、名前をフィールド内のインデックスにマッピングするためのリソースを作成する必要があります。  リソースはJSONで構造化されています。  各リソースには複数のグループを含めることができ、そのうちの1つはモジュールの実行時に選択されます。

以下の例では、Broのintel.logファイル内のエントリとCSV形式のカスタムアプリケーションログに名前を付けています:

```
{
	"Version": 2,
	"Set": [
		{
			"Delim": "\t",
			"Name": "Intel",
			"Subs": [
				{
					"Name": "source",
					"Index": 0
				},
				{
					"Name": "desc",
					"Index": 1
				},
				{
					"Name": "url",
					"Index": 2
				}
			]
		},{
			"Name": "App",
			"Engine": "csv",
			"Subs": [
				{
					"Name": "user",
					"Index": 0
				},
				{
					"Name": "host",
					"Index": 1
				},
				{
					"Name": "GUID",
					"Index": 2
				}
			]
		}

	]
}
```

重要なコンポーネントに注意してください:

* `Version` このファイルがnamedfieldsモジュールのどのバージョン用かを指定します。  1のままにしてください。
* `Set` グループの配列を含みます
* このファイルのセットには、 "Intel"という名前の1つのグループが含まれています。  区切り文字はタブ文字（ "\ t"）として指定され、のリストSubsが提供されます。
* "Subs"メンバーは、このグループ内のサブフィールドを定義します。  インデックス0のフィールドは "source"、インデックス1は "desc"、インデックス2は "url"という名前です。
* "Engine"メンバーは、グループでどのエンジンを使うべきかを宣言します（fields、csvなど）。  


Broログ用のGravwellに配布された[namedfields.json](https://github.com/gravwell/resources/blob/master/bro/namedfields/namedfields.json)ファイルには多くのグループが含まれています。  他の例についてはそれを参照してください。


### Namedfieldsのリソース生成

Gravwellはnamedfieldsリソースの生成を手助けする簡単なゴランライブラリを提供しました。  このライブラリは、namedfieldsモジュールが使用できるリソースをプログラム的に生成するために使用できます。  このライブラリは、 "nfgen"ディレクトリ内のgithubのtools Gravwellリポジトリにあります。

単一のリソース内に2つのグループを生成するための名前付きフィールドの最も簡単な使用法は、次のようになります:

```
package main

import (
	"github.com/gravwell/tools/nfgen"
	"log"
)

func main() {
	//create a new named fields resource using the CSV engine that knows how to deal with 2
	//data types, one for login events and one for password failed events
	nf := nfgen.NewGen()
	g, err := nfgen.NewGroup("logins", "csv", ``)
	if err != nil {
		log.Fatal(err)
	}
	if err = g.AddSub(`username`, ``, 1); err != nil {
		log.Fatal(err)
	}
	if err = g.AddSub(`host`, ``, 2); err != nil {
		log.Fatal(err)
	}
	if err = g.AddSub(`srcip`, ``, 3); err != nil {
		log.Fatal(err)
	}
	if err = nf.AddGroup(g); err != nil {
		log.Fatal(err)
	}
	if g, err = nfgen.NewGroup("failedlogins", "csv", ``); err != nil {
		log.Fatal(err)
	}
	if err = g.AddSub(`srcip`, ``, 2); err != nil {
		log.Fatal(err)
	}
	if err = g.AddSub(`username`, ``, 3); err != nil {
		log.Fatal(err)
	}
	if err = g.AddSub(`password`, ``, 4); err != nil {
		log.Fatal(err)
	}
	if err = g.AddSub(`host`, ``, 5); err != nil {
		log.Fatal(err)
	}
	if err = nf.AddGroup(g); err != nil {
		log.Fatal(err)
	}
	if err = nf.Export("/tmp/lookups.json"); err != nil {
		log.Fatal(err)
	}
}
```

golang buildchainがインストールされている場合は、上記のジェネレータの構築と実行は簡単です:

```
go get -u github.com/gravwell/tools/nfgen
go build main.go -o test
./test
```
