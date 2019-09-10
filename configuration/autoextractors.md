# 自動抽出

Gravwellはタグごとの抽出定義を可能にし、自己記述的ではない非構造化データおよびデータフォーマットと対話することの複雑さを緩和します。非構造化データでは、目的のデータフィールドを抽出するために複雑な正規表現が必要になることが多く、作成に時間がかかり、エラーが発生する可能性があります。

自動抽出機能は、タグに適用することができ、指定されたタグ内のデータからフィールドを正しく抽出する方法を説明するための単なる定義です。"ax"モジュールは自動的に他のモジュールの適切な機能を呼び出します。自動抽出システムは、以下の抽出方法をサポートしています。

* [CSV](../search/csv/csv.md)
* [フィールド](../search/fields/fields.md)
* [正規表現](../search/regex/regex.md)
* [スライス](../search/slice/slice.md)

自動抽出機能の定義は、タグに基づいて正しい抽出を透過的に参照する [AX](../search/ax/ax.md) モジュールによって使用されます。

## 自動抽出機能の設定

自動抽出機能は、axファイルを作成して各Gravwellノードの「extractions」ディレクトリにインストールすることで定義されます。 使用されるファイル名は関係ありません。 抽出ディレクトリに配置されたファイルはすべて、「ax」ファイルとして解析されます。デフォルトでは、抽出ディレクトリは `/opt/gravwell/extractions`ですが、`gravwell.conf`ファイルの "Autoextract-Definition-Path"設定変数を設定/変更することで設定できます。 Gravwellサービスは、自動抽出「ax」ファイルを変更した後に再起動する必要があります。

