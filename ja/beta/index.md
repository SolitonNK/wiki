＃Gravwellベータプログラム

ご挨拶

Gravwell Betaの早期アクセスグループに所属しているため、このメッセージを読んでいます。皆様のご参加に心より感謝申し上げますとともに、フィードバックやバグレポートをお待ちしております。

バグやフィードバックを送信してください：

* https://www.gravwell.io/beta
* beta@gravwell.io までメールでお問い合わせください。

ありがとうございます！

## 現在の "ベータ版の状態"

### 希望するテスト

このスプリントのテスト希望（優先度順）

* キットの閲覧、インストール、管理
* 検索テンプレートの作成と「調査」ダッシュボードの作成に使用します。
* プレイブック - プレイブックをもう少し豊かにするための新機能が来ていますが、マークダウンREADMEとして作成して使用するテストは、このフェーズのための願望です。
* マクロ
* テンプレート


## インストールとアップグレード

このビルドが利用可能になり、テストできるようになったことを大変うれしく思います。新しいubuntuリポジトリとDockerイメージを作成しました。StableからBetaへの切り替えは、aptソースリポジトリ（または最初からインストールする場合はクイックスタート手順）を変更することで実行されます。

### アップグレード：
`/etc/apt/sources.list.d/gravwell.list`ファイルを編集し、`https://update.gravwell.io/debian/`の代わりに`https://update.gravwell.io/debianbeta/`を使用します。次に、`apt update`と`apt upgrade`を実行すると、新しいリリースになります。

### ゼロからのインストールです。

```
curl https://update.gravwell.io/debian/update.gravwell.io.gpg.key | sudo apt-key add -
echo 'deb [ arch=amd64 ] https://update.gravwell.io/debianbeta/ community main' | sudo tee /etc/apt/sources.list.d/gravwell.list
sudo apt-get install apt-transport-https
sudo apt-get update
sudo apt-get install gravwell
```

### Dockerを使用しています。

Dockerイメージは[gravwell/beta](https://hub.docker.com/r/gravwell/beta)にあります。dockerのドキュメントで`gravwell/gravwell`を`gravwell/beta`に置き換えれば、"ただ動く"はずです。




## ありがとうございます

Gravwellがもたらす新しい機能にとても興奮しています。ベータプログラムに興味を持っていただき、ご参加いただきありがとうございました。あなたなしではできませんでした。

フィードバックやバグレポート、特に新しいツールを使って作ったクールなものを見せてください!
