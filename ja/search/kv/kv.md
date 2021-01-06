# KV

kvモジュールは、検索エントリからキー値フォーマットされたデータを抽出してフィルタリングし、さらに分析を行うために列挙された値に変換するために使用されます。キー値データは、`a = 1` や `Name`例えば、`a = 1` や `Name: Fred` などである。

## キー値データは、`a = 1` や `Name: Fred` のような1つ以上のキー値ペアで構成される。

kvモジュールは、キー値データ形式を説明するためにいくつかの特定の用語を使用しています。それらは以下の通りです。

* Separator : キーと値を区切る文字 (複数可)。
* デリミタ : 区切り文字：キーと値のペアを互いに分離する文字。キーとセパレータ、あるいはセパレータとデリミタの間に存在する場合もあります。

例えば、以下のエントリは4つのキーと値のペアを持ち、'='を区切り文字、スペースを区切り文字として使用しています。

```
x=1 y=2 z=3 foo=bar
```

キーと値のペアには、キー、セパレータ、値の間の区切り文字も含まれることに注意してください。以下は前のエントリと同じです。

```
x=1 y=2 z=3 foo=bar
```

このエントリには同じキーと値のペアが含まれていますが、':'を区切り文字として、'|'を区切り文字として使用しています。

```
x:1|y:2|z:3|foo:bar
```

## サポートされているオプション

* `-e <arg>` : e" オプションは、レコード全体ではなく、列挙された値に対して操作する。
* `-sep <separator>`: "-sep" フラグは、ユーザがセパレータを指定できるようにする。sep" フラグは、ユーザがセパレータを指定できるようにする。これには1文字以上の文字を指定することができ、例えば `-sep EQUALS` のように指定することができる。
* `-d <区切り文字>`. d" フラグは使用する区切り文字を指定する。複数のデリミタを指定することができる。例えば、ダブルクォート、タブ、スペースをデリミタとして使用したい場合は、`-d "\t"`を指定する。
* `-s`: s "オプションは、モジュールをstrictモードにする。厳密モードでは、指定されたすべての抽出が成功しない限り、エントリは削除される。
* `-q`: q" オプションは、引用符で囲まれた値を有効にする。これにより、例えば `key="this is the value"` のように、値に区切り文字を含めることができる。

## キーの名前変更

抽出されたキーは、キーの直後に `as <name>` というディレクティブをつけることで名前を変更することができます。 例えば、キー "foo" を "bar" という名前の列挙値に抽出するには、 抽出ディレクティブは `foo as bar` となります。 renameディレクティブが指定されていない場合は、抽出された値にはキーにマッチする名前が与えられます。

## フィルタリング

kvモジュールは、文字列の等号性に基づいてフィルタリングを行うことができます。等しくない」、「等しくない」、「含まれる」、「含まれない」を指定するフィルタが有効になっている場合、抽出された1つ以上の値でフィルタの指定に失敗したエントリは完全に削除されます。 等しくない"!="として指定されたキーが存在しない場合、そのフィールドは抽出されませんが、そのエントリは完全に削除されません(`-s`が設定されていない限り)。

| 演算子 | 名称 | 意味 |
|----------|------|-------------|
| == | 等しい  | フィールドは等しい|
| != | 等しくない | フィールドは等しくない|
| ~  | 含む | フィールドはその値を含む|
| !~ | 含まない | フィールドはその値を含まない|

## 例

syslog generator](https://github.com/gravwell/generators)で生成されたエントリは以下のようになります。

```
<5>1 2019-09-27T09:33:01.362929-06:00 unicorn chatter 14747 - [sofiamartinez461@test.net source-address="1.192.254.26" source-port=47734 destination-address="15.176.86.127" destination-port=626 useragent="Mozilla/5.0 (Windows NT 6.2; WOW64; rv:39.0) Gecko/20100101 Firefox/39.0"] Have I come to Utopia to hear this sort of thing?
```

次のクエリはソースポートとソースアドレスのフィールドを抽出してテーブルに表示します。

```
tag=syslog kv -q "source-port" "source-address" | table
```
![](syslog1.png)

このクエリはソースアドレスに "1.192. "が含まれていないエントリを全て削除します。

```
tag=syslog kv -q "source-port" "source-address"~"1.192. | table
```

![](syslog2.png)

### 区切り文字を変更

separatorは任意の文字列に設定することができます; 例えば、キーと値のペアは次のようになります。

```
x EQUALS 1 y EQUALS 2
```

以下のようにして'y'の値を抽出することができます。

```
kv -sep EQUALS y
```