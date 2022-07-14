---
title: "シンプルなDappを作成し、Ethereum Provider API経由でトランザクションを送信する"
emoji: "📒"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [web3, solidty]
published: true
---

# 本記事の対象者
* Web3技術に興味がある
* 独自通貨を作成してみたい

# 目次
* MetaMaskウォレットに接続する
* eth_accountsの結果を見る
* Dapps上でコインを送り合う

## 事前準備
1. MetaMask拡張機能をchromeに追加する
2. Node.jsをローカル環境にインストールされていること
3. 以下のプロダクトファイルをクローン
https://github.com/BboyAkers/simple-dapp-tutorial/tree/master/start

ファイルの構成は以下となります。
```
- index.html
- metamask.css
- contract.js
- favicon.ico
- package.json

## package.jsonは以下のように作成してください。
npm init

## package.jsonをインストールする
npm i
```

##　サーバーを起動する
今回はフロントのみでMetaMaskと通信を行いますので、http-serverを利用します。
```
## ローカルに入っていない場合は、以下のように実行
npm install -g http-server

## httpサーバーを起動
http-server
```

# MetaMaskウォレットに接続する
1. MetaMaskchrome拡張機能がインストールされているか確認する関数を準備する。
constranc.jsに記載されているinitialize関数を以下のように変更。
```
const initialize = () => {
  //index.htmlのconnectButtonを取得している
  const onboardButton = document.getElementById('connectButton');

  //拡張機能がインストールされているのか確認する
  const isMetaMaskInstalled = () => {
    const { ethereum } = window;
    return Boolean(ethereum && ethereum.isMetaMask);
  };

  const MetaMaskClientCheck = () => {
    //Now we check to see if MetaMask is installed
    if (!isMetaMaskInstalled()) {
      //ボタンのテキストを変更しているが、次でインストールするステップを加えます。
      onboardButton.innerText = 'Click here to install MetaMask!';
    } else {
      //インストールされている場合はボタンのテキストを変更する
      onboardButton.innerText = 'Connect';
    }
  };
  MetaMaskClientCheck();
};
```

2. MetaMaskがインストールされていない場合
  * click here to install MetaMaskをクリック
  * ボタンをクリックすると拡張機能をインストールするページに遷移する
  * ボタンを無効にする
```
const onboarding = new MetaMaskOnboarding({ forwarderOrigin });

const MetaMaskClientCheck = () => {
  if (!isMetaMaskInstalled()) {
    onboardButton.innerText = 'Click here to install MetaMask!';
    // インストールをクリックしたときに飛ばす
    onboardButton.onclick = onClickInstall;
    // インストールが完了したらボタンを無効化する
    onboardButton.disabled = true;
  } else {
    onboardButton.innerText = 'Connect';
  }
};

// onClickInstall関数を追加
const onClickInstall = () => {
  onboardButton.innerText = 'Onboarding in progress';
  onboardButton.disabled = true;
  onboarding.startOnboarding();
};
MetaMaskClientCheck();
```

3. MetaMaskがインストール済みの場合(MetaMaskに接続可能になります。)
  * Connectをクリック
  * ボタンをクリックするとMetamaskに接続する
  * ボタンを有効にする
```
const MetaMaskClientCheck = () => {
  if (!isMetaMaskInstalled()) {
    onboardButton.innerText = 'Click here to install MetaMask!';
    // インストールをクリックしたときに飛ばす
    onboardButton.onclick = onClickInstall;
    // インストールが完了したらボタンを無効化する
    onboardButton.disabled = true;
  } else {
    onboardButton.innerText = 'Connect';
    // インストールをクリックしたときに飛ばす
    onboardButton.onclick = onClickConnect;
    // インストールが完了したらボタンを有効化する
    onboardButton.disabled = false;
  }
};

## onClickConnect関数を追加する
const onClickConnect = async () => {
  try {
    // MetaMaskが開きます
    await ethereum.request({ method: 'eth_requestAccounts' });
  } catch (error) {
    console.error(error);
  }
};
MetaMaskClientCheck();
```
httpサーバーにアクセスして、MetaMaskにアクセスできているか確認する。
httpサーバーはキャッシュを持つので強制リセットを行う必要がある。

## イーサリアムアカウントを取得する
eth_acountsボタンを押すたびにイーサリアムアカウントを取得して、表示する。
initialize()関数に次の2つを加えます。
```
## eth_accountsボタンと表示する段落フィールドを取得
  const getAccountsButton = document.getElementById('getAccounts');
  const getAccountsResult = document.getElementById('getAccountsResult');

## 新たにMetaMaskClientCheck()関数に以下の関数を準備する
getAccountsButton.addEventListener('click', async () => {
  const accounts = await ethereum.request({ method: 'eth_accounts' });
  // 配列の最初の要素を取得します。
  getAccountsResult.innerHTML = accounts[0] || 'Not able to get accounts';
});
```

## トランザクションの送信を行う
Ethereumアカウト同士で、コインを送り合う。
```
## htmlにweb3.jsをインポートする
  <script src="node_modules/web3/dist/web3.min.js" defer></script>

## 以下のコードを追加し、ボタンを設置する
<section>
  <div class="row">
    <div class="col-lg-12">
      <button class="btn btn-danger" onclick="myFunction()">xxxxx</button>
    </div>
  </div>
</section>

## myFunctionを追加し、web3.jsで取得できていることを確認する。
const myFunction = async () => {
  const x = await web3.eth.getAccounts();
  console.log(x);
}

## web3.jsをコメントアウトし、webサイトからユーザーのEthereumアカウントへトランザクションを送る
const myFunction = async () => {
  // const x = await web3.eth.getAccounts();
  // console.log(x);
  
  window.ethereum
  .request({
    method: 'eth_sendTransaction',
    params: [
      {
        from: accounts[0],
        to: 'xxxxxxxxxxxxxxxxxxxxx', // 送りたいユーザーのwallet ID
        value: '0x29a2241af62c0000', // 16進数 3000000000000000000
        gasPrice: '0x09184e72a000',  // 16進数 10000000000000
        gas: '0x2710', // 16進数 10000
      },
    ],
  })
}
```
※ '0x':C言語その記法を受け継ぐ多くの言語をで16進数リテラルの表記に用いられる接続辞

# 参考
Ethereum Provider API
https://docs.metamask.io/guide/ethereum-provider.html#table-of-contents