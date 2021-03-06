# ankoモジュール

[検索モジュールドキュメント](#!search/searchmodules.md#Anko)で紹介されているように、Gravwellのankoモジュールは検索パイプライン内の汎用スクリプトツールです。スクリプト作成者にとっては複雑さを犠牲にしても、検索エントリの非常に柔軟な操作を可能にします。しかし、一度スクリプトが作成されると、他のユーザーと簡単に共有することができます。

言語自体の詳細については、[ankoスクリプト言語ドキュメント](scripting.md)で使用されているスクリプト言語の一般的な説明を参照してください。

### ankoスクリプトでネットワーク機能を無効にする

デフォルトでは、ankoスクリプトは http や net ライブラリ、sftp、sshなどのネットワークユーティリティの使用を許可されています。Gravwellユーザーにネットワークアクセスを与えたくないかもしれません。

## ankoスクリプトの管理

検索でankoスクリプトを実行するには、スクリプトを含むテキストファイルをリソースとしてアップロードする必要があります。リソースの作成とアップロード方法については、[リソースセクション](#!resources/resources.md)を参照してください。

この時点では、スクリプトに変更を加えるには、元のテキストファイルでスクリプトを編集してから、リソースにファイルを再アップロードする必要があります。Gravwellの将来のバージョンでは、スクリプトをよりシンプルにするために、統合されたテキストエディタが含まれます。

## ankoスクリプトの実行

スクリプトを実行するには、"anko"の後にスクリプト名を続け、そのスクリプトに必要な引数を指定するだけです。例えば、2つの数字を引数に取る "foo "という名前のスクリプトを実行するには、パイプラインに`anko foo 1 3`と入力します。

## ankoのスクリプトを書く

ankoスクリプトは eval コマンドと同じ構文を使用します。

### 重要な機能の定義

ankoスクリプトには、`Process`という名前の関数か、`Main`という名前の関数が含まれていなければなりません。これらの関数は引数を取りません。これら2つの関数は、検索エントリを処理するための2つの異なるオプションを表しています。

`Process`関数が定義されている場合、検索エントリごとに1回呼び出されます。エントリの列挙値はローカル変数として扱われる場合があり、`Process`関数の戻り値によって、エントリがパイプラインを続行できるか（true）、許可されないか（false）が決まります。

したがって、プログラマは `readEntry` と `writeEntry` 関数を呼び出して各検索項目を取得し、それを操作した後にそれを行に渡す必要があります。

可能な限り、`Main`ではなく`Process`でスクリプトを書くことを強く推奨します。

### オプションの関数定義

スクリプトには `Parse` と `Finalize` という名前の関数も含まれています。

関数 `Parse` は `Process` や `Main` の前に呼び出され、コマンドラインの引数を引数の配列として与えます。nil を返すことで引数が正常に処理されたことを示し、nil 以外の値はエラーとして扱われ、ユーザに表示されます。スクリプトの引数をパースする方法のサンプルについては、以下のサンプルスクリプトを参照してください。

注意:Parse関数は明示的に値を返さなければなりません(MUST)。nilを返せばパースが成功したことを示し、それ以外の値を返せばエラーを示します。エラーの場合は、問題を説明する文字列を返すことをお勧めします。

`Finalize`関数は、`Process`または`Main`が完了した後に呼び出されます。 これは、スクリプトで実行される最後のコードです。 これは、必要に応じてリソースを作成するのに適した場所です。

### Process()関数を使用したサンプルスクリプト

この例のスクリプトは、スクリプトの引数として指定された2つのモードで動作します。「build」モードでは、パケットモジュールから抽出された「SrcIP」フィールドを受け取り、現在の検索で見られるIPアドレスのリストを構築し、そのリストをリソースとして保存します。「apply」モードでは、以前に構築されたテーブルを使用し、以前に表示された「SrcIP」フィールドを含むエントリをすべて削除します。このスクリプトはネットワーク上の新しいデバイスを検索するために使用されていましたが、`lookup`モジュールは同じ機能をより柔軟に提供するようになりました。

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

列挙された "SrcIP "の値は、`Process`関数内の他の変数と同じように**read**できることに注意してください。

`table`変数と`task`変数は関数定義の外で宣言されていることに注意してください。 組み込みのjsonエンコーディングライブラリも、関数定義の前にインポートされます。 利用可能なライブラリの完全なリストを以下に示します。


### Main()関数を使用したサンプルスクリプト

`Main`関数を使用してスクリプトを書くことはより困難ですが、エントリを複製する必要がある場合には必要です。もしメッセージがタイプ0x10（"Write multiple registers")のリクエストであれば、スクリプトは書き込まれるレジスタごとに元のエントリを複製し、各エントリに単一のレジスタアドレス＋レジスタ値を含む "RegAddr "と "RegValue "の列挙値を付加します。

注：このスクリプトはそれ自体では機能しません。 記述されているように、パイプラインの早い段階で別のankoスクリプトの出力を消費することを目的としており、「Request」や「WriteAddr」などの列挙値が入力されます。

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

`readEntry`、`cloneEntry`、および `writeEntry`関数の使用に注意してください。 この検索エントリの明示的な管理は、`Process`関数を使用するスクリプトでは必要ありません。`getEntryEnum`および`setEntryEnum`関数を使用して、列挙値を変数として扱うのではなく読み取ることにも注意してください。

## ユーティリティー機能

ankoは組み込みのユーティリティ関数を提供しており、`functionName(<functionArgs>) <returnValues>`の形式で以下に列挙しています。以下の関数は、どのアンコスクリプトでも使用できます。

* `getResource(name) []byte, error` は指定されたリソースの内容をバイト数で返し、エラーはリソースの取得中に発生したエラーです。
* `setResource(name, value) error` は、(必要に応じて) `name` という名前のリソースを `value` の内容で作成して更新し、エラーが発生した場合にはエラーを返します。
* `len(val) int` valの長さを返します。これは、文字列、スライスなどにすることができます。
* `getMacro(name) string, error`は与えられたマクロの値を返し、存在しない場合はエラーを返します。この関数はマクロの展開を行わないことに注意してください。
* `toIP(string) IP` 文字列をIPに変換し、例えばパケットモジュールで生成されたIPと比較するのに適しています。
* `toMAC(string) MAC` 文字列をMACアドレスに変換します。
* `toString(val) string` valを文字列に変換します。
* `toInt(val) int64` はvalを可能であれば整数に変換します。変換できない場合は0を返します。
* `toFloat(val) float64` は、可能であればvalを浮動小数点数に変換します。変換できない場合は0.0を返します。
* `toBool(val) bool` はvalをブール値に変換しようとします。変換できなかった場合は、falseを返します。0以外の数値と文字列 "y", "yes", "true "はtrueを返します。
* `toHumanSize(val) string` valを整数に変換し、それを人間が読めるバイトカウントとして表現しようとします。 `toHumanSize（15127）`は「14.77KB」に変換されます。
*`toHumanCount（val）string`は、valを整数に変換し、それを人間にわかりやすい数値として表現しようとします。 `toHumanCount（15127）`は「15.13K」に変換されます。
* `typeOf(val) type` はvalの型を文字列として返す。
* `producesEnum(val)` スクリプトがその名前の列挙値を生成する予定であることをパイプラインに通知します。Parse()関数で呼び出す必要があります。
* `consumesEnum（val）`スクリプトがその名前の列挙値を消費することを計画していることをパイプラインに通知します。Parse()関数で呼び出す必要があります。

以下の関数は、`Process` 関数を実装したスクリプトでのみ利用可能である。

* `setEnum(key, value) error` は、`value` を含む `key` という名前の現在のエントリに列挙された値を作成します。
* `getEnum(key) value, error` は `key` で指定された列挙値を返します。
* `delEnum(key)` は、現在のエントリから `key` という名前の列挙値を削除します。
* `hasEnum(key)bool` は、現在のエントリにその列挙値があるかどうかを返します。

次の関数は、 `Main`関数を実装するスクリプトでのみ使用できます。

* ` readEntry() entry, error` は次のエントリとエラー(もしあれば)を返します。エントリが残っていない場合はエラーを返します。
* `writeEntry(ent) error` は指定されたエントリをパイプラインから書き出し、エラーがあればそれを返します。
* `cloneEntry(ent) entry` は指定されたエントリのコピーを返します。
* `dupEntryByReference(ent, names...) entry`は指定されたエントリを複製し、`names` パラメータ(複数可)で指定された列挙された値のコピーを作成します。
* `newEntry() entry` は、現在のクエリの終了時刻をタイムスタンプとして、完全に新しいエントリを作成します。
* `setEntryEnum(ent, key, value)` 指定されたエントリに列挙された値を設定します。
* `getEntryEnum(ent, key) value, error` 指定されたエントリから列挙された値を読み込みます。
* `hasEntryEnum(ent, key) bool` は、エントリに列挙された値が含まれているかどうかを返します。
* `delEntryEnum(ent, key)` 指定されたエントリから指定された列挙された値を削除します。
* `setEntryData(ent, value)` エントリのデータ部分を設定します。
* `setEntrySrc(ent, ip)` はエントリのソースフィールドを設定します。
* `setEntryTimestamp(ent, time)` はエントリのタイムスタンプを設定します。

注意:`Process` 関数と `Main` 関数を用いたスクリプトでは `setEnum`, `hasEnum`, `delEnum` 関数が異なるのは、`Process` 関数が特定のエントリに対して暗黙のうちに動作しているからです。

## 組み込み変数

ankoスクリプトには以下の変数があらかじめ定義されています:

* `START`:クエリの開始時刻を指定します。
* `END`:クエリの終了時刻を指定します。
* `TAGMAP`:文字列タグ名のentry.EntryTagタグ番号へのマップ。 これには、現在のクエリで使用されているタグのみが含まれるため、 `tag = default、foo`と言うと、TAGMAPには 'default'→0および 'foo'→1が含まれます。 これを `cloneEntry`または` newEntry`関数と組み合わせて使用します。

## 利用可能なパッケージ

次のような構文で、特定のGoライブラリのankoラッパーをインポートすることができます。

```
var json = import("encoding/json")
```

セキュリティ上の理由から、ankoモジュールは、完全なankoスクリプト言語に含まれる*すべての*パッケージへのアクセスを許可しません。 次のパッケージは、ankoスクリプトで使用できます。

* [bytes](https://golang.org/pkg/bytes)：バイトスライスを処理します。
* [crypto/md5](https://golang.org/pkg/crypto/md5), [crypto/sha1](https://golang.org/pkg/crypto/sha1), [crypto/sha256](https://golang.org/pkg/crypto/sha256), [crypto/sha512](https://golang.org/pkg/crypto/sha512):暗号化ハッシュ
* [crypto/tls](https://golang.org/pkg/crypto/tls): 制限付きTLS機能
* [encoding/base64](https://golang.org/pkg/encoding/base64): 制限付きベース64機能
* [encoding/csv](https://golang.org/pkg/encoding/csv)：CSVデータをエンコードおよびデコードします。
* [encoding/hex](https://golang.org/pkg/encoding/hex): 制限付きHEXエンコーディング 機能
* [encoding/json](https://golang.org/pkg/encoding/json)：jsonデータをエンコードおよびデコードします。
* [encoding/xml](https://golang.org/pkg/encoding/xml): 制限付きXMLエンコーディング機能
* [errors](https://golang.org/pkg/errors)：Goエラーを処理します。
* [flag](https://golang.org/pkg/flag):：コマンドライン引数を処理します。
* [fmt](https://golang.org/pkg/fmt)：文字列の印刷とフォーマット
* [github.com/google/uuid](https://github.com/google/uuid): UUIDの生成および検索
* [github.com/gravwell/ipexist](https://github.com/gravwell/ipexist): Gravwell IPヘルパー機能
* [github.com/RackSec/srslog](https://github.com/RackSec/srslog): golangの標準ライブラリに代わるsyslogパッケージ
* [io](https://golang.org/pkg/io): basic I/O primitives
* [io/util](https://golang.org/pkg/io/util): ただの`ioutil.ReadAll`関数 (以下を参照)
* [math](https://golang.org/pkg/math)：数学関数
* [math/big](https://golang.org/pkg/math/big)：bignums
* [math/rand](https://golang.org/pkg/math/rand)：乱数
* [net](https://golang.org/pkg/net): 制限付きネットワーク機能
* [net/https](https://golang.org/pkg/net/https)：制限されたHTTP機能（クライアントのみ）
* [net/url](https://golang.org/pkg/net/url)：URL
* [path](https://golang.org/pkg/path)：パス
* [path/filepath](https://golang.org/pkg/path/filepath)：ファイルに固有のパス関数
* [regexp](https://golang.org/pkg/regexp)：正規表現
* [sort](https://golang.org/pkg/sort)：並べ替え
* [strings](https://golang.org/pkg/strings):：文字列処理関数
* [time](https://golang.org/pkg/time)：時間処理関数
* [github.com/ziutek/telnet](https://github.com/ziutek/telnet): telnetクライアント関数 (Dial, DialTimeout, NewConn関数が利用できます。ドキュメントは [https://godoc.org/github.com/ziutek/telnet](https://godoc.org/github.com/ziutek/telnet) を参照してください。

このドキュメントでは、すべてのパッケージを網羅的に説明することはできません。各パッケージにエクスポートされた利用可能な機能は、 [公式 anko リポジトリ](https://github.com/mattn/anko/tree/master/packages) で見ることができます。公式 anko リポジトリでエクスポートされた完全な機能を提供していないパッケージもありますので、以下でさらに説明します。

## パッケージの制限

パッケージの中には、スクリプト言語を使ってエクスポートするのは危険な可能性がある機能を持っているものがあります。Gravwell は、特定のパッケージのエクスポートを、フルパッケージで利用可能なもののサブセットに制限しています。以下に、各パッケージの制限事項の一覧を示します。

### crypto/md5

`crypto/md5` は "New" と "Sum" 関数のみをエクスポートします:

- `md5.New`
- `md5.Sum`

### crypto/sha1

`crypto/sha1` は "New" と "Sum" 関数のみをエクスポートします:

- `sha1.New`
- `sha1.Sum`

### crypto/sha256

`crypto/sha256`は様々な "New" と "Sum" 関数のみをエクスポートします:

- `sha256.New`
- `sha256.New224`
- `sha256.Sum224`
- `sha256.Sum256`

### crypto/sha512

`crypto/sha512`は様々な "New" と "Sum" 関数のみをエクスポートします:

- `sha512.New`
- `sha512.New384`
- `sha512.New512_224`
- `sha512.New512_256`
- `sha512.Sum384`
- `sha512.Sum512`
- `sha512.Sum512_224`
- `sha512.Sum512_256`

### crypto/tls

このモジュールは、Gravwellの設定で`Disable-Network-Script-Functions`が`false`に設定されている場合にのみ利用可能です。`crypto/tls`は、`net/http`モジュールで使用するTLS設定タイプをエクスポートするだけです:

- `tls.Config`

### encoding/csv

`encoding/csv`はCSVのイニシャライザのみをエクスポートします:

- `csv.NewReader` (LazyQuotes オプションを true に設定した状態で `csv.NewReader` を実際に呼び出します)
- `csv.NewWriter`
- `csv.NewBuilder`

### encoding/base64

`encoding/base64`はbase64の初期化子とエンコーディング型のみをエクスポートします:

- `base64.NewDecoder`
- `base64.NewEncoder`
- `base64.NewEncoding`
- `base64.RawStdEncoding`
- `base64.RawURLEncoding`
- `base64.StdEncoding`
- `base64.URLEncoding`

### encoding/hex

`encoding/hex`は、イニシャライザとラッパーのサブセットをエクスポートします:

- `hex.Decode`
- `hex.DecodeString`
- `hex.DecodedLen`
- `hex.Dump`
- `hex.Dumper`
- `hex.Encode`
- `hex.EncodeToString`
- `hex.EncodedLen`
- `hex.NewDecoder`
- `hex.NewEncoder`

### encoding/xml

`encoding/exml`は、イニシャライザ、ラッパー、エンコーディングオプションのサブセットをエクスポートします:

- `xml.Escape`
- `xml.EscapeText`
- `xml.Marshal`
- `xml.MarshalIndent`
- `xml.Unmarshal`
- `xml.NewDecoder`
- `xml.NewTokenDecoder`
- `xml.NewEncoder`
- `xml.HTMLAutoClose`
- `xml.HTMLEntity`
- `xml.Attr`
- `xml.CharData`
- `xml.Comment`
- `xml.Directive`
- `xml.EndElement`
- `xml.Name`
- `xml.ProcInst`
- `xml.StartElement`

### flag 

`flag`は `flag` パッケージ全体ではなく、型のサブセットのみをエクスポートします:

- `flag.NewFlagSet`
- `flag.PanicOnError`
- `flag.ContinueOnError`

### github.com/google/uuid

`github.com/google/uuid` は"New"と "Parse"の関数のみをエクスポートします。

- `uuid.New`
- `uuid.Parse`
- `uuid.ParseBytes`

### github.com/gravwell/ipexist

このモジュールは、Gravwellの設定で `Disable-Network-Script-Functions` が `false` に設定されている場合にのみ利用可能です。`github.com/gravwell/ipexist` は "New"関連の関数のみをエクスポートします:

- `ipexist.New`
- `ipexist.NewIPBitMap`

### github.com/RackSec/srslog

このモジュールは、Gravwellの設定で`Disable-Network-Script-Functions`が `false`に設定されている場合にのみ利用可能です。`github.com/RackSec/srslog`はsyslog関連の機能のみを公開します:

- `srslog.Dial`
- `srslog.DefaultFormatter`
- `srslog.DefaultFramer`
- `srslog.RFC3164Formatter`
- `srslog.RFC5424Formatter`
- `srslog.RFC5425MessageLengthFramer`
- `srslog.UnixFormatter`
- `srslog.LOG_EMERG`
- `srslog.LOG_ALERT`
- `srslog.LOG_CRIT`
- `srslog.LOG_ERR`
- `srslog.LOG_WARNING`
- `srslog.LOG_NOTICE`
- `srslog.LOG_INFO`
- `srslog.LOG_DEBUG`
- `srslog.LOG_KERN`
- `srslog.LOG_USER`
- `srslog.LOG_MAIL`
- `srslog.LOG_DAEMON`
- `srslog.LOG_AUTH`
- `srslog.LOG_SYSLOG`
- `srslog.LOG_LPR`
- `srslog.LOG_NEWS`
- `srslog.LOG_UUCP`
- `srslog.LOG_CRON`
- `srslog.LOG_AUTHPRIV`
- `srslog.LOG_FTP`
- `srslog.LOG_LOCAL0`
- `srslog.LOG_LOCAL1`
- `srslog.LOG_LOCAL2`
- `srslog.LOG_LOCAL3`
- `srslog.LOG_LOCAL4`
- `srslog.LOG_LOCAL5`
- `srslog.LOG_LOCAL6`
- `srslog.LOG_LOCAL7`

### github.com/ziutek/telnet

このモジュールは、Gravwell設定で `Disable-Network-Script-Functions` が `false` に設定されている場合にのみ利用可能です。エクスポートされる関数と型は以下の通りです:

- `telnet.Dial`
- `telnet.DialTimeout`
- `telnet.NewConn`
- `telnet.Conn`

### io/ioutil

`io/ioutil`は`ioutil.ReadAll()`という関数を1つだけエクスポートします。

### net

このモジュールは、Gravwell設定で `Disable-Network-Script-Functions` が `false` に設定されている場合にのみ利用可能です。エクスポートされる関数と型は以下の通りです:

- `net.CIDRMask`
- `net.Dial`
- `net.DialIP`
- `net.DialTCP`
- `net.DialTimeout`
- `net.DialUDP`
- `net.ErrWriteToConnected`
- `net.FlagBroadcast`
- `net.FlagLoopback`
- `net.FlagMulticast`
- `net.FlagPointToPoint`
- `net.FlagUp`
- `net.IPv4`
- `net.IPv4Mask`
- `net.IPv4allrouter`
- `net.IPv4allsys`
- `net.IPv4bcast`
- `net.IPv4len`
- `net.IPv4zero`
- `net.IPv6interfacelocalallnodes`
- `net.IPv6len`
- `net.IPv6linklocalallnodes`
- `net.IPv6linklocalallrouters`
- `net.IPv6loopback`
- `net.IPv6unspecified`
- `net.IPv6zero`
- `net.InterfaceAddrs`
- `net.InterfaceByIndex`
- `net.InterfaceByName`
- `net.Interfaces`
- `net.JoinHostPort`
- `net.LookupAddr`
- `net.LookupCNAME`
- `net.LookupHost`
- `net.LookupIP`
- `net.LookupMX`
- `net.LookupNS`
- `net.LookupPort`
- `net.LookupSRV`
- `net.LookupTXT`
- `net.ParseCIDR`
- `net.ParseIP`
- `net.ParseMAC`
- `net.ResolveIPAddr`
- `net.ResolveTCPAddr`
- `net.ResolveUDPAddr`
- `net.ResolveUnixAddr`
- `net.SplitHostPort`

### net/http

このモジュールは、Gravwellの設定で `Disable-Network-Script-Functions` が `false` に設定されている場合にのみ利用可能です。


`net/http`は、HTTP *リクエスト*を実行するための関数、タイプ、変数のサブセットをエクスポートします。 タイプ `Client`、`Cookie`、`Request`、および` Response`がエクスポートされます。 これらのタイプの説明については、[Goのドキュメント](https://golang.org/pkg/net/http/)を参照してください。

関数 `NewRequest（operation、url、body）（Request、error）`は、指定されたURL文字列に対して指定された操作（ "PUT"、"POST"、"GET"、"DELETE"）を使用して新しいhttp.Requestを準備します。 本文は、PUTおよびPOSTリクエストで使用するためのオプションのパラメーターです。 'nil'またはio.Readerのいずれかに設定する必要があります。

ほとんどの場合、NewRequest関数を使用してリクエストを作成し、http.DefaultClientを使用してそのリクエストを実行します。

```
req, _ = http.NewRequest("GET", "http://example.org/foo", nil)
resp, _ = http.DefaultClient.Do(req)
resp.Body.Close()
```

送信前にリクエストに追加のヘッダやクッキーを設定することができます。

```
req, _ = http.NewRequest("GET", "http://example.org/foo", nil)
# Add a header
req.Header.Add("My-Header", "gravwell")
# Add a cookie
cookie = make(http.Cookie)
cookie.Name = "foo"
cookie.Value = "bar"
req.AddCookie(&cookie)
resp, _ = http.DefaultClient.Do(req)
resp.Body.Close()
```

警告：上記のように、終了したらhttp.ResponseのBodyフィールドを閉じる必要があります。 Bodyを開いたままにすると、ネットワーク接続が開いたままになり、最終的に検索エージェントのソケットが不足します。 `httpGet`関数と` httpPost`関数はBodyを自動的に閉じます。 可能な限りそれらの使用を検討してください。