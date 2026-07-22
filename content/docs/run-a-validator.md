---
title: "Run a Validator"
weight: 5
---

# Run a Validator

Validators produce and sign blocks, and in return earn a share of block rewards. Running one is
straightforward, but it is an operational commitment: a validator that goes offline or signs
twice will be penalised by the protocol.

This guide is host-agnostic. Any Linux server that meets the requirements below will work.

## What running a validator involves

- **Uptime.** Your node must stay online and in sync. Missing too many blocks leads to jailing.
- **Key custody.** You are responsible for your validator key. Lose it and you lose the validator;
  leak it and someone else can sign with it.
- **Upgrades.** You must apply chain upgrades on schedule.
- **No duplicate signing.** The same validator key must never run on two nodes at once.

If you cannot commit to those, delegating to an existing validator is the better option — see the
[Staking guide]({{< relref "staking.md" >}}).

---

## Hardware requirements

| Component | Testnet (minimum) | Recommended for mainnet |
|---|---|---|
| CPU | 4 vCPU (x86-64) | 8 dedicated cores, modern CPU |
| RAM | 8 GB | 32 GB |
| Storage | 100 GB NVMe SSD | 500 GB+ NVMe SSD |
| Network | 1 Gbps, unmetered | 1 Gbps, unmetered |
| OS | Ubuntu 22.04 LTS | Ubuntu 22.04 / 24.04 LTS |

Prefer **NVMe** over SATA SSDs — consensus is sensitive to disk latency, and slow I/O is the most
common cause of missed blocks. Avoid shared/burstable CPU plans for mainnet.

---

## Choosing a host

What matters, in order:

1. **Dedicated (not burstable) CPU** and NVMe storage
2. **Unmetered or generous bandwidth** — a validator gossips continuously
3. **Reliable network and power**, with a clear uptime record
4. **A region that isn't already crowded** with other validators

Reputable providers used across the Cosmos ecosystem:

| Provider | Notes |
|---|---|
| [Hetzner](https://www.hetzner.com/) | Excellent price/performance, EU + US regions |
| [OVHcloud](https://www.ovhcloud.com/) | Wide global footprint, VPS and bare metal |
| [Latitude.sh](https://www.latitude.sh/) | Bare metal, popular with professional operators |
| [Vultr](https://www.vultr.com/) | Many regions, VPS and bare metal |
| [DigitalOcean](https://www.digitalocean.com/) | Simple tooling, good documentation |
| [Akamai (Linode)](https://www.linode.com/) | Long-established, predictable pricing |

Hyperscalers (AWS, Google Cloud, Azure) work fine but cost significantly more for the same
performance. **Bare metal** is preferable to a VPS for mainnet.

> [!NOTE]
> **On decentralisation:** the network is healthier when validators are spread across different
> providers, countries, and jurisdictions. If a provider or region already hosts a large share of
> the validator set, consider choosing another. Check the
> [explorer](https://explorer.ethwei.com/testnet) to see where the set is concentrated.

---

## 1. Prepare the server

Connect to your server and create a non-root user to run the node:

```bash
adduser ethwei
usermod -aG sudo ethwei
```

Set up SSH key authentication for that user, then harden the SSH daemon by editing
`/etc/ssh/sshd_config`:

```
PermitRootLogin no
PasswordAuthentication no
```

```bash
sudo systemctl restart sshd
```

Enable a firewall, allowing only SSH and the peer-to-peer port:

```bash
sudo ufw allow 22/tcp
sudo ufw allow 26656/tcp
sudo ufw enable
```

> [!WARNING]
> Do **not** expose the RPC (`26657`), API (`1317`), or gRPC (`9090`) ports to the public internet
> on a validator. Keep them bound to localhost.

Switch to the new user for the remaining steps:

```bash
su - ethwei
```

---

## 2. Install dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl jq build-essential
```

Install Go (check [go.dev/dl](https://go.dev/dl) for the current version):

```bash
wget https://go.dev/dl/go1.23.4.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.23.4.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bashrc
source ~/.bashrc
go version
```

---

## 3. Build the binary

```bash
git clone https://github.com/ethwei/ethwei-chain
cd ethwei-chain
make install
ethweid version
```

---

## 4. Initialise the node

Your moniker is the public name shown on the explorer.

```bash
ethweid init <your-moniker> --chain-id ethwei-testnet-1
```

Fetch the genesis file:

```bash
curl -s https://rpc-testnet.ethwei.com/genesis | \
  jq '.result.genesis' > ~/.ethwei/config/genesis.json
```

---

## 5. Configure peers

Edit `~/.ethwei/config/config.toml`:

```toml
[p2p]
seeds = ""              # see Network Info
persistent_peers = ""   # see Network Info
```

Current seed and peer addresses are published on the
[Network Info]({{< relref "network.md" >}}) page.

---

## 6. Configure the node

Edit `~/.ethwei/config/app.toml`:

```toml
minimum-gas-prices = "0.025WEI"

# Keep disk usage under control on a validator
pruning = "custom"
pruning-keep-recent = "100"
pruning-interval = "10"
```

Validators do not need to serve queries, so leave the API and gRPC services disabled unless you
have a specific reason to enable them.

---

## 7. Run as a service

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

Follow the logs:

```bash
journalctl -u ethweid -f
```

---

## 8. Wait for sync

Check progress:

```bash
ethweid status | jq '.SyncInfo'
```

Wait until `catching_up` is `false`.

> [!WARNING]
> Do not create your validator until the node is fully synced. A validator that enters the active
> set before syncing will immediately start missing blocks.

---

## 9. Create and fund your key

```bash
ethweid keys add validator
```

Store the mnemonic offline — it is the only way to recover this key. Then fund the address with
testnet ETE.

---

## 10. Create the validator

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

Confirm your validator appears on the [explorer](https://explorer.ethwei.com/testnet):

```bash
ethweid query staking validator $(ethweid keys show validator --bech val -a)
```

---

## Operations

### Monitoring

At minimum, alert on: node process down, `catching_up` becoming true, missed blocks rising, and
low disk space. The node exposes Prometheus metrics — enable them in `config.toml` and scrape
from a private network only.

### Upgrading

```bash
cd ethwei-chain && git pull && make install
sudo systemctl restart ethweid
```

Coordinated upgrades happen at a specified block height. Watch the announcement channels and
upgrade during the maintenance window, not before.

### Backups

Back up these two files and keep them offline:

| File | Purpose |
|---|---|
| `~/.ethwei/config/priv_validator_key.json` | Your validator's signing key |
| `~/.ethwei/config/node_key.json` | Your node's p2p identity |

---

## Security

- **Never run two nodes with the same `priv_validator_key.json`.** Double-signing is detected by
  the protocol and slashed — this is the single most damaging mistake an operator can make.
- **Keep the validator's ports closed** except p2p. Use a sentry-node architecture for mainnet so
  the validator is not directly reachable from the public internet.
- **Restrict SSH** to key-based authentication, ideally from known addresses only.
- **Consider a remote signer** (such as Horcrux or TMKMS) for mainnet, so the signing key never
  lives on the validator host.

> [!WARNING]
> **Slashing:** validators that miss too many blocks are jailed, and validators that double-sign
> are slashed and permanently removed from the set. Both penalties also affect your delegators.
