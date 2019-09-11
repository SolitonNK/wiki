# 検索スクリプト

Gravwellは、検索の実行、リソースの更新、アラートの送信、またはアクションを実行できる堅牢なスクリプトエンジンを提供します。オーケストレーションエンジンにより、調査の面倒な手順を自動化し、検索結果に基づいて人間を巻き込むことなくアクションを実行できます。

オーケストレーションスクリプトは、[スケジュール](scheduledsearch.md)に従って実行することも、[コマンドラインクライアント](#!cli/cli.md)から手動で実行することもできます。CLIではスクリプトを対話形式で再実行できるため、スケジュール検索を作成する前にCLIでスクリプトを開発してテストすることをお勧めします。

## 組み込み関数
スクリプト検索では、 [anko](#!scripting/anko.md)モジュールで使用可能なものにほぼ一致する組み込み関数を使用できます。検索の起動および管理用にいくつかの追加機能があります。機能は以下の形式でリストされています
```
functionName(<functionArgs>) <returnValues>
```
### リソースと永続データ

* `getResource(name) []byte, error` 指定されたリソースの内容をバイトスライスで返します。エラーは、リソースの取得中に発生したエラーです。
* `setResource(name, value) error` の内容で指定された`name`のリソースを（必要なら）作成して更新し`value`、エラーが発生したらそれを返します。
* `setPersistentMap(mapname, key, value)` スケジュールされたスクリプトの実行間で持続するキーと値のペアをマップに格納します。
* `getPersistentMap(mapname, key) value` 指定された永続マップから、指定されたキーに関連付けられている値を返します。
* `delPersistentMap(mapname, key)` 指定されたマップから指定されたキーと値のペアを削除します。

### 検索エントリ操作

* `setEntryEnum(ent, key, value)` 指定されたエントリに列挙値を設定します。
* `getEntryEnum(ent, key) value, error` 指定されたエントリから列挙値を読み取ります。
* `delEntryEnum(ent, key)` 指定されたエントリから指定された列挙値を削除します。 

### 一般ユーティリティ

* `len(val) int` の長さを返します。これは文字列、スライスなどです。
* `toIP(string) IP` 文字列をIPに変換します。パケットモジュールなどで生成されたIPと比較するのに適しています。
* `toMAC(string) MAC` 文字列をMACアドレスに変換します。
* `toString(val) string` valを文字列に変換します。
* `toInt(val) int64` 可能であればvalを整数に変換します。変換が不可能な場合は0を返します。
* `toFloat(val) float64` 可能であればvalを浮動小数点数に変換します。変換が不可能な場合は0.0を返します。
* `toBool(val) bool`  boolvalをブール値に変換しようとします。変換が不可能な場合はfalseを返します。ゼロ以外の数字と文字列 "y"、 "yes"、および "true"はtrueを返します。
* `typeOf(val) type`  valの型を文字列として返します。例えば、 "string"、 "bool"です。

### 検索管理
Gravwellの検索システムが機能する方法のため、このセクションの関数の中には（構造体のよう`search` に書かれた）検索構造体を返すものもあれば（模型の中に書かれた）検索IDを返すものも`searchID`あります。各Search構造体には、アクセス可能な検索IDが含まれています`searchID`。

検索構造体は検索からエントリを積極的に読み込むために使用されますが、検索IDは、私たちが添付したり管理したりする可能性のある非アクティブ検索を指す傾向があります。

* `startBackgroundSearch(query, start, end) (search, err)` `start`と `end`で指定された時間範囲で実行され、与えられたクエリ文字列でバックグラウンド検索を作成します。戻り値はSearch構造体です。これらの時間値は、タイムライブラリを使用して指定する必要があります。デモンストレーション用の例を参照してください。
* `startSearch(query, start, end) (search, err)` まったく同じよう`startBackgroundSearch`に動作しますが、検索の背景にはなりません。
* `detachSearch(search)` 与えられた検索（Search構造体）を切り離します。これにより、非バックグラウンド検索が自動的にクリーンアップされ、検索が終了したときにはいつでも呼び出すことができます。
* `attachSearch(searchID) (search, error)` 与えられたIDで検索にアタッチし、エントリなどを読むために使用できる検索構造体を返します。
* `getSearchStatus(searchID) (string, error)` 指定された検索のステータス、例えば "SAVED"を返します。
* `getAvailableEntryCount(search) (uint64, bool, error)` 指定された検索から読み込むことができるエントリの数、検索が完了したかどうかを示すブール値、何か問題があった場合はエラーを返します。
* `getEntries(search, start, end) ([]SearchEntry, error)` 指定された検索から指定されたエントリを取得します。の境界`start`と関数`end`で見つけることができます `getAvailableEntryCount`。
* `isSearchFinished(search) (bool, error)` 与えられた検索が完了したらtrueを返します.
* `executeSearch(query, start, end) ([]SearchEntry, error)` 検索を開始し、検索が完了するのを待ち、最大1万件のエントリを取得し、検索から切り離してエントリを返します。
* `deleteSearch(searchID) error` 指定したIDの検索を削除します。
* `backgroundSearch(searchID) error` 指定された検索をバックグラウンドに送信します。これは後で手作業で検査するための検索を「維持」するのに役立ちます。
* `downloadSearch(searchID, format, start, end) ([]byte, error)` ユーザーがWeb UIの「ダウンロード」ボタンをクリックした場合と同じように、指定された検索をダウンロードします。`format`必要に応じて、 "json"、 "csv"、 "text"、 "pcap"、または "lookupdata"のいずれかを含む文字列にしてください。`start`と`end`は時間の値です。
* `getDownloadHandle(searchID, format, start, end) (io.Reader, error)` ユーザーがWeb UIの「ダウンロード」ボタンをクリックした場合と同様に、指定された検索の結果にストリーミングハンドルを返します。返されるハンドルは、このドキュメントで後述するHTTPライブラリ関数での使用に適しています。

### 検察データ型
startSearchまたはstartBackgroundSearch関数を介して検索を実行すると、`search` データ型が返されます。`search` データ型は、以下のメンバーが含まれています。

* `ID` - 検索IDを含む文字列。getSearchStatusやattachSearchなどの他の機能にこのメンバーを使用します。
* `RenderMod` - 検索に関連付けられているレンダラーを示す文字列。raw、text、table、chart、またはfdgのようなものです。
* `SearchString` - リクエスト中に渡された検索文字列を含む文字列
* `SearchStart` -  検索の開始タイムスタンプを含む文字列
* `SearchEnd` - 検索の終了タイムスタンプを含む文字列
* `Background` - 検索がバックグラウンド検索として開始されたかどうかを示すブール値
* `Name` - 検索名を含むオプションの文字列。

### 結果を送信する

スクリプトシステムは、スクリプトの結果を外部システムに送信するためのいくつかの方法を提供します。

以下の関数は基本的なHTTP機能を提供します：

* `httpGet(url) (string, error)` 与えられたURLに対してHTTP GETリクエストを実行し、レスポンスボディを文字列として返します。
* `httpPost(url, contentType, data) (response, error)` 指定されたコンテンツタイプ（ "application / json"など）と指定されたデータをPOST本文として指定されたURLにHTTP POST要求を実行します。
* "net / http"ライブラリを使えばもっと複雑なHTTP操作が可能です。利用可能なものの説明についてはankoドキュメントのパッケージドキュメントを見てください、または例については以下を見てください。

ユーザーがGravwell内で自分の個人的なEメール設定を構成している場合、この`email`機能はEメールを送信するための非常に簡単な方法です。

* `email(from, to, subject, message) error`SMTP経由でEメールを送信します。この`from`フィールドは単なる文字列ですが、`to`電子メールアドレスを含む文字列のスライスである必要があります。フィールドは、電子メールの件名と本文を含まなければならない文字列です。subjectmessage
* 次の機能は廃止予定ですがまだ利用可能で、ユーザーの電子メールオプションを設定せずに電子メールを送信することができます。

* `sendMail(hostname, port, username, password, from, to, subject, message) error` SMTP経由でEメールを送信します。`hostname`そして`port`使用するSMTPサーバーを指定します。`username` そして`password`、サーバーへの認証のためのものです。`from`一方、フィールドは、単に文字列である`to`フィールドは、電子メールアドレスを含む文字列のスライスでなければなりません。フィールドは、電子メールの件名と本文を含まなければならない文字列です。subjectmessage

*`sendMailTLS(hostname, port, username, password, from, to, subject, message, disableValidation) error`TLSを使用してSMTP経由でEメールを送信します。`hostname`そして`port`使用するSMTPサーバーを指定します。`username`そして`password`、サーバーへの認証のためのものです。`from`一方、フィールドは、単に文字列である`to`フィールドは、電子メールアドレスを含む文字列のスライスでなければなりません。フィールドは、電子メールの件名と本文を含まなければならない文字列です。disableValidation引数は、TLS証明書の検証を無効にするブール値です。disableValidationをtrueに設定することは安全ではなく、電子メールクライアントを中間者攻撃にさらす可能性があります。subjectmessage

### 通知を作成する

スクリプトは、スクリプトの所有者を対象とした通知を作成する場合があります。 通知は、整数ID、文字列メッセージ、オプションのHTTPリンク、および有効期限で構成されます。 有効期限が過去の場合、または将来24時間を超える場合、Gravwellは代わりに有効期限を12時間に設定します。

	addSelfTargetedNotification(7, "This is my notification", "https://gravwell.io", time.Now().Add(3 * time.Hour)

通知IDは、通知を一意に識別します。 これにより、ユーザーは同じ通知IDで関数を再度呼び出して既存の通知を更新できますが、異なるIDを指定して複数の同時通知を追加することもできます。

### エントリの作成と取り込み

次の機能を使用して、スクリプト内からインデクサーに新しいエントリを取り込むことができます。

* `newEntry（time.Time、data）Entry`は、指定されたタイムスタンプ（time.Time、time.Now（）のように）とデータ（多くの場合文字列）で新しいエントリを返します。
* `ingestEntries（[] Entry、tag）error`は、指定されたタグ文字列を持つ指定されたエントリのスライスを取り込みます。

`getEntries`関数によって返されたエントリは、必要に応じて変更して` ingestEntries`で再取り込みするか、新しいエントリを大量に作成できます。 たとえば、以前の検索の一部のエントリをタグ「newtag」に再度取り込むには：

```
# Get the first 100 entries from the search
ents, _ = getEntries(mySearch, 0, 100)
ingestEntries(ents, "newtag")
```

他の条件に基づいて新しいエントリを取り込むには：

```
if condition == true {
	ents = make([]Entry)
	ent += newEntry(time.Now(), "Script condition triggered")
	ingestEntries(ents, "results")
}
```

### その他のネットワーク機能

一連のラッパー関数がSSHおよびSFTPクライアントへのアクセスを提供します。これらのリターン構造に呼び出すことができる方法の詳細については、[sshのライブラリのドキュメント](https://godoc.org/golang.org/x/crypto/ssh) や[SFTPライブラリのドキュメント](https://godoc.org/golang.org/x/crypto/ssh) を参照してください.

* `sftpConnectPassword(hostname, username, password, hostkey) (*sftp.Client, error)`指定されたユーザー名とパスワードを使用して、指定されたSSHサーバー上でSFTPセッションを確立します。hostkeyパラメータがnil以外の場合、ホストからの公開鍵としてホスト鍵の検証を実行するために使用されます。hostkeyパラメータがnilの場合、ホスト鍵の検証はスキップされます。
* `sftpConnectKey(hostname, username, privkey, hostkey) (*sftp.Client, error)` 提供された秘密鍵（文字列または[]バイト）を使用して、指定されたユーザー名で指定されたSSHサーバー上でSFTPセッションを確立します。
* `sshConnectPassword(hostname, username, password, hostkey) (*ssh.Client, error)` 与えられたホスト名に対してSSHクライアントを返し、パスワードで認証します。Clientを確立したら、通常client.NewSession（）を呼び出して使用可能なセッションを確立します。下記のgoのドキュメントや例を見てください。
* `sshConnectKey(hostname, username, privkey, hostkey) (*sftp.Client, error)` 提供された秘密鍵（文字列または[]バイト）を使用して、指定されたユーザー名で指定されたSSHサーバーに接続して認証します。

Note: ホスト鍵のパラメータは、known_hostsファイル/ authorized_keysの形式である必要があり、例えば"ECDSA-SHA2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOcrwoHMonZ / l3OJOGrKYLky2FHKItAmAMPzZUhZEgEb86NNaqfdAj4qmiBDqM04 / o7B45mcbjnkTYRuaIUwkno ="。〜/ .ssh / known_hostsから適切なキーを抽出するには、実行してください`ssh-keygen -H -F <hostname>`.

telnetライブラリもあります。直接ラッパーは提供されていません`github.com/ziutek/telnet`が、スクリプトにインポートしてtelnet.Dialなどを呼び出すことで使用できます。以下の例は、telnetライブラリの簡単な使用法を示しています。

最後に、低レベルのGo [ネットライブラリ](https://golang.org/pkg/net)が利用可能です。 リスナー機能は無効になっていますが、スクリプトはIP解析機能とDial、DialIP、DialTCPなどのダイヤル機能を使用する場合があります。例については以下を参照してください。

#### SFTPの例

このスクリプトは、パスワード認証を使用し、ホストキーチェックを使用しないでSFTPサーバーに接続します。ユーザ「sshtest」としてログインし、そのユーザのホームディレクトリの内容を印刷して、「hello.txt」という名前の新しいファイルを作成します。
```
conn, err = sftpConnectPassword("example.com:22", "sshtest", "foobar", nil)
if err != nil {
	println(err)
	return
}

w = conn.Walk("/home/sshtest")
for w.Step() {
    if w.Err() != nil {
        continue
    }
    println(w.Path())
}

f, err = conn.Create("/home/sshtest/hello.txt")
if err != nil {
    conn.Close()
	println(err)
	return
}
_, err = f.Write("Hello world!")
if err != nil {
    conn.Close()
	println(err)
	return
}

// check it's there
fi, err = conn.Lstat("hello.txt")
if err != nil {
    conn.Close()
	println(err)
	return
}
println(fi)
conn.Close()
```

#### SSHの例

このスクリプトは、公開鍵認証を使用してサーバーに接続します。ここでは、読みやすくするために秘密鍵ブロックを短くしています。ホスト鍵の検証も行います。その後、実行/bin/ps auxして結果を印刷します。

```
var bytes = import("bytes")

# Get this via `ssh-keygen -H  -F <hostname`
pubkey = "ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOcrwoHMonZ/l3OJOGrKYLky2FHKItAmAMPzZUhZEgEb86NNaqfdAj4qmiBDqM04/o7B45mcbjnkTYRuaIUwkno="

privkey = `-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAr1vpoiftxU7Jj7P0bJIvgCQLTpM0tMrPmuuwvGMba/YyUO+A
[...]
5iMqZFaUncYZyOFE9hhHqY1xhgwxyjgCTeaI/J/KfbsaCSvrkeBq
-----END RSA PRIVATE KEY-----`

# Log in with a private key
conn, err = sshConnectKey("example.com:22", "sshtest", privkey, pubkey)
if err != nil {
	println(err)
	return
}

session, err = conn.NewSession()
if err != nil {
    println("Failed to create session: ", err)
	return err
}

// Once a Session is created, you can execute a single command on
// the remote side using the Run method.
var b = make(bytes.Buffer)
session.Stdout = &b
err = session.Run("/bin/ps aux")
if err != nil {
    println("Failed to run: " + err.Error())
	return err
}
println(b.String())

session.Close()
```

#### Telnetの例

このスクリプトはtelnetサーバに接続し、 "testing"というパスワードでrootとしてログインしてから、prompt（`$`）までの間に受け取ったものすべてを出力します。

```
var telnet = import("github.com/ziutek/telnet")
t, err = telnet.Dial("tcp", "example.org:23")
if err != nil {
	println(err)
	return
}
t.SetUnixWriteMode(true)
b, err = t.ReadUntil(":")
println(toString(b))
t.Write("root\n")
b, err = t.ReadUntil(":")
println(toString(b))
t.Write("testing\n")
for {
	r, err = t.ReadUntil("$ ")
	print(toString(r))
}
```

#### Netの例

このスクリプトは、TCPポート7778でlocalhostに接続し、「foo」を接続に書き込みます。 `nc -l 7778`を実行すると、netcatに対してこれをテストできます。

```
var net = import("net")

c, err = net.Dial("tcp", "localhost:7778")
if err != nil {
	println(err)
	return err
}

c.Write("foo")
c.Close()
```

## 検索スクリプトの例

このスクリプトは、過去1日間にどのIPがCloudflareの1.1.1.1 DNSサービスと通信したかを検索するバックグラウンド検索を作成します。結果が見つからない場合、検索は削除されますが、結果がある場合、検索はGUIの[Manage Searches]画面でユーザーが後で確認するために残ります。

```
# Import the time library
var time = import("time")
# Define start and end times for the search
start = time.Now().Add(-24 * time.Hour)
end = time.Now()
# Launch the search
s, err = startSearch("tag=netflow netflow Dst==1.1.1.1 Src | unique Src | table Src", start, end)
if err != nil {
	return err
}
printf("s.ID = %v\n", s.ID)
# Wait until the search is finished
for {
	f, err = isSearchFinished(s)
	if err != nil {
		return err
	}
	if f {
		break
	}
	time.Sleep(1 * time.Second)
}
# Find out how many entries were returned
c, _, err = getAvailableEntryCount(s)
if err != nil {
	return err
}
printf("%v entries\n", c)
# If no entries returned, delete the search
# Otherwise, background it
if c == 0 {
	deleteSearch(s.ID)
} else {
	err = backgroundSearch(s.ID)
	if err != nil {
		return err
	}
}
# Always detach from the search at the end of execution
detachSearch(s)
```

スクリプトをテストするために、内容をファイルに貼り付けることができます `/tmp/script `。その後、Gravwell CLIツールを使用してスクリプトを実行します。必要に応じて何度も再実行を促し、実行ごとにファイルを再読み込みします。これにより、スクリプトへの変更を簡単にテストできます。

```
$ gravwell -s gravwell.example.org watch script
Username: <user>
Password: <password>
script file path> /tmp/script
0 entries
deleting 910782920
Hit [enter] to re-run, or [q][enter] to cancel

s.ID = 285338682
1 entries
Hit [enter] to re-run, or [q][enter] to cancel
```

上記の例は、スクリプトが2回実行されたことを示しています。最初の実行では、結果が見つからず、検索は削除されました。2番目のエントリは1エントリ返されたため、検索はそのまま行われました。

CLIでスクリプトをテストするときは、古い検索を削除するように注意してください。スケジュール検索システムは自動的に古い検索を削除しますが、CLIは削除しません。

## スクリプトでHTTPを使う

最近の多くのコンピュータシステムはHTTP要求を使用してアクションを起動します。Gravwellスクリプトは、以下の基本的なHTTP操作を提供します。

* httpGet(url)
* httpPost(url、 コンテンツタイプ、データ)

これらの関数とAnkoに含まれているJSONライブラリを使用して、前のスクリプト例を変更できます。後の閲覧のために検索をバックグラウンドするのではなく、1.1.1.1と通信したIPのリストを作成し、それらをJSONとしてエンコードし、HTTP POSTを実行してそれらをサーバーに送信します。

```
var time = import("time")
var json = import("encoding/json")
var fmt = import("fmt")

# Launch search
start = time.Now().Add(-24 * time.Hour)
end = time.Now()
s, err = startSearch("tag=netflow netflow Dst==1.1.1.1 Src | unique Src", start, end)
if err != nil {
	return err
}
# wait for search to finish
for {
	f, err = isSearchFinished(s)
	if err != nil {
		return err
	}
	if f {
		break
	}
	time.Sleep(1*time.Second)
}
# Get number of entries returned
c, _, err = getAvailableEntryCount(s)
if err != nil {
	return err
}
# Return if no entries
if c == 0 {
	return nil
}
# fetch the entries
ents, err = getEntries(s, 0, c)
if err != nil {
	return err
}
# Build up a list of IPs
ips = []
for i = 0; i < len(ents); i++ {
	src, err = getEntryEnum(ents[i], "Src")
	if err != nil {
		continue
	}
	ips = ips + fmt.Sprintf("%v", src)
}
# Encode IP list as JSON
encoded, err = json.Marshal(ips)
if err != nil {
	return err
}
# Post to an HTTP server
httpPost("http://example.org:3002/", "application/json", encoded)
detachSearch(s)
```

時には、検索結果が非常に大きく、大きすぎてメモリに保持できないことがあります。"net / http"ライブラリをこのgetDownloadHandle機能と組み合わせると、Gravwell検索の結果をHTTP POST / PUT要求に直接ストリーミングすることができます。また、クッキーや追加のヘッダを設定することもできます。

```
var http = import("net/http")
var time = import("time")
var bytes = import("bytes")

start = time.Now().Add(-72 * time.Hour)
end = time.Now()
s, err = startSearch("tag=gravwell", start, end)
if err != nil {
        return err
}
for {
        f, err = isSearchFinished(s)
        if err != nil {
                return err
        }
        if f {
                break
        }
		time.Sleep(1*time.Second)
}

# Get a handle on the search results
rhandle, err = getDownloadHandle(s.ID, "text", start, end)
if err != nil {
        return err
}
# Build the request
req, err = http.NewRequest("POST", "http://example.org:3002/", body)
if err != nil {
        return err
}
# Add a header
req.Header.Add("My-Header", "gravwell")
# Add a cookie
cookie = make(http.Cookie)
cookie.Name = "foo"
cookie.Value = "bar"
req.AddCookie(&cookie)

# Perform the actual request
resp, err = http.DefaultClient.Do(req)
detachSearch(s)
return err
```
