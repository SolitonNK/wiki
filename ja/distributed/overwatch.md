# Gravwell オーバーウォッチ

いくつかの理由により、複数のGravwellインスタンスを別々に実行するのが有利な場合があります。マネージドセキュリティサービスプロバイダ(MSSP)は、データ分離とユーザー管理を簡単にするために、顧客ごとに Gravwell インデクサーとウェブサーバーインスタンスをセットアップする場合があるでしょう。しかし、顧客Aが顧客Bのデータにアクセスすることは望ましくなくとも、MSSPが顧客Aと顧客Bにまたがる検索文を同時に実行できることは有用です。

オーバーウォッチはこれを可能にします。オーバーウォッチウェブサーバーは、両方の顧客の*インデクサー*に接続し、すべてのデータを同時に検索することができます。それは、顧客クラスタからは完全に独立でのユーザー、リソース、スケジュールされた検索などの独自のセットを維持します。

![](overwatch.png)

上の画像はオーバーウォッチの設定例です。顧客AとBはそれぞれ4つのインデクサーと1つのウェブサーバーのセットを持っています。オーバーウォッチウェブサーバー（メガネ付きで右側に表示）は、顧客のウェブサーバーをバイパスしてインデクサーに直接接続しています。

## セキュリティメモ

外部クライアントのために複数のGravwellクラスタを稼働させる場合、特にオーバーウォッチを使用する場合には、以下のようにすることをお勧めします。

* クライアントがウェブサーバーやインデクサーに SSH でアクセスすることを許可すべきではありません。これは設定を壊すことを許容してしまう可能性があります。
* さらなるセキュリティのために、*クライアント*のGravwellクラスタはお互いにルーティングできないようにしてください。オーバーウォッチサーバーは各クラスタへのルーティングを許可しなければなりませんが、クラスター間での通信を許可してはいけません。

## ライセンスメモ

Gravwellのオーバーウォッチは、オーバーウォッチウェブサーバーに固有のライセンスを必要とします。オーバーウォッチ機能にもしご興味がおありでしたら、[セールス](mailto:sales@gravwell.io)までお問い合わせください。

オーバーウォッチウェブサーバーはクライアントノードにライセンスを配布しません。 つまり、オーバーウォッチウェブサーバーは Gravwell クラスタの初期セットアップには使用できません。 オーバーウォッチのライセンスはインデクサーでも使用できません。

## オーバーウォッチの設定

オーバーウォッチサーバーを設定する前に、*すべて*のインデクサーは同じ `Control-Auth`トークンを使用しなければならないことに注意してください。これによりオーバーウォッチサーバーはすべてのインデクサーに同時に接続することができます。

オーバーウォッチサーバーをインストールするには、Gravwell インストーラーを使ってウェブサーバーコンポーネントのみをインストールしてください。クライアントインデクサーで使用しているのと同じ `Control-Auth`を使用するように設定し、`Remote-Indexers`リストに *すべての*クライアントインデクサーを含めるように設定します。

次に、ウェブサーバーは**オーバーウォッチドメイン**で設定する必要があります。ドメインはgravwell.confの`Webserver-Domain`パラメータで設定します。クライアントのウェブサーバーはドメイン0のままでも問題ありませんが、オーバーウォッチのウェブサーバーは別の数値を設定しなければなりません。これは0から32767の間の任意の整数にすることができます。

以上がオーバーウォッチウェブサーバーのために必要な設定です。[TLS設定](#!configuration/certificates.md)や他のオプションを設定したいかもしれませんが、この時点でウェブサーバーを再起動(`systemctl restart gravwell_webserver.service`)して使用を開始しても安全です。

### 複数のオーバーウォッチサーバーを設定する

クライアント・インデクサーのすべてまたは一部のサブセットのいずれかに関連付けられた複数のオーバーウォッチ・システムを構成することが望ましいかもしれません。 MSSP は、特定のアナリストがクライアントのいくつかのサブセットで動作するように、顧客ベースをセグメント化する機能を望むかもしれません。 企業は、複数の組織に完全に独立したオーバーウォッチ・ウェブサーバーを提供したい場合があります。 オーバーウォッチシステムはドメイン構成パラメータで動作するため、複数のオーバーウォッチウェブサーバーを複数のドメインに構成することができます。

警告: 複数のオーバーウォッチウェブサーバーは、分散モードで動作するように設定されていない限り、別々のドメイン上に存在しなければなりません。複数のオーバーウォッチウェブサーバーが同じドメイン上に設定されている場合、リソースがインデクサー上で不適切に管理され、検索エラーが発生します。

![](OverwatchMutiple.png)

## オーバーウォッチサーバーの利用

オーバーウォッチウェブサーバーは、通常のGravwellウェブサーバーなので、通常のGravwellウェブサーバーと同じように使用できます。オーバーウォッチサーバーを使用して、誰かのActive Directoryサーバーがログのアップロードを停止していないか、すべてのクライアントのインジェストレートを監視することができますし、依頼に応じてインシデント調査に使うこともできます。ただし、オーバーウォッチウェブサーバー上で作成されたユーザーやリソースはクライアントクラスタ上で作成されたものと干渉しないものの、リソース集約的な検索を行うとクライアントクラスタ上の*インデクサー*リソースを消費しますし、すべてのクライアントで大規模な検索を行えば、何百ギガバイトもの結果がオーバーウォッチサーバーに送られることにもなりかねないため、それらのことを念頭に置いて使用してください。