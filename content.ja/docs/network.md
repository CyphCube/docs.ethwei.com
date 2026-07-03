---
title: "ネットワーク情報"
weight: 6
---

# ネットワーク情報

## テストネット

| パラメータ | 値 |
|---|---|
| Chain ID | `ethwei-testnet-1` |
| RPC | `https://rpc-testnet.ethwei.com` |
| REST / API | `https://api-testnet.ethwei.com` |
| エクスプローラー | [explorer.ethwei.com/testnet](https://explorer.ethwei.com/testnet) |
| gRPC | `grpc-testnet.ethwei.com:9090` |
| ピアポート | `26656` |
| RPC ポート | `26657` |

## メインネット

メインネットはまだ開始されていません。

---

## Keplr に追加

{{< add-to-keplr network="testnet" >}}

---

## チェーンパラメータ

| パラメータ | 値 |
|---|---|
| ボンド単位（Bond denom） | `WEI` |
| Bech32 プレフィックス | `ete` |
| コインタイプ（BIP44） | `118` |
| ブロック時間（目標） | 約 5–6 秒 |
| アンボンディング時間 | 未定（エクスプローラーで確認） |
| 最大バリデータ数 | 未定 |

## エンドポイント一覧

```
RPC   https://rpc-testnet.ethwei.com
API   https://api-testnet.ethwei.com
```

これらのエンドポイントは、テストネットの利便性のために Ethwei チームが運用しています。バリデータは、
独自の公開エンドポイントを提供することが推奨されます。
