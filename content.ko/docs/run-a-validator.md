---
title: "검증인 운영하기"
weight: 5
---

# 검증인 운영하기

검증인은 블록을 생성하고 서명하며, 그 대가로 블록 보상의 일부를 받습니다. 구축 자체는 어렵지 않지만
운영에 대한 책임이 따릅니다. 검증인이 오프라인이 되거나 이중 서명을 하면 프로토콜이 불이익을 부과합니다.

이 가이드는 특정 호스팅 업체에 종속되지 않습니다. 아래 요구 사항을 충족하는 Linux 서버라면 무엇이든
사용할 수 있습니다.

## 검증인 운영에 따르는 책임

- **가동 시간.** 노드는 항상 온라인 상태로 동기화를 유지해야 합니다. 블록을 너무 많이 놓치면 감금(jail)됩니다.
- **키 관리.** 검증인 키는 전적으로 본인 책임입니다. 분실하면 검증인을 잃고, 유출되면 타인이 서명할 수 있습니다.
- **업그레이드.** 체인 업그레이드를 일정에 맞춰 적용해야 합니다.
- **중복 서명 금지.** 동일한 검증인 키를 두 노드에서 동시에 실행해서는 절대 안 됩니다.

이러한 책임을 감당하기 어렵다면, 기존 검증인에게 위임하는 편이 더 좋습니다 —
[스테이킹 가이드]({{< relref "staking.md" >}})를 참고하세요.

---

## 하드웨어 요구 사항

| 구성 요소 | 테스트넷(최소) | 메인넷 권장 |
|---|---|---|
| CPU | 4 vCPU (x86-64) | 전용 코어 8개, 최신 CPU |
| RAM | 8 GB | 32 GB |
| 스토리지 | 100 GB NVMe SSD | 500 GB 이상 NVMe SSD |
| 네트워크 | 1 Gbps, 무제한 | 1 Gbps, 무제한 |
| OS | Ubuntu 22.04 LTS | Ubuntu 22.04 / 24.04 LTS |

SATA SSD보다 **NVMe**를 우선하세요 — 합의는 디스크 지연에 민감하며, 느린 I/O는 블록 누락의 가장 흔한
원인입니다. 메인넷에서는 공유형·버스트형 CPU 요금제를 피하세요.

---

## 호스팅 선택하기

중요한 순서대로:

1. **전용(버스트형이 아닌) CPU**와 NVMe 스토리지
2. **무제한 또는 넉넉한 대역폭** — 검증인은 지속적으로 통신합니다
3. **안정적인 네트워크와 전원**, 그리고 검증된 가동 이력
4. **이미 붐비지 않는 지역** 선택

Cosmos 생태계에서 널리 쓰이는 신뢰할 만한 업체:

