---
title: "运行验证人节点"
weight: 5
---

# 运行验证人节点

验证人负责生产和签署区块，并因此获得一部分区块奖励。搭建过程并不复杂，但这是一项运维承诺：
一旦验证人掉线或发生双签，协议就会对其进行惩罚。

本指南与主机商无关。任何满足下列要求的 Linux 服务器都可以使用。

## 运行验证人意味着什么

- **在线率。** 你的节点必须持续在线并保持同步。错过过多区块会被监禁（jail）。
- **密钥保管。** 你需要对验证人密钥负责。丢失即失去验证人；泄露则他人可用它签名。
- **版本升级。** 你必须按计划完成链的升级。
- **杜绝重复签名。** 同一个验证人密钥绝不能同时运行在两个节点上。

如果你无法承担这些责任，委托给现有验证人是更好的选择——参见[质押指南]({{< relref "staking.md" >}})。

---

## 硬件要求

| 组件 | 测试网（最低） | 主网建议 |
|---|---|---|
| CPU | 4 vCPU（x86-64） | 8 个独享核心，较新的 CPU |
| 内存 | 8 GB | 32 GB |
| 存储 | 100 GB NVMe SSD | 500 GB 以上 NVMe SSD |
| 网络 | 1 Gbps，不限流量 | 1 Gbps，不限流量 |
| 操作系统 | Ubuntu 22.04 LTS | Ubuntu 22.04 / 24.04 LTS |

优先选择 **NVMe** 而非 SATA SSD——共识对磁盘延迟很敏感，I/O 缓慢是错过区块最常见的原因。
主网请避免使用共享型/突发型 CPU 套餐。

---

## 如何选择主机

按重要性排序：

1. **独享（而非突发型）CPU** 以及 NVMe 存储
2. **不限或充裕的带宽**——验证人会持续进行网络广播
3. **稳定的网络与供电**，并有良好的在线记录
4. **选择尚不拥挤的地区**，避免与大量验证人扎堆

Cosmos 生态中常用的可靠服务商：