自動抽出ファイルは、[＃]文字を使用したコメントを許可する[TOML V4](https://github.com/toml-lang/toml) 形式に従います。 各「ax」ファイルには、複数の自動抽出定義を含めることができ、抽出ディレクトリには複数のファイルを含めることができます。

<span style="color: red; ">注：タグごとに定義できる抽出は1つだけです。</span>

<span style="color: red; ">注：自動抽出機能は、エントリーの基礎データ全体を常に操作します。列挙値の抽出には使用できません（ "-e"引数は許可されていません）。</span>

各抽出機能には、ヘッダーと以下のパラメーターが含まれています。

* tag - 抽出に関連したタグ
* name - 抽出用の人間に優しい名前
* desc - 抽出を説明する人間に優しい文字列
* module - 抽出に使用される処理モジュール（regex、slice、csv、fieldsなど）
* args - extractonモジュールの振る舞いを変更するために使用されるモジュール固有の引数
* params - 抽出定義

以下はregexモジュールを使ってApache 2.0アクセスログからいくつかの基本的なデータを引き出すように設計されたサンプル自動抽出ファイルです。

```
#Simple extraction to pull ip, method, url, proto, and status from apache access logs
[[extraction]]
	tag="apache"
	name="apacheaccess"
	desc="Apache 2.0 access log extraction to pull requester items"
	module="regex"
	args="" #empty values can be completely ommited, the regex processor does not support args
	params='^(?P<ip>\d+\.\d+\.\d+\.\d+)[^\"]+\"(?P<method>\S+)\s(?P<url>\S+)\s(?P<proto>\S+)\"\s(?P<status>\d+)'
```

抽出パラメータの定義方法に関する重要な注意事項がいくつかあります。

1. 各抽出パラメータの値は、文字列として定義し、二重引用符または一重引用符で囲む必要があります。
2. 二重引用符で囲まれた文字列は文字列エスケープ規則に従います（regexを使うときは注意してください）。
  a。"\ b"は、文字 "\ b"ではなく、バックスペースコマンド（文字0x08）になります。
3. 一重引用符で囲まれた文字列は生であり、文字列エスケープ規則に従いません。
 a。'\ b'は文字通りバックスラッシュ文字で、その後に 'b'文字が続きます。バックスペースではありません。

文字列エスケープルールを無視する機能は、バックスラッシュを多用するため、 "regex"プロセッサにとって特に便利です。

新しい[extraction]ヘッダと新しい指定を設定するだけで、単一のファイルに複数の抽出を指定できます。これは、単一のファイルに2つの抽出がある例です。

```
#Simple extraction to pull ip, method, url, proto, and status from apache access logs
[[extraction]]
	tag="apache"
	name="apacheaccess"
	desc="Apache 2.0 access log extraction to pull requester items"
	module="regex"
	args="" #empty values can be completely ommited, the regex processor does not support args
	params='^(?P<ip>\d+\.\d+\.\d+\.\d+)[^\"]+\"(?P<method>\S+)\s(?P<url>\S+)\s(?P<proto>\S+)\"\s(?P<status>\d+)'

#Extraction to apply names to CSV data
[[extraction]]
	tag="locs"
	name="locationrecords"
	desc="AX extraction for CSV location data"
	module="csv"
	params="date, name, country, city, hash, a comma ,\"field with comma,\""
```

"locs"タグの2回目の抽出では、必須ではないパラメーター（ここではargsを指定しない）を省略し、バックスラッシュを使用してストリング内で二重引用符を使用できるようにしています。抽出には3つの必須パラメーターしかありません。

* モジュール
* パラーム
* タグ

## フィルタリング

AXモジュールは検索時に統合フィルタリングをサポートします。ただし、フィルタリングは、自動抽出定義には適用できません。

#### フィルタリング演算子

| オペレーター | 名 | 説明|
|----------|------|-------------|
| == | 等しい | フィールドは等しくなければなりません
| != | 等しくない | フィールドは等しくてはいけません
| ~  | サブセット | フィールドに値が含まれています
| !~ | サブセットではない | フィールドに値が含まれていません

#### フィルタリング例

```
ax foo=="bar" baz~"stuff"
ax foo != bar baz !~ "stuff and things"
```

## プロセッサの例

いくつかの自動抽出定義を示し、自動抽出機能の有無にかかわらず同じタスクを実行するクエリを比較対照します。また、AX内でフィルタを使用する方法も示します。

### CSV

CSVまたは「カンマ区切り値」は、比較的効率的なテキスト転送および保存システムです。ただし、CSVデータは自己記述型ではないため、CSVデータをすべてまとめただけでは、実際にどの列になっているのかを判断するのは困難になります。自動抽出機能を使用して列名を事前定義し、CSVデータでの作業を劇的に簡単にすることができます。

CSVを使用してエンコードされたデータエントリの例を次に示します。

```
2019-02-07T10:52:49.741926-07:00,fuschia,275,68d04d32-6ea1-453f-886b-fe87d3d0e0fe,174.74.125.81,58579,191.17.155.8,1406,"It is no doubt an optimistic enterprise. But it is good for awhile to be free from the carping note that must needs be audible when we discuss our present imperfections, to release ourselves from practical difficulties and the tangle of ways and means. It is good to stop by the track for a space, put aside the knapsack, wipe the brows, and talk a little of the upper slopes of the mountain we think we are climbing, would but the trees let us see it Benjamin", "TL",Bury,396632323a643862633a653733343a643166383a643762333a373032353a653839633a62333361
```

そこにはたくさんのデータがあり、どのフィールドが何であるかを示すものはありません。さらに悪いことに、CSVデータにはコンマとその周囲のスペースが含まれているため、裸眼による列の識別が非常に困難になります。自動抽出機能を使用すると、列名と型を一度識別してから、 "ax"モジュールを使用してそれらを透過的に活用できます。

各要素を手動で抽出して名前を付けると、クエリは次のようになります。

```
tag=csvdata csv [0] as ts [1] as name [2] as id [3] as guid [4] as src [5] as srcport [6] as dst [7] as dstport [8] as data [9] as country [10] as city [11] as hash | table
```

次のような自動抽出設定の宣言があります。

```
[[extraction]]
	name="testcsv"
	desc="CSV auto-extraction for the super ugly CSV data"
	module="csv"
	tag="csvdata"
	params="ts, name, id, guid, src, srcport, dst, dstport, data, country, city, hash"
```

同じクエリは次のようになります。

```
tag=csvdata ax | table
```

<span style="color: red; ">注：CSV自動抽出プロセッサーは引数をサポートしていません</span>

<span style="color: red; ">注：params変数内の名前の位置はフィールド名を示します。CSVヘッダとして扱う</span>

### フィールド
fieldsモジュールは非常に柔軟な処理モジュールで、データを抽出するために任意の区切り文字とフィールドルールを定義することができます。Bro / Zeekのような一般的なセキュリティアプリケーションの多くは、データのエクスポートにTSV（タブ区切り値）をデフォルトとしています。他のカスタムアプリケーションは "|"のような変なセパレータを使うかもしれません または "//"のような一連のバイト。フィールドエクストラクタを使えば、それをすべて扱うことができます。自動エクストラクタと組み合わせると、データフォーマットの詳細について心配する必要はありません。

他の自動抽出プロセッサとは異なり、fieldsモジュールにはさまざまな設定引数があります。引数のリストは[fieldsモジュールのドキュメント](/#!search/fields/fields.md)に完全に文書化されています。"-e"フラグのみがサポートされていません。

タブ区切りのデータから始めましょう。

```
2019-02-07T11:27:14.308769-07:00	green	21.41.53.11	1212	57.27.200.146	40348	Have I come to Utopia to hear this sort of thing?
```

fieldsモジュールを使用して各データ項目を抽出するためには、次のようになります。
```
tag=tabfields fields -d "\t" [0] as ts [1] as app [2] as src [3] as srcport [4] as dst [5] as dstport [6] as data | table
```

同じことを実行するための自動抽出構成は次のとおりです。

```
[[extraction]]
	tag="tagfields"
	name="tabfields"
	desc="Tab delimited fields"
	module="fields"
	args='-d "\t"'
	params="ts, app, src, srcport, dst, dstport, data"
```
axモジュールと上記の設定を使用すると、クエリは次のようになります。

```
tag=tagfields ax | table
```

少し変な区切り文字 "|"を使っていくつかのデータを見てみましょう：

```
2019-02-07T11:57:24.230578-07:00|brave|164.5.0.239|1212|179.15.183.3|40348|"In C the OR operator is ||."
```

最後のフィールドには区切り文字が含まれています。このデータを生成したシステムは、データ項目に区切り文字を含める必要があることを知っていたため、そのデータ項目を二重引用符で囲みました。fieldsモジュールはクォートされたデータをどのように扱うかを知っています。"-q"フラグを指定すると、モジュールは引用符付きフィールドを尊重します。"-clean"フラグも指定されていない限り、引用符は抽出されたデータに保持されます。

fieldsモジュールを使うと、クエリは次のようになります。
```
tag=barfields fields -d "|" -q -clean [0] as ts [1] as app [2] as src [3] as srcport [4] as dst [5] as dstport [6] as data 
```

しかし、適切な自動抽出設定（下記参照）があれば、クエリは非常に単純なものになる可能性がありますtag=barfields ax | table。

```
[[extraction]]
	tag="barfields"
	name="barfields"
	desc="bar | delimited fields with quotes and cleaning"
	module="fields"
	args='-d "|" -q -clean'
	params="ts, app, src, srcport, dst, dstport, data"
```

結果は引用符が削除されて正しくクリーニングされます。

![Fields Results](fieldsax.png)

### 正規表現(regex)

正規表現は、自動抽出プログラムの最も一般的な用途です。正規表現は、理解するのが難しく、タイプミスがしやすく、そして最適化が困難です。正規表現の第一人者がいる場合は、あらゆる方法で効率的かつ柔軟な抽出を実行する、非常に高速な正規表現を作成するのに役立ちます。その後、単純に自動抽出に展開して、それをすべて忘れることができます。

これは率直に言って混乱しているエントリセットの例です（これはカスタムアプリケーションログでは珍しいことではありません）。

```
2019-02-06T16:57:52.826388-07:00 [fir] <6f21dc22-9fd6-41ee-ae72-a4a6ea8df767> 783b:926c:f019:5de1:b4e0:9b1a:c777:7bea 4462 c34c:2e88:e508:55bf:553b:daa8:59b9:2715 557 /White/Alexander/Abigail/leg.en-BZ Mozilla/5.0 (Linux; Android 8.0.0; Pixel XL Build/OPR6.170623.012) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.107 Mobile Safari/537.36 {natalieanderson001@test.org}
```

データはいくつかのカスタムアプリケーションにとって本当に醜いアクセスログです。次のようにいくつかのフィールドにアクセスしようとしています。

* ts - 各エントリの先頭のタイムスタンプ
* app - 処理アプリケーションを表す文字列
* uuid - 一意の識別子
* src - 送信元アドレス、IPv4とIPv6の両方
* srcport - 送信元ポート
* dst -  宛先アドレス、IPv4とIPv6の両方
* dstport - 宛先ポート
* path - パスのようなURL
* useragent - useragent
* email - リクエストに関連付けられているメールアドレス

これが私たちの抽出器定義の例です：

```
[[extraction]]
	module="regex"
	tag="test"
	params='(?P<ts>\S+)\s\[(?P<app>\S+)\]\s<(?P<uuid>\S+)>\s(?P<src>\S+)\s(?P<srcport>\d+)\s(?P<dst>\S+)\s(?P<dstport>\d+)\s(?P<path>\S+)\s(?P<useragent>.+)\s\{(?P<email>\S+)\}$'
```

すべてのデータ項目を1つ抽出してそれらを表に入れたいとしましょう。

正規表現を使用すると、クエリは次のようになります。

```
tag=test regex "(?P<ts>\S+)\s\[(?P<app>\S+)\]\s<(?P<uuid>\S+)>\s(?P<src>\S+)\s(?P<srcport>\d+)\s(?P<dst>\S+)\s(?P<dstport>\d+)\s(?P<path>\S+)\s(?P<useragent>.+)\s\{(?P<email>\S+)\}$" | table
```

ただし、自動抽出機能とaxモジュールを使用すると、次のようになります。

```
tag=test ax | table
```

結果は同じです。

![Regex Results](regexax.png)

axモジュールを使ってフィールドをフィルタリングしたい場合は、axモジュールの名前付きフィールドに単純にfilterディレクティブを付けることができます。この例では、抽出されたすべてのフィールドを含むテーブルをレンダリングしながら、電子メールアドレスに "test.org"を含むすべてのエントリを表示します。

```
tag=test ax email~"test.org" | table
```

特定のフィールドのみが必要な場合は、デフォルトですべてのフィールドを抽出するのではなく、axモジュールに特定のフィールドのみを抽出するように指示するフィールドを指定できます。

```
tag=test ax email~"test.org" app path | table
```

### スライス(Slice)
[スライスモジュール](/search/slice/slice.md) は、バイナリデータストリームから直接データを抽出することができる強力なバイナリ・スライス方式です。Gravwellのエンジニアはスライスモジュール以外の何も使用しないでプロトコルディセクタ全体を開発しました。しかし、データのバイナリストリームを切り取ってデータを解釈することは、心の弱い人には向いていません。独自のデータストリームをスライスしてさいの目に切るような美しいクエリを作成したら、覚えたりコピーしたりすることはできません。

テキスト形式でバイナリデータを表示するのは難しいので、このドキュメントではデータを16進エンコーディングで表示します。我々は、醸造システム内の正確な温度制御を維持するために冷媒圧縮機を調整する小さな制御システムから来るバイナリデータストリームを切り出す予定です。制御システムは、文字列、整数、およびいくつかの浮動小数点値を出荷します。制御システムの場合と同様に、すべてのデータは[ビッグエンディアン](https://en.wikipedia.org/wiki/Endianness)順になっています。

注：スライスAXプロセッサは引数をサポートしていません（例： "-e"は使用不可）

#### フィルタリング

スライスAXプロセッサは、データを特定のタイプにキャストするように設計されています。そのため、そのフィルタリングオプションは他のモジュールよりも少し微妙です。抽出された各値には、そのタイプに基づいて特定のフィルター演算子のセットがあります。フィルタ演算子と型の詳細については、[スライスモジュールのドキュメント](../search/slice/slice.md)を参照してください。

#### 例

まずは、[hexlify](#!search/hexlify/hexlify.md)モジュールを使って、16進数のデータを見てみましょう。

```
tag=keg hexlify
```

これは、次のようなエントリになります。

```
12000000000ed3ee7d4300000000014de536401800004b65672031
```

ちょっとしたことで、パックされたバイナリ構造体が次のものを含んでいることを確認することができました。

| ID | タイムスタンプ秒| タイムスタンプナノ秒 | 32ビットフロートとしての温度 | ASCII name |
|----|-------------------|-----------------------|----------------------------|------------|
|bits 0:2 | bits 2:10              | bits 10:18                 | bits 18:22                      | bits 22:        |

各データ項目を抽出するために次のスライスクエリを生成するために使用できました。

```
tag=keg slice uint16be([0:2]) as id int64be([2:10]) as sec uint64be([10:18]) as nsec float32be([18:22]) as temp [22:] as name | table
```

![Slice Table](sliceres.png)

手動クエリから、次の自動抽出設定を生成できます。

```
[[extraction]]
	tag="keg"
	name="kegdata"
	desc="binary temperature control extractions"
	module="slice"
	params="uint16be([0:2]) as id int64be([2:10]) as sec uint64be([10:18]) as nsec float32be([18:22]) as temp [22:] as name"
```

複雑なスライスクエリは次のようになります。

```
tag=keg ax | table
```

フィルタリングといくつかの数学モジュールを使用して、さらに一歩進めて、各プローブの最高温度を示すクールグラフを生成できます。

```
tag=keg ax id==0x1200 temp name | max temp by name | chart max by name
```

![Probe Temperature](temps.png)

追加のフィルタリングを使用して樽温度のみを選択し、温度変動を調べて制御システムが一定の温度をどの程度維持しているかを確認できます。

```
tag=keg ax id==0x1200 temp name~Keg | stddev temp by name | chart stddev by name
```

![Probe Stddev](tempstddev.png)

自動抽出機能といくつかの基本的な数学を使用して、バイナリデータを分析し、コンプレッサの周期的な関与を明確に確認できます。これにより、時間の経過とともに温度が変動します。私たちが醸造所を経営しているのなら、私たちは醸造所と呼んで温度許容度を厳しくするために制御システムロジックを微調整することを提案するかもしれません。