| 업체 | 비고 |
|---|---|
| [Hetzner](https://www.hetzner.com/) | 뛰어난 가격 대비 성능, 유럽 + 미국 리전 |
| [OVHcloud](https://www.ovhcloud.com/) | 폭넓은 글로벌 인프라, VPS 및 베어메탈 |
| [Latitude.sh](https://www.latitude.sh/) | 베어메탈, 전문 운영자들에게 인기 |
| [Vultr](https://www.vultr.com/) | 다양한 리전, VPS 및 베어메탈 |
| [DigitalOcean](https://www.digitalocean.com/) | 간결한 도구, 우수한 문서 |
| [Akamai(Linode)](https://www.linode.com/) | 오랜 업력, 예측 가능한 요금 |

하이퍼스케일러(AWS, Google Cloud, Azure)도 문제없이 동작하지만, 동일한 성능 대비 비용이 훨씬 높습니다.
메인넷에서는 VPS보다 **베어메탈**이 바람직합니다.

> [!NOTE]
> **탈중앙화에 관하여:** 검증인이 여러 업체, 국가, 관할권에 분산되어 있을수록 네트워크는 더 건강합니다.
> 특정 업체나 지역이 이미 검증인 집합의 큰 비중을 차지하고 있다면 다른 곳을 고려하세요.
> 현재 분포는 [익스플로러](https://explorer.ethwei.com/testnet)에서 확인할 수 있습니다.

---

## 1. 서버 준비

서버에 접속한 뒤, 노드를 실행할 비root 사용자를 만듭니다:

```bash
adduser ethwei
usermod -aG sudo ethwei
```

해당 사용자에 대해 SSH 키 인증을 설정한 다음, `/etc/ssh/sshd_config`를 편집해 SSH 데몬을 강화합니다:

```
PermitRootLogin no
PasswordAuthentication no
```

```bash
sudo systemctl restart sshd
```

방화벽을 활성화하고 SSH와 P2P 포트만 허용합니다:

```bash
sudo ufw allow 22/tcp
sudo ufw allow 26656/tcp
sudo ufw enable
```

> [!WARNING]
> 검증인의 RPC(`26657`), API(`1317`), gRPC(`9090`) 포트를 공개 인터넷에 **노출하지 마세요.**
> 로컬호스트에만 바인딩해 두세요.

남은 단계를 위해 새 사용자로 전환합니다:

```bash
su - ethwei
```

---

## 2. 의존성 설치

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl jq build-essential
```

Go 설치(현재 버전은 [go.dev/dl](https://go.dev/dl)에서 확인):

```bash
wget https://go.dev/dl/go1.23.4.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.23.4.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bashrc
source ~/.bashrc
go version
```

---

## 3. 바이너리 빌드

```bash
git clone https://github.com/ethwei/ethwei-chain
cd ethwei-chain
make install
ethweid version
```

---

## 4. 노드 초기화

moniker는 익스플로러에 표시되는 공개 이름입니다.

```bash
ethweid init <your-moniker> --chain-id ethwei-testnet-1
```

제네시스 파일을 가져옵니다:

```bash
curl -s https://rpc-testnet.ethwei.com/genesis | \
  jq '.result.genesis' > ~/.ethwei/config/genesis.json
```

---

## 5. 피어 설정

`~/.ethwei/config/config.toml`을 편집합니다:

```toml
[p2p]
seeds = ""              # 네트워크 정보 참고
persistent_peers = ""   # 네트워크 정보 참고
```

현재 시드 및 피어 주소는 [네트워크 정보]({{< relref "network.md" >}}) 페이지에 게시됩니다.

---

## 6. 노드 구성

`~/.ethwei/config/app.toml`을 편집합니다:

```toml
minimum-gas-prices = "0.025WEI"

# 검증인의 디스크 사용량 관리
pruning = "custom"
pruning-keep-recent = "100"
pruning-interval = "10"
```

검증인은 조회 요청을 제공할 필요가 없으므로, 특별한 이유가 없다면 API와 gRPC 서비스는 비활성화된 상태로
두세요.

---

## 7. 서비스로 실행

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

---

## 8. 동기화 대기

진행 상황 확인:

```bash
ethweid status | jq '.SyncInfo'
```

`catching_up`이 `false`가 될 때까지 기다리세요.

> [!WARNING]
> 노드가 완전히 동기화되기 전에는 검증인을 생성하지 마세요. 동기화 전에 활성 집합에 진입한 검증인은
> 즉시 블록을 놓치기 시작합니다.

---

## 9. 키 생성 및 자금 충전

```bash
ethweid keys add validator
```

니모닉은 오프라인에 보관하세요 — 이 키를 복구할 유일한 수단입니다. 그런 다음 해당 주소에 테스트넷 ETE를
충전합니다.

---

## 10. 검증인 생성

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

[익스플로러](https://explorer.ethwei.com/testnet)에서 검증인이 표시되는지 확인하세요:

```bash
ethweid query staking validator $(ethweid keys show validator --bech val -a)
```

---

## 운영

### 모니터링

최소한 다음 항목에 대해 알림을 설정하세요: 노드 프로세스 중단, `catching_up`이 true로 전환, 블록 누락 증가,
디스크 공간 부족. 노드는 Prometheus 지표를 제공합니다 — `config.toml`에서 활성화하되 반드시 사설
네트워크에서만 수집하세요.

### 업그레이드

```bash
cd ethwei-chain && git pull && make install
sudo systemctl restart ethweid
```

협의된 업그레이드는 지정된 블록 높이에서 진행됩니다. 공지 채널을 주시하고, 미리 하지 말고 유지보수
시간에 맞춰 업그레이드하세요.

### 백업

다음 두 파일을 백업해 오프라인에 보관하세요:

| 파일 | 용도 |
|---|---|
| `~/.ethwei/config/priv_validator_key.json` | 검증인의 서명 키 |
| `~/.ethwei/config/node_key.json` | 노드의 P2P 신원 |

---

## 보안

- **같은 `priv_validator_key.json`으로 두 개의 노드를 절대 실행하지 마세요.** 이중 서명은 프로토콜이
  탐지하여 슬래싱합니다 — 운영자가 저지를 수 있는 가장 치명적인 실수입니다.
- **검증인의 포트는 P2P를 제외하고 닫아두세요.** 메인넷에서는 센트리 노드 구조를 사용해 검증인이 공개
  인터넷에서 직접 접근되지 않도록 하세요.
- **SSH를 제한**하여 키 기반 인증만 허용하고, 가능하면 알려진 주소에서만 접속하도록 하세요.
- 메인넷에서는 **원격 서명기**(Horcrux, TMKMS 등) 사용을 고려하세요. 서명 키가 검증인 호스트에 존재하지
  않게 됩니다.

> [!WARNING]
> **슬래싱(Slashing):** 블록을 너무 많이 놓친 검증인은 감금되고, 이중 서명한 검증인은 슬래싱되어 집합에서
> 영구히 제외됩니다. 두 불이익 모두 위임자에게도 영향을 미칩니다.
