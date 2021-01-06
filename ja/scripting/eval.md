# evalモジュール

[検索モジュール](#!search/searchmodules.md#Eval)で紹介されているように、Gravwellのevalモジュールは、他のモジュールが足りない場合に検索エントリを操作するための一般的なツールです。 [Ankoスクリプト言語](scripting.md) を使用して、パイプライン内で一般的なスクリプト機能を提供します。

evalモジュールには、いくつかの重要な制限があります。

* 単一のステートメントのみを定義できます：`(x==y || j < 2)`は、`if SrcPort==80 { setEnum("http", true) }`のように許容されますが、 `setEnum(foo, "1"); setEnum(bar, "2")`は2つのステートメントであり、機能しません。詳細については、次のセクションを参照してください。
* 関数の定義またはインポートはできません。
* ループは許可されていません。
* リソースシステムへのアクセスはできません。

注：eval式の構造をより明確にするには、必要に応じて検索文の入力中にCtrl-Enterを押して改行を挿入します。

言語自体の詳細については、[Ankoスクリプト言語](scripting.md) で使用されているスクリプト言語の一般的な説明を参照してください。

## フィルタリング：式とステートメント

引数が式の場合、evalモジュールはエントリをフィルタリングします。 引数がステートメントの場合はフィルターされません。 次の例を考えてみましょう。

```
tag=reddit json Body | eval len(Body) < 20 | table Body
```

`len(Body) < 20`は式であるため、式に一致するエントリのみがパイプラインを続行できます。 対照的に、次のことを考慮してください。

```
tag=reddit json Body | eval if len(Body) <= 10 { setEnum("postlen", "short"); setEnum(“anotherEnum”, “foo”) } | table Body
```

`if`形式はステートメントです。 ifステートメントの結果に関係なく、すべてのエントリはパイプラインを継続します。

式は、文字列`"foo"`や数値`1.5`などの値に評価されるものです。 これには、`DoStuff(15)`などの関数呼び出しや、`myVariable == 42`などのブール式も含まれます。 基本的に、変数に割り当てたり、関数の引数として渡すことができるものはすべて、式と見なすことができます。

ステートメントは、`if`および`switch`ステートメント、変数の作成/割り当て、戻り値など、スクリプトのフローと構造を制御します。ステートメント自体には複数のサブステートメントが含まれる場合があります。 たとえば、 `if`ステートメントには式とステートメントのいくつかのリストが含まれます。 式がtrueと評価されると、ステートメントのリストが1つ実行されます。 式がfalseと評価されると、別のリストが実行されます。

## 列挙値

evalステートメント内では、既存の列挙値は、通常の変数であるかのように参照できます。

```
tag=reddit json Body | eval len(Body) < 20 | table Body
```

ただし、列挙値を設定するには、より明示的なステートメントが必要です。 `setEnum`関数は引数として名前と値を取ります。 指定された名前の列挙値を作成または更新し、指定された値の正しい列挙値タイプを推測します。

```
tag=reddit json Body | eval if len(Body) <= 10 { setEnum("postlen", "short") } else if len(Body) > 10 && len(Body) < 300 { setEnum("postlen", "medium") } else { setEnum("postlen", "long") } | count by postlen | table postlen count
```

`delEnum`関数は必要に応じて指定された列挙値を削除しますが、これはほとんど必要ありません。

## ユーティリティ関数

Evalは、`functionName(<functionArgs>) <returnValues>`の形式で以下にリストされている組み込みユーティリティ関数を提供します。

* `setEnum(key, value)` は、valueという値を持つkeyという名前の列挙値を作成します。これには、任意の有効な列挙値タイプを指定できます。
* `delEnum(key)` は、keyという名前の列挙値を削除します。
* `hasEnum(key) bool` は、エントリに列挙値があるかどうかを示すブール値を返します。
* `setPersistentMap(mapname, key, value)` は、検索全体にわたって持続するキーと値のペアをマップに保存します。
* `getPersistentMap(mapname, key) value` は、指定された永続マップから、指定されたキーに関連付けられた値を返します。
* `len(val) int` はvalの長さを返します。valには文字列、スライスなどを指定できます。
* `toIP(string) IP` は、stringをIPに変換します。パケットモジュールなどで生成されたIPとの比較に適しています。
* `toMAC(string) MAC` はstringをMACアドレスに変換します。
* `toString(val) string` は、valを文字列に変換します。
* `toInt(val) int64` は、可能であればvalを整数に変換します。変換できない場合は0を返します。
* `toFloat(val) float64` は、可能であればvalを浮動小数点数に変換します。変換できない場合は0.0を返します。
* `toBool(val) bool` は、valをブール値に変換します。変換できない場合はfalseを返します。ゼロ以外の数字と文字列「y」、「yes」、「true」はtrueを返します。
* `toHumanSize(val) string` はvalを整数に変換し、可読なバイト数にして表示します。例えば、`toHumanSize(15127)` は "14.77 KB" に変換されます。
* `toHumanCount(val) string` はvalを整数に変換し、読みやすい数値として表現します。例えば、`toHumanCount(15127)`は "15.13 K "に変換されます。
* `typeOf(val) type` はvalのタイプを「string」や「bool」といった文字列として返します。。

変換関数は特に重要です。なぜなら、evalは常に必要な方法で暗黙的な変換を実行できるとは限らないからです。 たとえば、パケットモジュールは、文字列だけでなく、特殊なタイプのIPを抽出します。 IPを含む列挙値と適切に比較するには、 `IP`関数を使用する必要があります。

```
tag=pcap packet ipv4.SrcIP | eval SrcIP != toIP("192.168.0.1") | count by SrcIP | table SrcIP count
```

予期しない結果が発生する場合、 `typeOf`関数を使用して列挙値の型を確認できます。

```
tag=pcap packet ipv4.SrcIP | eval setEnum("type", typeOf(SrcIP)) | table type
```

この場合、結果は`SrcIP`がnet.IPであることを示しています。

#### 永続的なマップ

Evalは、マップにデータを保存する方法を提供します。つまり、あるエントリのデータを保存して、後で別のエントリと比較することができます。 `SetPersistentMap(mapname, key, value)` 関数は、指定された文字列 `key`を`value`にマッピングする（`mapname`パラメーターで指定された）エントリを作成または更新します。 その後、 `GetPersistentMap(mapname, key)`関数を使用して、その情報を取得できます。

永続的なマップ機能の例を示すために、次のjson形式のHacker Newsコメント（実際のコメントではない）を検討してください。

```
{"body": "The eval function has persistent map capabilities", "author": "Gravwell", "article-id": 1234000, "parent-id": 1234111, "article-title": "Gravwell for data analysis", "date-string": "0 minutes ago", "type": "comment", "id": 1234222}
```

コメントには、現在のコメントのID（1234222）、作成者の名前（「Gravwell」）、および親コメントのID（12341111）が含まれますが、親コメントの作成者の名前は含まれません。

次のコマンドは、親IDを作成者名に一致させようとします。 表示されるすべてのコメントについて、コメントIDと著者名のマッピングを「id_to_name」という名前のマップに保存します。 次に、親IDのマップにエントリがあるかどうかを確認します。 その場合、`parentauthor`列挙値をその名前に設定し、そうでない場合は列挙値を「不明」に設定します。

```
tag=hackernews json author id "parent-id" as parentid | sort asc | eval if true { setPersistentMap("id_to_name", id, author);  name = getPersistentMap("id_to_name", parentid); if name != nil { setEnum("parentauthor", name) } else { setEnum("parentauthor", "unknown") } } | table author id parentid parentauthor
```

このアプローチの制限の1つは、最初に処理された最も古いコメントが、検索に表示されないさらに古いコメントに返信することです。 したがって、最初の結果にはすべて「不明」の親作成者が含まれ、結果をスクロールすると、有効な親作成者を見つけるコメントが増えます。 より良い解決策は、おそらく1週間にわたって検索を実行し、各コメントの著者名とコメントIDを引き出して、結果をルックアップテーブルに保存することです。 次に、そのルックアップテーブルは、より短い期間の検索で`lookup`モジュールで使用するためのリソースとして保存できます。

この例では、現在の状態のevalモジュールの別の制限も示しています。1つのステートメントまたは式のみを実行するため、`if true {}`ステートメント内にロジックをラップする必要があります。
