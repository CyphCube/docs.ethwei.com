---
title: "검증인 운영하기"
weight: 5
---

# 검증인 운영하기

이 가이드는 Hetzner VPS에서 Ethwei 테스트넷 검증인을 설정하는 방법을 다룹니다. 동일한 단계가 모든 Linux
VPS에서 작동합니다.

## 권장 하드웨어 (테스트넷)

| 구성 요소 | 최소 사양 |
|---|---|
| CPU | 4 vCPU (x86-64) |
| RAM | 8 GB |
| 스토리지 | 100 GB NVMe SSD |
| 네트워크 | 1 Gbps, 무제한 대역폭 |
| OS | Ubuntu 22.04 LTS |

**Hetzner CX32** 또는 **CPX31**이면 테스트넷에 충분합니다.

---

## 1. VPS 준비

Hetzner 프로젝트를 만들고 Ubuntu 22.04 서버를 생성한 후 SSH로 접속합니다:

```bash
ssh root@<your-server-ip>
```

비root 사용자를 생성합니다:

```bash
adduser ethwei
usermod -aG sudo ethwei
su - ethwei
```

---

## 2. 의존성 설치

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl jq build-essential
```

Go 설치([go.dev/dl](https://go.dev/dl)에서 최신 버전 확인):

```bash
wget https://go.dev/dl/go1.23.4.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.23.4.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bashrc
source ~/.bashrc
go version
```

---

## 3. 체인 바이너리 빌드

```bash
git clone https://github.com/ethwei/ethwei-chain
cd ethwei-chain
make install
ethweid version
```

---

## 4. 노드 초기화

```bash
ethweid init <your-moniker> --chain-id ethwei-testnet-1
```

제네시스 파일 다운로드:

```bash
curl -s https://rpc-testnet.ethwei.com/genesis | \
  jq '.result.genesis' > ~/.ethwei/config/genesis.json
```

---

## 5. 피어 및 시드 구성

`~/.ethwei/config/config.toml`을 편집합니다:

```toml
[p2p]
seeds = ""          # 공개되면 입력
persistent_peers = ""   # 공개되면 입력
```

현재 시드 / 피어 주소는 [네트워크 정보]({{< relref "network.md" >}}) 페이지에서 확인하세요.

---

## 6. 최소 가스 가격 설정

`~/.ethwei/config/app.toml`을 편집합니다:

```toml
minimum-gas-prices = "0.025WEI"
```

---

## 7. systemd 서비스 생성

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

로그 확인:

```bash
journalctl -u ethweid -f
```

검증인을 생성하기 전에 노드가 완전히 동기화될 때까지 기다리세요.

---

## 8. 검증인 키 생성

```bash
ethweid keys add validator
```

니모닉을 안전한 곳에 보관하세요. 계속하기 전에 주소에 테스트넷 ETE를 충전하세요.

---

## 9. 검증인 생성

노드가 동기화되면:

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

[익스플로러](https://explorer.ethwei.com/testnet)에서 검증인이 표시되는지 확인하세요.

---

## 유지 관리

### 검증인 상태 확인

```bash
ethweid query staking validator $(ethweid keys show validator --bech val -a)
```

### 동기화 상태 확인

```bash
ethweid status | jq '.SyncInfo'
```

### 바이너리 업데이트

```bash
cd ethwei-chain && git pull && make install
sudo systemctl restart ethweid
```

> [!WARNING]
> **슬래싱(Slashing):** 너무 많은 블록을 놓치거나 이중 서명하는 검증인은 슬래싱되어 감금(jail)됩니다.
> 노드를 항상 온라인으로 유지하고, 동일한 키로 중복 서명자를 절대 실행하지 마세요.
