# anko

[検索モジュールのドキュメント](#!search/searchmodules.md#Anko)で紹介されているように、Gravwellのankoモジュールは、検索パイプライン内の汎用スクリプトツールです。 スクリプト作成者の複雑さを犠牲にして、検索エントリの非常に柔軟な操作を可能にします。 ただし、スクリプトを作成したら、他のユーザーと簡単に共有できます。

言語自体の詳細については、[Ankoスクリプト言語のドキュメント](scripting.md) で使用されているスクリプト言語の一般的な説明を参照してください。

## ankoスクリプトの管理

検索でankoスクリプトを実行するには、スクリプトを含むテキストファイルをリソースとしてアップロードする必要があります。 リソースを作成およびアップロードする方法については、[リソースセクション](#!resources/resources.md)を参照してください。

この時点で、スクリプトを変更するには、元のテキストファイルのスクリプトを編集してから、ファイルをリソースに再アップロードする必要があります。 Gravwellの将来のバージョンには、スクリプト作成を簡単にする統合テキストエディタが含まれます。

## ankoスクリプトを実行する

スクリプトを実行するには、単に「anko」を指定し、その後にスクリプト名、スクリプト用の引数を指定します。 たとえば、引数として2つの数値を受け取る「foo」という名前のスクリプトを実行するには、パイプラインで「anko foo 1 3」と入力します。

## ankoスクリプトの作成

Ankoスクリプトは、evalコマンドと同じ構文を使用します。 以下の例とevalモジュールのセクションを参照してください。

### 必須機能の定義

ankoスクリプトには、「Process」という名前の関数または「Main」という名前の関数のいずれかを含める必要があります。 これらの関数は引数を取りません。 これら2つの関数は、検索エントリを処理するための2つの異なるオプションを表します。

`Process`関数が定義されている場合、検索エントリごとに1回呼び出されます。 エントリの列挙値はローカル変数として扱われ、 `Process`関数の戻り値は、エントリがパイプラインを続行できるか（true）、否か（false）を決定します。

`Main`関数が定義されている場合、一度だけ呼び出されます。 そのため、プログラマは、「readEntry」および「writeEntry」関数を呼び出して、各検索エントリを取得し、操作後に行に渡す必要があります。

概念的にははるかに単純なので、可能な限り「Main」ではなく「Process」でスクリプトを書くことを強くお勧めします。

スクリプトには、「Parse」および「Finalize」という名前の関数も含まれる場合があります。

「Parse」関数は「Process」または「Main」の前に呼び出され、引数の配列としてコマンドライン引数が与えられます。 `Parse`関数は、nilを返すことで引数が正常に処理されたことを示します。 ゼロ以外の戻り値はエラーとして扱われ、ユーザーに表示されます。 スクリプトの引数を解析する方法のサンプルについては、以下のサンプルスクリプトを参照してください。

「Finalize」関数は、「Process」または「Main」が完了した後に呼び出されます。 スクリプトで最後に実行されたコードです。 これは、必要に応じてリソースを作成するのに適した場所です。

### Process（）関数を使用したサンプルスクリプト

このサンプルスクリプトは、スクリプトの引数として指定された2つのモードで動作します。 「ビルド」モードでは、パケットモジュールから抽出された「SrcIP」フィールドを使用して、現在の検索で見つかったIPアドレスのリストを作成し、そのリストをリソースとして保存します。 「適用」モードでは、以前に作成されたテーブルを取得し、以前に表示された「SrcIP」フィールドを含むすべてのエントリを削除します。 このスクリプトは、ネットワーク上の新しいデバイスを探すために使用されていましたが、「lookup」モジュールは同じ機能をより柔軟に提供するようになりました。

```
table = make(map[string]interface)
task = "build"

var json = import("encoding/json")

# first arg = "build" or "apply"
func Parse(args) {
	errstr = "argument must be 'build' or 'apply'"
	if len(args) == 1 {
		task = args[0]
	} else {
		return errstr
	}
	switch task {
	case "apply":
		# load the table
		data, _ = getResource("lookuptable")
		json.Unmarshal(data, &table)
	case "build":
	default:
		return errstr
	}
	return nil
}

func Process() {
	if task == "build" {
		s = toString(SrcIP)
		table[s] = true
		return true
	} else if task == "apply" {
		s = toString(SrcIP)
		# create & set an enumerated value named "new" to true or false
		setEnum("new", !table[s])
		# if it's not in the table, return true
		return !table[s]
	}
}

func Finalize() {
	if task == "build" {
		data, err = json.Marshal(table)
		if err != nil {
			return err
		}
		return setResource("lookuptable", data)
	}
}
```

「SrcIP」列挙値は、「Process」関数内の他の変数と同じように**読み取り**される可能性がありますが、列挙値を設定するには、アクションを明示するために `setEnum`関数を使用する必要があります。

`table`と` task`変数は関数定義の外で宣言されていることに注意してください。 組み込みのjsonエンコーディングライブラリも、関数定義の前にインポートされます。 利用可能なライブラリの完全なリストを以下に示します。


### Main（）関数を使用したサンプルスクリプト

`Main`関数を使用してスクリプトを記述することはより困難ですが、エントリを複製する必要がある場合は必要です。 次のスクリプトは、Modbusメッセージを含むエントリを読み取ります。 メッセージがタイプ0x10（「複数のレジスタを書き込む」）の要求である場合、スクリプトは書き込まれるレジスタごとに元のエントリを1回複製し、各エントリに単一のレジスタアドレスを含む「RegAddr」および「RegValue」列挙値を付加します +レジスタ値。

注：このスクリプトは単独では機能しません。 書かれているように、パイプラインの早い段階で別のankoスクリプトの出力を消費することを意図しており、「Request」や「WriteAddr」などの列挙値を入力します。

```
func Main() {
	for i = 0; i != -1; i++ {
		ent, err = readEntry()
		if err != nil {
			return
		}

		# Check if this is a request or a response
		Request, err = getEntryEnum(ent, "Request")
		if err != nil {
			#Request isn't set, this isn't a modbus packet, skip
			continue
		}

		# read the function value
		Function, err = getEntryEnum(ent, "Function")
		if err != nil {
			continue
		}

		ReqResp, err = getEntryEnum(ent, "ReqResp")
		if err != nil {
			continue
		}

		if Request == true {
			if Function == 0x10 {
				# write multiple registers
				Addr, err = getEntryEnum(ent, "WriteAddr")
				if err != nil {
					writeEntry(ent)
					continue
				}
				Count, err = getEntryEnum(ent, "WriteCount")
				if err != nil {
					writeEntry(ent)
					continue
				}
				if Count == 0 || len(ReqResp) < 5 + 2*Count {
					writeEntry(ent)
					continue
				}
				for j = 0; j < Count; j++ {
					newEnt = cloneEntry(ent)
					# read the register value
					val = (toInt(ReqResp[5+(2*j)]) << 8) | toInt(ReqResp[5+(2*j)+1])
					setEntryEnum(newEnt, "RegAddr", Addr + j)
					setEntryEnum(newEnt, "RegValue", val)
					err = writeEntry(newEnt)
					if err != nil {
						continue
					}
				}
			} else {
				writeEntry(ent)
			}
		}
	}
}
```

「readEntry」、「cloneEntry」、および「writeEntry」関数の使用に注意してください。 この検索エントリの明示的な管理は、「プロセス」関数を使用するスクリプトでは必要ありません。 また、列挙値を変数として扱うのではなく、列挙値を読み取るための「getEntryEnum」および「setEntryEnum」関数の使用にも注意してください。

## ユーティリティ関数

Ankoは、 `functionName(<functionArgs>) <returnValues>`の形式で以下にリストされている組み込みユーティリティ関数を提供します。 次の関数は、ankoスクリプトで使用できます。

* `getResource(name) []byte, error`は、指定されたリソースのコンテンツであるバイトのスライスを返しますが、エラーはリソースの取得中に発生したエラーです。
* `setResource(name, value) error`は（必要な場合）`name`という名前のリソースを作成し、 `value`の内容で更新し、エラーが発生した場合にエラーを返します。
* `len(val) int`はvalの長さを返します。valには文字列、スライスなどを指定できます。
* `toIP(string) IP`は、stringをIPに変換します。パケットモジュール。
* `toMAC(string) MAC`はstringをMACアドレスに変換します。
* `toString(val) string`はvalを文字列に変換します。
* `toInt(val) int64`は、可能であればvalを整数に変換します。変換できない場合は0を返します。
* `toFloat(val) float64`は、可能であればvalを浮動小数点数に変換します。変換できない場合は0.0を返します。
* `toBool(val) bool`はvalをブール値に変換しようとします。変換できない場合はfalseを返します。ゼロ以外の数字と文字列「y」、「yes」、「true」はtrueを返します。
* `typeOf(val) type`はvalのタイプを文字列として返します。 「string」、「bool」。

次の関数は、「Process」関数を実装するスクリプトでのみ使用できます。

* `setEnum（key、value）error`は、`value`を含む `key`という名前の現在のエントリに列挙値を作成します。
* `getEnum（key）value、error`は、`key`で指定された列挙値を返します
* `delEnum（key）`は、現在のエントリから `key`という名前の列挙値を削除します。
* `hasEnum（key）bool`は、現在のエントリに列挙値があるかどうかを返します。

次の関数は、 `Main`関数を実装するスクリプトでのみ利用可能です：

* `readEntry（）entry、error`は次のエントリとエラー（存在する場合）を返します。エントリが残っていない場合、エラーを返します。
* `writeEntry（ent）error`は与えられたエントリをパイプラインに書き出し、エラーがあればそれを返します。
* `cloneEntry（ent）entry`は、指定されたエントリのコピーを返します。
* `setEntryEnum（ent、key、value）`は、指定されたエントリに列挙値を設定します。
* `getEntryEnum（ent、key）value、error`は、指定されたエントリから列挙値を読み取ります。
* `hasEntryEnum（ent、key）bool`は、エントリに列挙値が含まれているかどうかを返します。
* `delEntryEnum（ent、key）`は、指定された列挙値を特定のエントリから削除します。
* `setEntryData（ent、value）`はエントリのデータ部分を設定します。

注： `Process`関数は特定のエントリで暗黙的に動作するため、` Process`関数と `Main`関数を使用するスクリプトでは、` setEnum`、 `hasEnum`、および` delEnum`関数は異なります。

## 利用可能なパッケージ

特定のGoライブラリのankoラッパーは、次のような構文でインポートできます。

```
var json = import("encoding/json")
```

セキュリティ上の理由から、ankoモジュールは完全なAnkoスクリプト言語に含まれる*すべて*パッケージへのアクセスを許可しません。 次のパッケージは、ankoスクリプトで使用できます。

* `bytes`：バイトスライスを使用
* `crypto / md5`、` crypto / sha1`、 `crypto / sha256`、` crypto / sha512`：暗号ハッシュ
* `encoding / csv`：CSVデータのエンコードとデコード
* `encoding / json`：JSONデータのエンコードとデコード
* `errors`：Goエラーを処理する
* `flag`：コマンドライン引数を処理します
* `fmt`：文字列の印刷とフォーマット
* `math`：数学関数
* `math / big`：bignums
* `math / rand`：乱数
* `net / http`：制限されたHTTP機能（クライアントのみ）
* `net / url`：URL
* `path`：パス
* `path / filepath`：ファイル固有のパス関数
* `regexp`：正規表現
* `sort`：ソート
* `strings`：文字列処理関数
* `time`：時間処理関数
* `github.com/ziutek/telnet`：telnetクライアント関数（Dial、DialTimeout、およびNewConn関数が使用可能です。[https://godoc.org/github.com/ziutek/telnet](https://godoc.org/github.com/ziutek/telnet) を参照してください。

このドキュメントでは、すべてのパッケージを網羅的に説明することはできません。 [公式ankoリポジトリ](https://github.com/mattn/anko/tree/master/packages)で各パッケージにエクスポートされた使用可能な関数を表示できます。 いくつかの特定のパッケージは、公式のankoリポジトリによってエクスポートされる完全な機能を提供しないため、以下でさらに説明されます。

### `net / http`パッケージ

`net/http`はHTTP *リクエスト*を実行するための関数、型、変数のサブセットをエクスポートします。 タイプ「Client」、「Cookie」、「Request」、および「Response」がエクスポートされます。 これらのタイプの説明については、[Goドキュメント]（https://golang.org/pkg/net/http/）を参照してください。

関数 `NewRequest（operation、url、body）（Request、error）`は、指定されたURL文字列で指定された操作（ "PUT"、 "POST"、 "GET"、 "DELETE"）で新しいhttp.Requestを準備します。 本文は、PUTおよびPOST要求で使用するオプションのパラメーターです。 「nil」またはio.Readerに設定する必要があります。

ほとんどの場合、NewRequest関数を使用してリクエストを作成し、http.DefaultClientを使用してそのリクエストを実行します。

```
req, _ = http.NewRequest("GET", "http://example.org/foo", nil)
http.DefaultClient.Do(req)
```

追加のヘッダーまたはCookieは、送信前にリクエストで設定できます。

```
req, _ = http.NewRequest("GET", "http://example.org/foo", nil)
# Add a header
req.Header.Add("My-Header", "gravwell")
# Add a cookie
cookie = make(http.Cookie)
cookie.Name = "foo"
cookie.Value = "bar"
req.AddCookie(&cookie)
http.DefaultClient.Do(req)
```