| 服务商 | 说明 |
|---|---|
| [Hetzner](https://www.hetzner.com/) | 性价比出色，提供欧洲与美国机房 |
| [OVHcloud](https://www.ovhcloud.com/) | 全球节点广泛，提供 VPS 与独立服务器 |
| [Latitude.sh](https://www.latitude.sh/) | 独立服务器，深受专业运维者欢迎 |
| [Vultr](https://www.vultr.com/) | 机房众多，提供 VPS 与独立服务器 |
| [DigitalOcean](https://www.digitalocean.com/) | 工具简洁，文档完善 |
| [Akamai（Linode）](https://www.linode.com/) | 老牌服务商，价格稳定 |

大型云厂商（AWS、Google Cloud、Azure）同样可用，但相同性能下成本明显更高。
主网更推荐使用**独立服务器**而非 VPS。

> [!NOTE]
> **关于去中心化：** 验证人分散在不同的服务商、国家和司法辖区，网络才更健康。
> 如果某个服务商或地区已经承载了验证人集合中的很大比例，建议另选其他。
> 可在[区块浏览器](https://explorer.ethwei.com/testnet)上查看当前的集中情况。

---

## 1. 准备服务器

连接服务器，并创建一个用于运行节点的非 root 用户：

```bash
adduser ethwei
usermod -aG sudo ethwei
```

为该用户配置 SSH 密钥登录，然后编辑 `/etc/ssh/sshd_config` 以加固 SSH 服务：

```
PermitRootLogin no
PasswordAuthentication no
```

```bash
sudo systemctl restart sshd
```

启用防火墙，只放行 SSH 与 P2P 端口：

```bash
sudo ufw allow 22/tcp
sudo ufw allow 26656/tcp
sudo ufw enable
```

> [!WARNING]
> **切勿**将验证人的 RPC（`26657`）、API（`1317`）或 gRPC（`9090`）端口暴露到公网。
> 请让它们仅监听本地回环地址。

切换到新用户，继续后续步骤：

```bash
su - ethwei
```

---

## 2. 安装依赖

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl jq build-essential
```

安装 Go（请到 [go.dev/dl](https://go.dev/dl) 查看当前版本）：

```bash
wget https://go.dev/dl/go1.23.4.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.23.4.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bashrc
source ~/.bashrc
go version
```

---

## 3. 编译二进制程序

```bash
git clone https://github.com/ethwei/ethwei-chain
cd ethwei-chain
make install
ethweid version
```

---

## 4. 初始化节点

moniker 是在区块浏览器上展示的公开名称。

```bash
ethweid init <your-moniker> --chain-id ethwei-testnet-1
```

获取创世文件：

```bash
curl -s https://rpc-testnet.ethwei.com/genesis | \
  jq '.result.genesis' > ~/.ethwei/config/genesis.json
```

---

## 5. 配置对等节点

编辑 `~/.ethwei/config/config.toml`：

```toml
[p2p]
seeds = ""              # 见网络信息页
persistent_peers = ""   # 见网络信息页
```

当前的种子与对等节点地址发布在[网络信息]({{< relref "network.md" >}})页面。

---

## 6. 配置节点

编辑 `~/.ethwei/config/app.toml`：

```toml
minimum-gas-prices = "0.025WEI"

# 控制验证人节点的磁盘占用
pruning = "custom"
pruning-keep-recent = "100"
pruning-interval = "10"
```

验证人无需对外提供查询服务，因此除非有特殊需要，请让 API 与 gRPC 服务保持关闭。

---

## 7. 以服务方式运行

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

---

## 8. 等待同步完成

查看进度：

```bash
ethweid status | jq '.SyncInfo'
```

等待 `catching_up` 变为 `false`。

> [!WARNING]
> 在节点完全同步之前，请不要创建验证人。未同步就进入活跃集的验证人会立即开始错过区块。

---

## 9. 创建并充值密钥

```bash
ethweid keys add validator
```

请将助记词离线保存——这是恢复该密钥的唯一途径。随后为该地址充值测试网 ETE。

---

## 10. 创建验证人

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

在[区块浏览器](https://explorer.ethwei.com/testnet)上确认你的验证人已出现：

```bash
ethweid query staking validator $(ethweid keys show validator --bech val -a)
```

---

## 运维

### 监控

至少应对以下情况告警：节点进程停止、`catching_up` 变为 true、漏块数上升、磁盘空间不足。
节点可暴露 Prometheus 指标——在 `config.toml` 中启用，并且只在内网抓取。

### 升级

```bash
cd ethwei-chain && git pull && make install
sudo systemctl restart ethweid
```

协调升级会在指定的区块高度进行。请关注公告频道，并在维护窗口内升级，不要提前。

### 备份

请备份以下两个文件并离线保存：

| 文件 | 用途 |
|---|---|
| `~/.ethwei/config/priv_validator_key.json` | 验证人的签名密钥 |
| `~/.ethwei/config/node_key.json` | 节点的 P2P 身份 |

---

## 安全

- **绝不要用同一份 `priv_validator_key.json` 运行两个节点。** 双签会被协议检测并罚没——
  这是运维者可能犯下的最具破坏性的错误。
- **关闭验证人的对外端口**，仅保留 P2P。主网请采用哨兵节点（sentry node）架构，
  使验证人不直接暴露在公网。
- **限制 SSH**，仅使用密钥认证，最好只允许已知地址访问。
- **主网可考虑使用远程签名器**（如 Horcrux 或 TMKMS），让签名密钥不必存放在验证人主机上。

> [!WARNING]
> **罚没（Slashing）：** 漏块过多的验证人会被监禁，双签的验证人会被罚没并永久移出验证人集合。
> 这两种惩罚同样会影响你的委托人。
