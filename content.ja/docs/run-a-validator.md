---
title: "バリデータを運用する"
weight: 5
---

# バリデータを運用する

バリデータはブロックを生成・署名し、その対価としてブロック報酬の一部を受け取ります。構築自体は難しく
ありませんが、運用上の責任を伴います。オフラインになったり二重署名を行ったバリデータは、プロトコルに
よってペナルティを受けます。

本ガイドは特定のホスティング事業者に依存しません。以下の要件を満たす Linux サーバーであれば利用できます。

## バリデータ運用に伴う責任

- **稼働率。** ノードは常にオンラインで同期を維持する必要があります。ブロックを多く逃すと監禁（jail）されます。
- **鍵の管理。** バリデータ鍵は自己責任です。紛失すればバリデータを失い、漏洩すれば第三者が署名できます。
- **アップグレード。** チェーンのアップグレードを予定どおり適用する必要があります。
- **重複署名の禁止。** 同一のバリデータ鍵を 2 つのノードで同時に実行してはいけません。

これらを担えない場合は、既存のバリデータに委任する方が適しています —
[ステーキングガイド]({{< relref "staking.md" >}})を参照してください。

---

## ハードウェア要件

| コンポーネント | テストネット（最小） | メインネット推奨 |
|---|---|---|
| CPU | 4 vCPU（x86-64） | 専有 8 コア、新しめの CPU |
| RAM | 8 GB | 32 GB |
| ストレージ | 100 GB NVMe SSD | 500 GB 以上の NVMe SSD |
| ネットワーク | 1 Gbps、無制限 | 1 Gbps、無制限 |
| OS | Ubuntu 22.04 LTS | Ubuntu 22.04 / 24.04 LTS |

SATA SSD より **NVMe** を優先してください — コンセンサスはディスク遅延に敏感で、I/O の遅さはブロック
欠落の最も一般的な原因です。メインネットでは共有型・バースト型 CPU プランは避けてください。

---

## ホスティングの選び方

重要な順に:

1. **専有（バースト型でない）CPU** と NVMe ストレージ
2. **無制限または十分な帯域** — バリデータは常時通信します
3. **安定したネットワークと電源**、および明確な稼働実績
4. **すでに混雑していない地域** を選ぶ

Cosmos エコシステムで広く使われている信頼できる事業者:

