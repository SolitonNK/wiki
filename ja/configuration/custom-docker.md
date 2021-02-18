# カスタムDockerデプロイメント

注:本ページはSoliton NKの開発元Gravwell社のマニュアルを翻訳したものです。現在、Soliton NK用のDockerfileの提供は行っておりません。

ほとんどのGravwellコンポーネントは、最新のLinuxホストでの実行に適した静的にコンパイルされた実行可能ファイルとしてデプロイされます。これにより、Dockerでのデプロイも容易になります。 Gravwellのエンジニアは、開発、テスト、内部展開にDockerを幅広く使用しています。 また、Dockerを使用することによって、大規模なGravwellシステムの迅速な、起動、停止、管理、を可能にします。 お客様がDockerでのデプロイを迅速に実行できるようにするために、クラスター構成とシングル構成両方のエディションのSKUに対応したDockerfileのサンプルを提供しています。

Dockerfileの完全なセットは [こちら](https://update.gravwell.io/files/docker_buildfiles_ad05723a547d31ee57ed8880bb5ef4e9.tar.bz2) にあります。MD5のチェックサムはad5723a547d31ee57ed8880bb5ef4e9です。

## Dockerコンテナーの構築

提供されたDockerfileを使用してDockerコンテナーを構築するのは非常に簡単です。GravwellのDockerデプロイメントは、非常に小さなbusyboxベースコンテナーを利用して、非常に小さなコンテナを実現しています。

### Dockerfile

特定のデプロイ要件に合わせてDockerfileを変更するのに必要な作業はほとんどありません。 標準のgravwell Dockerfileは、小さな起動スクリプトを使用して、起動時にX509証明書を再生成する必要があるかどうかを確認します。 もし既に有効な証明書がある場合には、インストーラーによって/opt/gravwell/binにデプロイされるユーティリティ（gencertおよびcrashreport）を削除して、Gravwellバイナリを直接起動するようにDockerfileを変更することができます。

単一インスタンスのベースDockerfile：
```
FROM busybox
MAINTAINER support@gravwell.io
ARG INSTALLER=gravwell_installer.sh
COPY $INSTALLER /tmp/installer.sh
COPY start.sh /tmp/start.sh
RUN /bin/sh /tmp/installer.sh --no-questions
RUN rm -f /tmp/installer.sh
RUN mv /tmp/start.sh /opt/gravwell/bin/
CMD ["/bin/sh", "/opt/gravwell/bin/start.sh"]
```

基本的な起動スクリプト：
```
#!/bin/sh

# unless environment variable says no, generate new SSL certs
if [ "$NO_SSL_GEN" == "true" ]; then
	echo "Skipping SSL certificate generation"
else
	/opt/gravwell/bin/gencert -key-file /opt/gravwell/etc/key.pem -cert-file /opt/gravwell/etc/cert.pem -host 127.0.0.1
	if [ "$?" != "0" ]; then
		echo "Failed to generate certificates"
		exit -1
	fi
fi

#fire up the indexer and webserver processes and wait
/opt/gravwell/bin/gravwell_indexer -config-override /opt/gravwell/etc/ &
/opt/gravwell/bin/gravwell_webserver -config-override /opt/gravwell/etc/ &
wait
```

## 環境変数を使用する

標準のdocker起動スクリプトは環境変数 `NO_SSL_GEN`を探し、「true」に設定されている場合はX509証明書の生成をスキップします。 もしデプロイメントに有効な証明書が含まれている場合、コンテナを起動するときに引数 `-e NO_SSL_GEN = true`を必ず含めてください。

インデクサー、Webサーバー、およびインジェスターは、必要に応じて、構成ファイルではなく、環境変数を介して設定されるいくつかの構成パラメーターを持つこともできます。 詳細については、[環境変数](environment-variables.md)のドキュメントを参照してください。

## インジェスター用のサンプルDockerfile

Gravwellは新しいインジェスターとコンポーネントを継続的にリリースしていますが、常に各インジェスターに対応するDockerfileを提供しているとは限りません。 ただし、Dockerfileは非常に単純であり、簡単に変更できます。 以下は、SimpleRelay ingesterを介してDockerコンテナーを作成するDockerfileの例です。

```
FROM busybox
MAINTAINER support@gravwell.io
ARG INSTALLER=gravwell_installer.sh
COPY $INSTALLER /tmp/installer.sh
RUN /bin/sh /tmp/installer.sh --no-questions
RUN rm -f /tmp/installer.sh
CMD ["/opt/gravwell/bin/gravwell_simple_relay"]
```

コンテナを構築するには、単純なリレーインストーラーをDockerファイルと同じ作業ディレクトリにコピーし、次のコマンドを実行します。
```
docker build --ulimit nofile=32000:32000 --compress --build-arg INSTALLER=gravwell_simple_relay_installer_2.0.sh --no-cache --tag gravwell:simple_relay_2.0.0 .
```
