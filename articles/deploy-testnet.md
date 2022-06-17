---
title: "truffleを使ってpolygon(testnet)へコントラクトをデプロイする"
emoji: "🗒"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [web3, solidty]
published: true
---

# 本記事の対象者
* Web3技術に興味がある
* 独自通貨を作成してみたい

# 本記事のリポジトリ
あとで追記します
ぜひ参考にしてください！

# Torus Walletへテストコインを送信
Block chainのpolygonに対して、独自のコントラクトを作成したりトランザクションを利用する際に、テスト用のNetworkに接続したいことがあります。
テスト用でも本番環境と同様に手数料が発生するので、事前に資金トークンをfaucetからwallet(今回はTorus Walletを使用)に入れておく必要があります。

## Polygonとは？
Ethereumと互換性のあるブロックチェーンを簡単に構築し、Ethereumやその他のブロックチェーンネットワークと相互運用するためのプロトコルとフレームワークを提供するサービス。

## Torus Walletとは？
ノンカストディアル型のwebベースのウォレットです。
Torus Walletは分散鍵生成プロトコルを使用し、多数のSNSログイン認証を使い利便性を向上させ、秘密鍵を中央集権しない仕組みです。
他のウォレット同様にアドレスで暗号資産を送ることができ、ENSドメインやSNSアカウント指定によってName Resolverという仕組みで暗号資産を送ることができます。

カストディアル型とノンカストディアル型については下記に記載いたします。
- カストディアル型
第三者によるログインや暗号資産アカウントの管理します。自分の秘密鍵に自分でアクセスすることはできません。

- ノンカストディアル型
取引先やサービス提供企業といった中央管理組織ではなく、ユーザー自身がウォレットの秘密鍵を管理することができます。
管理しているのはユーザーのため秘密鍵を噴出してしまったり、バックアップ用のシードフレーズを思い出せなかったりすると、ウォレットとその資金は失われてしまいます。

①TORUS walletへアクセス
※始めたばかりの人はトークンがありません。
![](https://i.gyazo.com/5ba109204dd59d8940c527b30b9c233a.png)
https://toruswallet.io/

②設定からmunbaiネットワークに変更

③Wallletへテストコイン払い出し
1. Torus Wallet内の共有鍵をコピー
2. polygon Faucetからテストコイン払い出し
![](https://i.gyazo.com/b357e43c21d944ad72192dc9befd9f89.png)

https://faucet.polygon.technology/
Netwaork:Munbai
Select Token:MATIC Token
Wallet Address:手順1でコピーしたもの
3. Submit→Comfirm
数分待つとTorus Walletへ反映される

# polygonのTestnet(mumbai)へトークンをデプロイする
①migrations内ファイルのinit supplyを設定する

②truffle.config.jsにtestnet接続用の情報を追記する
infura経由でデプロイしている。
```
// testnet
mumbai: {
    provider: () => new HDWalletProvider(mnemonic, `https://polygon-mumbai.infura.io/v3/81984a3dbd7d4446886a8add0f51aa79`),
    network_id: 80001,
    confirmations: 2,
    // networkCheckTimeout: 10000,
    timeoutBlocks: 2000,
    skipDryRun: true
}
confirmationsはデプロイする際に確認する回数を指します。
```

③truffle.config.js内の以下をコメントアウト外す
```
const HDWalletProvider = require('@truffle/hdwallet-provider');

const fs = require('fs');
const mnemonic = fs.readFileSync(".secret").toString().trim();
```

④HDWalletProviderとfsをプロジェクトにインストールする
HDWalletProviderとは?
fsとは?
```
npm i @truffle/hdwallet-provider fs
```

⑤nmemonicにて利用している.secretファイルを作成する
記載する内容はWalletの秘密鍵です。Torus Walletを利用しているので、設定の中のアカウント詳細より閲覧することが可能です。
nmemonicでデプロイを行うこともできますが、今回は秘密鍵を使って行います。(Arrayでも可能ですが、今回はstringでも可能。)
![](https://i.gyazo.com/7ee981c05b94f4134dc94d8d17065a83.png)

⑥mumbaiへデプロイ
```
truffle deploy --network mumbai
```
失敗した場合は、truffle.config.jsのtimeoutBlocksを伸ばします。
![](https://i.gyazo.com/e840c31b7c5d2228654e83ff6cb36d73.png)

## polygonのtestnetにデプロイされているのか確認
①以下にアクセスし、デプロイした際のcontract addressを検索する。
https://mumbai.polygonscan.com/

②自分のWalletアドレスとpolygon上のwalletアドレスが同じか確認する
wallet側が一部大文字、polygon側が一部小文字になっていると思います。
こちらはEthereumのアドレスが16進数で表記されるので、本来、大文字小文字を区別する必要はありません。
しかし、アドレスの打ち間違いや欠損を区別するためにチェックサムとして、英字部分にする処理を施すことができるためwalletの方では大文字で分かれているようです。

## Torus Wallet内にトークンを追加する
①画面下部の「ここにトークンの追加」
②「トークン契約アドレス」へcontract address
![](https://i.gyazo.com/e840c31b7c5d2228654e83ff6cb36d73.png)

## 作成したトークン残高とwallet内に入っている残高を取得する
①以下のコードを準備
``` test.js
const Web3 = require('web3');

const ubcoinJson = require('./build/contracts/[deployした際に作成されたファイル]');

const web3 = new Web3(
  new Web3.providers.HttpProvider(`https://polygon-mumbai.infura.io/v3/[infuraendpoint]`),
);
//testnet mainnet
const ubContractAddresss = ''; // 永続的

const WalletAddress = ''; // My address
const toWallet = ''; /infuraのエンドポイント

const contract = new web3.eth.Contract(ubcoinJson.abi, ubContractAddresss);
(async () => {
  const balanceUB = await contract.methods.balanceOf(WalletAddress).call();
  const balance = await web3.eth.getBalance(WalletAddress);
  console.log('🚀 ~ file: test.js ~ line 20 ~ balanceUB', balanceUB);
  console.log('🚀 ~ file: test.js ~ line 22 ~ balance', balance);
})();
```

balanceOf(account):各アドレスのトークン残高を取得でする関数
getBalance(account):アカウントの残高を取得する関数

②web3をプロジェクトにインストール
```
npm i web3
```
③コマンドラインで以下を実行し、ログが表示されれば成功
```
node ファイル名.js
```

# 参考
試して学ぶ　スマートコントラクト開発
加嵜 長門  (著), 篠原 航  (著), 金 志京  (著), 河西 紀明  (著), 田中 克典  (著), 佐々木 亮彰 (著), 平野 浩司 (著), 前川 彰 (著), DMM.comブロックチェーン研究室 (著)