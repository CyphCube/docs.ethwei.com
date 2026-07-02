---
title: "运行验证人节点"
weight: 5
---

# 运行验证人节点

本指南介绍如何在 Hetzner VPS 上搭建以太维测试网验证人。同样的步骤适用于任何 Linux VPS。

## 推荐硬件（测试网）

| 组件 | 最低要求 |
|---|---|
| CPU | 4 vCPU（x86-64） |
| 内存 | 8 GB |
| 存储 | 100 GB NVMe SSD |
| 网络 | 1 Gbps，不限流量 |
| 操作系统 | Ubuntu 22.04 LTS |

一台 **Hetzner CX32** 或 **CPX31** 足以从容满足测试网需求。

---

## 1. 开通 VPS

创建一个 Hetzner 项目，开一台 Ubuntu 22.04 的服务器，然后 SSH 登录：

```bash
ssh root@<your-server-ip>
```

创建一个非 root 用户：

```bash
adduser ethwei
usermod -aG sudo ethwei
su - ethwei
```

---

## 2. 安装依赖

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl jq build-essential
```

安装 Go（请到 [go.dev/dl](https://go.dev/dl) 查看最新版本）：

```bash
wget https://go.dev/dl/go1.23.4.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.23.4.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bashrc
source ~/.bashrc
go version
```

---

## 3. 编译链的二进制程序

```bash
git clone https://github.com/ethwei/ethwei-chain
cd ethwei-chain
make install
ethweid version
```

---

## 4. 初始化节点

```bash
ethweid init <your-moniker> --chain-id ethwei-testnet-1
```

下载创世文件：

```bash
curl -s https://rpc-testnet.ethwei.com/genesis | \
  jq '.result.genesis' > ~/.ethwei/config/genesis.json
```

---

## 5. 配置对等节点与种子节点

编辑 `~/.ethwei/config/config.toml`：

```toml
[p2p]
seeds = ""          # 公布后填写
persistent_peers = ""   # 公布后填写
```

请到[网络信息]({{< relref "network.md" >}})页面查看当前的种子 / 对等节点地址。

---

## 6. 设置最低 gas 价格

编辑 `~/.ethwei/config/app.toml`：

```toml
minimum-gas-prices = "0.025WEI"
```

---

## 7. 创建 systemd 服务

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

查看日志：

```bash
journalctl -u ethweid -f
```

在创建验证人之前，请等待节点完全同步。

---

## 8. 创建你的验证人密钥

```bash
ethweid keys add validator
```

请将助记词妥善保管。在继续之前，给你的地址充值一些测试网 ETE。

---

## 9. 创建验证人

节点同步完成后：

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

在[区块浏览器](https://explorer.ethwei.com/testnet)上确认你的验证人已出现。

---

## 维护

### 查看验证人状态

```bash
ethweid query staking validator $(ethweid keys show validator --bech val -a)
```

### 查看同步状态

```bash
ethweid status | jq '.SyncInfo'
```

### 更新二进制程序

```bash
cd ethwei-chain && git pull && make install
sudo systemctl restart ethweid
```

> [!WARNING]
> **罚没（Slashing）：** 错过过多区块或双签的验证人将被罚没并监禁。
> 请保持节点在线，切勿从同一个密钥运行重复的签名者。