| 事業者 | 備考 |
|---|---|
| [Hetzner](https://www.hetzner.com/) | 優れたコストパフォーマンス、欧州 + 米国リージョン |
| [OVHcloud](https://www.ovhcloud.com/) | 幅広いグローバル拠点、VPS とベアメタル |
| [Latitude.sh](https://www.latitude.sh/) | ベアメタル、プロの運用者に人気 |
| [Vultr](https://www.vultr.com/) | 多数のリージョン、VPS とベアメタル |
| [DigitalOcean](https://www.digitalocean.com/) | シンプルなツール群と充実したドキュメント |
| [Akamai（Linode）](https://www.linode.com/) | 長い実績、予測しやすい料金 |

ハイパースケーラー（AWS、Google Cloud、Azure）でも問題なく動作しますが、同等の性能に対して費用は
大幅に高くなります。メインネットでは VPS より**ベアメタル**が望ましいです。

> [!NOTE]
> **分散化について:** バリデータが異なる事業者・国・法域に分散しているほど、ネットワークは健全になります。
> ある事業者や地域がすでにバリデータ集合の大きな割合を占めている場合は、別の選択肢を検討してください。
> 現在の偏りは[エクスプローラー](https://explorer.ethwei.com/testnet)で確認できます。

---

## 1. サーバーの準備

サーバーに接続し、ノードを実行する非 root ユーザーを作成します:

```bash
adduser ethwei
usermod -aG sudo ethwei
```

そのユーザー向けに SSH 鍵認証を設定し、`/etc/ssh/sshd_config` を編集して SSH デーモンを強化します:

```
PermitRootLogin no
PasswordAuthentication no
```

```bash
sudo systemctl restart sshd
```

ファイアウォールを有効にし、SSH と P2P ポートのみを許可します:

```bash
sudo ufw allow 22/tcp
sudo ufw allow 26656/tcp
sudo ufw enable
```

> [!WARNING]
> バリデータの RPC（`26657`）、API（`1317`）、gRPC（`9090`）ポートを公開インターネットに
> **公開しないでください。** localhost にバインドしたままにします。

以降の手順のため、新しいユーザーに切り替えます:

```bash
su - ethwei
```

---

## 2. 依存関係のインストール

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl jq build-essential
```

Go をインストール（現在のバージョンは [go.dev/dl](https://go.dev/dl) で確認）:

```bash
wget https://go.dev/dl/go1.23.4.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.23.4.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bashrc
source ~/.bashrc
go version
```

---

## 3. バイナリのビルド

```bash
git clone https://github.com/ethwei/ethwei-chain
cd ethwei-chain
make install
ethweid version
```

---

## 4. ノードの初期化

moniker はエクスプローラーに表示される公開名です。

```bash
ethweid init <your-moniker> --chain-id ethwei-testnet-1
```

ジェネシスファイルを取得します:

```bash
curl -s https://rpc-testnet.ethwei.com/genesis | \
  jq '.result.genesis' > ~/.ethwei/config/genesis.json
```

---

## 5. ピアの設定

`~/.ethwei/config/config.toml` を編集します:

```toml
[p2p]
seeds = ""              # ネットワーク情報を参照
persistent_peers = ""   # ネットワーク情報を参照
```

現在のシード / ピアアドレスは[ネットワーク情報]({{< relref "network.md" >}})ページで公開しています。

---

## 6. ノードの設定

`~/.ethwei/config/app.toml` を編集します:

```toml
minimum-gas-prices = "0.025WEI"

# バリデータのディスク使用量を抑える
pruning = "custom"
pruning-keep-recent = "100"
pruning-interval = "10"
```

バリデータはクエリを提供する必要がないため、特別な理由がない限り API と gRPC サービスは無効のままに
してください。

---

## 7. サービスとして実行

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

ログを確認します:

```bash
journalctl -u ethweid -f
```

---

## 8. 同期の完了を待つ

進捗を確認します:

```bash
ethweid status | jq '.SyncInfo'
```

`catching_up` が `false` になるまで待ちます。

> [!WARNING]
> ノードが完全に同期するまでバリデータを作成しないでください。同期前にアクティブセットへ入ったバリデータは、
> ただちにブロックを逃し始めます。

---

## 9. 鍵の作成と資金の準備

```bash
ethweid keys add validator
```

ニーモニックはオフラインで保管してください — この鍵を復元する唯一の手段です。その後、アドレスに
テストネット ETE をチャージします。

---

## 10. バリデータの作成

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
  --from=validator
```

[エクスプローラー](https://explorer.ethwei.com/testnet)にバリデータが表示されることを確認します:

```bash
ethweid query staking validator $(ethweid keys show validator --bech val -a)
```

---

## 運用

### モニタリング

最低限、次の項目でアラートを設定してください: ノードプロセスの停止、`catching_up` が true になる、
ブロック欠落の増加、ディスク空き容量の不足。ノードは Prometheus メトリクスを公開できます —
`config.toml` で有効化し、必ずプライベートネットワークからのみ収集してください。

### アップグレード

```bash
cd ethwei-chain && git pull && make install
sudo systemctl restart ethweid
```

協調アップグレードは指定されたブロック高で実施されます。アナウンスチャンネルを注視し、前倒しせず
メンテナンス時間内にアップグレードしてください。

### バックアップ

次の 2 つのファイルをバックアップし、オフラインで保管してください:

| ファイル | 用途 |
|---|---|
| `~/.ethwei/config/priv_validator_key.json` | バリデータの署名鍵 |
| `~/.ethwei/config/node_key.json` | ノードの P2P アイデンティティ |

---

## セキュリティ

- **同じ `priv_validator_key.json` で 2 つのノードを絶対に実行しないでください。** 二重署名は
  プロトコルによって検出されスラッシングされます — 運用者が犯しうる最も深刻な誤りです。
- **バリデータのポートは P2P 以外を閉じてください。** メインネットではセントリーノード構成を用い、
  バリデータが公開インターネットから直接到達できないようにします。
- **SSH を制限**し、鍵ベース認証のみとし、可能であれば既知のアドレスからのみ許可します。
- メインネットでは**リモート署名機**（Horcrux や TMKMS など）の利用を検討してください。署名鍵を
  バリデータのホストに置かずに済みます。

> [!WARNING]
> **スラッシング（Slashing）:** ブロックを過度に逃したバリデータは監禁され、二重署名したバリデータは
> スラッシングされ、セットから恒久的に除外されます。いずれのペナルティも委任者に影響します。
