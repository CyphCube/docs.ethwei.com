---
title: "ウォレットと始め方"
weight: 2
---

# ウォレットと始め方

イーサウェイ は標準的な Cosmos ウォレットを使用します。[Keplr](https://www.keplr.app/) を推奨しており、
デスクトップのブラウザ拡張機能とモバイルアプリの両方が提供されています。

## 1. Keplr をインストール

**パソコンで** — Keplr 拡張機能をインストールしてください:
[Chrome / Brave](https://chrome.google.com/webstore/detail/keplr/dmkamcknogkgcdfhhbddcghachkejeap)
または [Firefox](https://addons.mozilla.org/firefox/addon/keplr/)。

**モバイルで** — Keplr アプリをインストールしてください:
[App Store](https://apps.apple.com/app/keplr/id1567851089)（iOS）または
[Google Play](https://play.google.com/store/apps/details?id=com.chainapsis.keplr)（Android）。

ウォレットを作成またはインポートし、シードフレーズをバックアップしてください。

## 2. イーサウェイ テストネットを追加

イーサウェイ テストネットを追加する方法は 2 つあります — やりやすい方を選んでください。

### 方法 A — Keplr 内で検索

**パソコンで** — Keplr 拡張機能を開き、上部のメニュー（☰）をクリックして **Add/Remove Chains** を選び、
**「Ethwei Testnet」** を検索してトグルをオンにします。

**モバイルで** — Keplr アプリを開き、メニュー（☰）をタップして **Add/Remove Chains**（チェーン表示の管理）
を選び、**「Ethwei Testnet」** を検索してトグルをオンにします。

![Keplr の Add/Remove Chains 画面で「Ethwei Testnet」を検索](/keplr-add-chain.png)

### 方法 B — ワンクリックボタン

**パソコンで** — 下のボタンをクリックしてください。Keplr がネットワークの承認を求めます。承認したら、
Keplr 内で **Ethwei Testnet** に切り替えてください。

**モバイルで** — Keplr アプリ内蔵ブラウザでこのページを開いてください（Keplr → ブラウザタブで
`docs.ethwei.com/ja/docs/wallet` を入力）。その後、下のボタンをタップします。モバイルの Safari と Chrome
は Keplr アプリと直接通信できないため、このボタンは Keplr 内蔵ブラウザでのみ動作します。

{{< add-to-keplr network="testnet" >}}

## 3. テストネットトークンを入手

テストネットのフォーセット（蛇口）: 準備中です。それまでは、コミュニティでテストネット ETE を
リクエストしてください。

## メインネット

{{< add-to-keplr network="mainnet" >}}

メインネットはまだ開始されていません。メインネットが利用可能になると、このボタンが有効になります。

---

## アドレス形式

すべての イーサウェイ アドレスは `ete` で始まります。例:

```
ete1qv2scq5r3qws0gu3h0e7h9c7m2t7h5m0zgm5xq
```

ネットワークを追加すると、イーサウェイ テストネットの Keplr アドレスが Keplr アイコンの下に表示されます。
