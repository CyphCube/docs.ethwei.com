---
title: "バリデータを運用する"
weight: 5
---

# バリデータを運用する

このガイドでは、Hetzner VPS 上に Ethwei テストネットのバリデータを構築する方法を説明します。同じ手順は
あらゆる Linux VPS で動作します。

## 推奨ハードウェア（テストネット）

| コンポーネント | 最小要件 |
|---|---|
| CPU | 4 vCPU（x86-64） |
| RAM | 8 GB |
| ストレージ | 100 GB NVMe SSD |
| ネットワーク | 1 Gbps、帯域無制限 |
| OS | Ubuntu 22.04 LTS |

**Hetzner CX32** または **CPX31** であれば、テストネットには余裕をもって対応できます。

---

## 1. VPS を用意する

Hetzner プロジェクトを作成し、Ubuntu 22.04 のサーバーを立ち上げて SSH で接続します:

```bash
ssh root@<your-server-ip>
```

非 root ユーザーを作成します:

```bash
adduser ethwei
usermod -aG sudo ethwei
su - ethwei
```

---

## 2. 依存関係をインストール

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl jq build-essential
```

Go をインストール（最新バージョンは [go.dev/dl](https://go.dev/dl) で確認）:

```bash
wget https://go.dev/dl/go1.23.4.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.23.4.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bashrc
source ~/.bashrc
go version
```

---

## 3. チェーンのバイナリをビルド

```bash
git clone https://github.com/ethwei/ethwei-chain
cd ethwei-chain
make install
ethweid version
```

---

## 4. ノードを初期化

```bash
ethweid init <your-moniker> --chain-id ethwei-testnet-1
```

ジェネシスファイルをダウンロード:

```bash
curl -s https://rpc-testnet.ethwei.com/genesis | \
  jq '.result.genesis' > ~/.ethwei/config/genesis.json
```

---

## 5. ピアとシードを設定

`~/.ethwei/config/config.toml` を編集します:

```toml
[p2p]
seeds = ""          # 公開され次第入力
persistent_peers = ""   # 公開され次第入力
```

現在のシード / ピアアドレスは[ネットワーク情報]({{< relref "network.md" >}})ページで確認してください。

---

## 6. 最小ガス価格を設定

`~/.ethwei/config/app.toml` を編集します:

```toml
minimum-gas-prices = "0.025WEI"
```

---

## 7. systemd サービスを作成

```bash
sudo tee /etc/systemd/system/ethweid.service > /dev/null <<EOF
[Unit]
Description=Ethwei Node
After=network-online.target

[Service]
User=ethwei
ExecStart=$(which ethweid) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable ethweid
sudo systemctl start ethweid
```

ログを確認:

```bash
journalctl -u ethweid -f
```

バリデータを作成する前に、ノードが完全に同期するまで待ってください。

---

## 8. バリデータキーを作成

```bash
ethweid keys add validator
```

ニーモニックを安全な場所に保管してください。続行する前に、アドレスにテストネット ETE をチャージします。

---

## 9. バリデータを作成

ノードが同期したら:

```bash
ethweid tx staking create-validator \
  --amount=1000000WEI \
  --pubkey=$(ethweid tendermint show-validator) \
  --moniker="<your-moniker>" \
  --chain-id=ethwei-testnet-1 \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --gas="auto" \
  --gas-adjustment=1.5 \
  --fees="5000WEI" \
  --from=validator \
  --node=https://rpc-testnet.ethwei.com:443
```

[エクスプローラー](https://explorer.ethwei.com/testnet)でバリデータが表示されることを確認してください。

---

## メンテナンス

### バリデータの状態を確認

```bash
ethweid query staking validator $(ethweid keys show validator --bech val -a)
```

### 同期状態を確認

```bash
ethweid status | jq '.SyncInfo'
```

### バイナリを更新

```bash
cd ethwei-chain && git pull && make install
sudo systemctl restart ethweid
```

> [!WARNING]
> **スラッシング（Slashing）:** ブロックを過度に見逃したり二重署名したバリデータは、スラッシングされ
> 監禁（jail）されます。ノードを常にオンラインに保ち、同じキーで重複した署名者を絶対に実行しないでください。
