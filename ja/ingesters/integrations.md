# インテグレーション

Gravwellの取り込みフレームワークは、BSD 2-clauseライセンスでオープンソース化されており、オープンソース製品と商用製品の両方に直接組み込むことができます。データプロセッサやジェネレータは、取り込みフレームワークを直接埋め込むことができ、Gravwellとの統合を簡単に構成できます。 このドキュメントページは、Gravwellの統合を強調し、必要に応じてドキュメント化するために使用されます。

## CoreDNS

CoreDNSは、DNSサービスのベースプラットフォームを提供することを目的とした、高度に設定可能なプラグインフレンドリーなDNSサーバーです。 CoreDNSは基本機能として、リレー、プロキシ、または本格的なDNSサーバーとして機能する、堅牢で高パフォーマンスなDNSサーバーを提供します。 CoreDNSはApache-2.0でライセンスされており、[github](https://github.com/coredns/coredns)で入手できます。 CoreDNSの詳細については、[https://coredns.io](https://coredns.io)にアクセスしてください。

### Gravwell CoreDNSプラグイン

Gravwellプラグインは、取り込みフレームワークをCoreDNSに直接組み込んで使用できます。このプラグインを使用すると、静的にコンパイルされた高性能DNSサーバーがDNS監査データをGravwellインスタンスに直接送信できます。プラグインはBSD 2-Clauseの下でライセンスされており、[github](https://github.com/gravwell/coredns)で入手できます。

このプラグインは、高信頼性のためのローカルキャッシング、ロードバランシング、フェイルオーバーなどの機能を一通りサポートする完全な取り込みシステムを提供します。 Gravwell プラグインに関する追加情報は、CoreDNS [外部プラグイン](https://coredns.io/explugins/gravwell/)のページにあります。

#### GravwellでCoreDNSを構築する

Gravwellプラグインを使用してCoreDNSを構築するには、Golangツールチェーンとコンパイラがインストールされている必要があります。詳細は[こちら](https://golang.org/)をご覧ください。

```
go get github.com/coredns/coredns
pushd $GOPATH/src/github.com/coredns/coredns/
sed -i 's/metadata:metadata/metadata:metadata\ngravwell:github.com\/gravwell\/coredns/g' plugin.cfg
go generate
CGO_ENABLED=0 go build -o /tmp/coredns
popd
```

結果として静的にコンパイルされたバイナリは、_/tmp/coredns_に配置されます。有効なCorefileを最初の引数として指定することにより、CoreDNSを開始できます。非rootユーザーとしてCoreDNSを実行している場合、バイナリに[サービスバインド機能](https://wiki.apache.org/httpd/NonRootPortBinding)を追加する必要があります。

```
setcap 'cap_net_bind_service=+ep' /tmp/coredns
```

#### Gravwellプラグインの構成

構成は、**directive** **value**の基本構文を持つCoreDNS Corefileを介して実行されます。 コメントの前には「#」文字が付きます。

次の構成パラメーターを使用できます：

* **Ingest-Secret** は、インデクサーでの認証に使用されるトークンを定義します。 **Ingest-Secret**が必要です。
* **Cleartext-Target** は、TCP接続を使用するリモートインデクサーのアドレスとポートを定義します。 IPv4およびIPv6アドレスとホスト名がサポートされています。
* **Ciphertext-Target** は、TLS接続を使用するリモートインデクサーのアドレスとポートを定義します。 IPv4およびIPv6アドレスとホスト名がサポートされています。
* **Tag** は、DNS監査ログが割り当てられるタグを指定します。特殊文字やスペースを含まない任意の英数字値を使用できます。有効なタグ値が必要です。
* **Encoding** は、転送されるDNS監査ログの形式を指定します。オプションは_json_または_text_です。デフォルトは_json_です。
* **Insecure-Novalidate-TLS** は、TLS接続で証明書の検証を切り替えます。検証はデフォルトでオンになっています。
* **Log-Level** は、統合されたgravwellタグのロギングの詳細度を指定します。オプションは_OFF_ _INFO_ _WARN_ _ERROR_です。デフォルトは_ERROR_です。
* **Ingest-Cache-Path** は、インデクサーの接続が失われたときに関与するキャッシュシステムのファイルパスを指定します。パスは、書き込み可能なファイルへの絶対パスである必要があります。
* **Max-Cache-Size-MB** は、キャッシュファイルの最大サイズをメガバイト単位で指定します。これは安全ネットとして使用されます。ゼロ値がデフォルトであり、無制限を表します。
* **Cache-Depth** は、メモリ内インジェスト・バッファサイズをエントリ単位で指定します。デフォルトは128です。
* **Cache-Mode** は、インデクサーの接続状態に基づいてバッキングキャッシュの動作を指定します。デフォルトは"always"です。


基本的なGravwellの設定は次のようになります。

~~~
gravwell {
    Ingest-Secret IngestSecretToken
    Cleartext-Target 192.168.1.1:4023
    Tag dns
    Encoding json
    Log-Level INFO
    #Cleartext-Target 192.168.1.2:4023 #second indexer
    #Ciphertext-Target 192.168.1.1:4024
    #Insecure-Novalidate-TLS true #disable TLS certificate validation
    #Ingest-Cache-Path /tmp/coredns_ingest.cache #enable the local ingest cache
    #Max-Cache-Size-MB 1024
}
~~~

独自のGravwellプラグインセクションを各DNSリスナーに適用できます。 2つの異なるインターフェースをリッスンし、それぞれに固有のGravwell構成を適用するCorefileの例は次のようになります。

~~~
.:53 {
  forward . 8.8.8.8:53 8.8.4.4:53 9.9.9.9:53
  errors stdout
  bind 172.20.0.1
  cache 240
  whoami
  gravwell {
   Ingest-Secret SecretSetOne
   Cleartext-Target 172.20.0.1:4023
   Tag dns
   Encoding json
  }
}

.:53 {
  forward	. tls://1.1.1.1
  errors stdout
  bind 192.168.1.1
  hosts
  cache 60s
  gravwell {
   Ingest-Secret SecretSetTwo
   Cleartext-Target 192.168.1.100:4023
   Cleartext-Target 192.168.1.101:4023
   Cleartext-Target 192.168.1.102:4023
   Tag dns
   Encoding json
  }
}
~~~

各DNSリスナーを2つの完全に独立したGravwellインストールに送信していることに注意してください。1つは単一のインデクサー、もう1つは分散クラスターです。

暗号化されていない接続を介して単一のインデクサーにDNS要求を送信するGravwell Corefileセクションのサンプルです。ローカルキャッシュは無効です。

~~~
gravwell {
    Ingest-Secret IngestSecretToken
    Cleartext-Target 192.168.1.1:4023
    Tag dns
  }
~~~

TLS接続を介して2つのインデクサーにDNS要求を送信し、署名のない証明書を受け入れるGravwell Corefileセクションのサンプルです。 ローカルキャッシュは無効です。
IPv4およびIPv6アドレスは、クリアテキストおよび暗号文ターゲットの両方でサポートされています。IPv6アドレスは括弧で囲む必要があります。

~~~
gravwell {
    Ingest-Secret IngestSecretToken
    Ciphertext-Target 192.168.1.1:4024
    Ciphertext-Target [fe80::dead:beef:feed:febe%p1p1]:4024 #connecting to link local address via device p1p1
    Tag dns
    Encoding json
    Log-Level INFO
  }
~~~

TLS接続を介して2つのインデクサーにDNS要求を送信し、署名のない証明書を受け入れるGravwell Corefileセクションのサンプルです。 ローカルキャッシュは無効です。

~~~
gravwell {
    Ingest-Secret IngestSecretToken
    Ciphertext-Target 192.168.1.1:4024
    Ciphertext-Target [dead::beef]:4024
    Insecure-Novalidate-TLS true
    Tag dns
    Encoding json
    Log-Level INFO
  }
~~~

DNS要求を2つのインデクサーに送信し、インデクサー通信が失敗した場合にローカルキャッシュを有効にするGravwell Corefileセクションのサンプルです。 ローカルで最大1GBのデータをキャッシュできます。

~~~
gravwell {
    Ingest-Secret IngestSecretToken
    Cleartext-Target 192.168.1.1:4023
    Ciphertext-Target 192.168.1.2:4024
    Insecure-Novalidate-TLS true
    Ingest-Cache-Path /tmp/coredns_ingest.cache
    Max-Cache-Size-MB 1024
    Tag dns
    Encoding json
    Log-Level INFO
  }
~~~
