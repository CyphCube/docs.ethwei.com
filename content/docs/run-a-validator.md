---
title: "Run a Validator"
weight: 5
---

# Run a Validator

This guide covers setting up an Ethwei testnet validator on a Hetzner VPS. The same steps work
on any Linux VPS.

## Recommended hardware (testnet)

| Component | Minimum |
|---|---|
| CPU | 4 vCPU (x86-64) |
| RAM | 8 GB |
| Storage | 100 GB NVMe SSD |
| Network | 1 Gbps, unrestricted bandwidth |
| OS | Ubuntu 22.04 LTS |

A **Hetzner CX32** or **CPX31** covers this comfortably for testnet.

---

## 1. Provision the VPS

Create a Hetzner project, spin up a server with Ubuntu 22.04, and SSH in:

```bash
ssh root@<your-server-ip>
```

Create a non-root user:

```bash
adduser ethwei
usermod -aG sudo ethwei
su - ethwei
```

---

## 2. Install dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl jq build-essential
```

Install Go (check [go.dev/dl](https://go.dev/dl) for the latest version):

```bash
wget https://go.dev/dl/go1.23.4.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.23.4.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bashrc
source ~/.bashrc
go version
```

---

## 3. Build the chain binary

```bash
git clone https://github.com/ethwei/ethwei-chain
cd ethwei-chain
make install
ethweid version
```

---

## 4. Initialize the node

```bash
ethweid init <your-moniker> --chain-id ethwei-testnet-1
```

Download the genesis file:

```bash
curl -s https://rpc-testnet.ethwei.com/genesis | \
  jq '.result.genesis' > ~/.ethwei/config/genesis.json
```

---

## 5. Configure peers and seeds

Edit `~/.ethwei/config/config.toml`:

```toml
[p2p]
seeds = ""          # fill in once published
persistent_peers = ""   # fill in once published
```

Check the [Network Info]({{< relref "network.md" >}}) page for current seed / peer addresses.

---

## 6. Set minimum gas price

Edit `~/.ethwei/config/app.toml`:

```toml
minimum-gas-prices = "0.025WEI"
```

---

## 7. Create a systemd service

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

Check logs:

```bash
journalctl -u ethweid -f
```

Wait for the node to sync fully before creating the validator.

---

## 8. Create your validator key

```bash
ethweid keys add validator
```

Save the mnemonic somewhere safe. Fund your address with testnet ETE before continuing.

---

## 9. Create the validator

Once the node is synced:

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

Verify your validator appears on the [explorer](https://explorer.ethwei.com/testnet).

---

## Maintenance

### Check validator status

```bash
ethweid query staking validator $(ethweid keys show validator --bech val -a)
```

### Check sync status

```bash
ethweid status | jq '.SyncInfo'
```

### Update the binary

```bash
cd ethwei-chain && git pull && make install
sudo systemctl restart ethweid
```

> [!WARNING]
> **Slashing:** Validators that miss too many blocks or double-sign will be slashed and jailed.
> Keep your node online and never run duplicate signers from the same key.
