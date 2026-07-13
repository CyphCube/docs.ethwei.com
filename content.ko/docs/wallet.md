---
title: "지갑 및 시작하기"
weight: 2
---

# 지갑 및 시작하기

이더웨이는 표준 Cosmos 지갑을 사용합니다. [Keplr](https://www.keplr.app/)를 권장하며, 데스크톱 브라우저
확장 프로그램과 모바일 앱 모두 제공됩니다.

## 1. Keplr 설치

**컴퓨터에서** — Keplr 확장 프로그램을 설치하세요:
[Chrome / Brave](https://chrome.google.com/webstore/detail/keplr/dmkamcknogkgcdfhhbddcghachkejeap)
또는 [Firefox](https://addons.mozilla.org/firefox/addon/keplr/).

**모바일에서** — Keplr 앱을 설치하세요:
[App Store](https://apps.apple.com/app/keplr/id1567851089)(iOS) 또는
[Google Play](https://play.google.com/store/apps/details?id=com.chainapsis.keplr)(Android).

지갑을 생성하거나 가져온 후 시드 문구를 백업하세요.

## 2. 이더웨이 테스트넷 추가

이더웨이 테스트넷을 추가하는 방법은 두 가지입니다 — 더 편한 방법을 선택하세요.

### 방법 A — 원클릭 버튼

**컴퓨터에서** — 아래 버튼을 클릭하세요. Keplr가 네트워크 승인을 요청합니다. 승인한 후 Keplr에서
**Ethwei Testnet**으로 전환하세요.

**모바일에서** — Keplr 앱 내장 브라우저에서 이 페이지를 여세요(Keplr → 브라우저 탭에서
`docs.ethwei.com/ko/docs/wallet` 입력). 그런 다음 아래 버튼을 누르세요. 모바일 Safari와 Chrome은 Keplr
앱과 직접 통신할 수 없으므로, 이 버튼은 Keplr 내장 브라우저에서만 작동합니다.

{{< add-to-keplr network="testnet" >}}

### 방법 B — Keplr에서 검색

이더웨이 테스트넷은 이제 공식 Keplr 체인 레지스트리(Keplr Chain Registry)에 등록되어 있으므로, 버튼이나
브라우저 없이 Keplr에서 직접 추가할 수 있습니다.

**컴퓨터에서** — Keplr 확장 프로그램을 열고 상단의 메뉴(☰)를 클릭한 후 **Add/Remove Chains**를 선택하고,
**"Ethwei Testnet"**을 검색한 다음 토글을 켜세요.

**모바일에서** — Keplr 앱을 열고 메뉴(☰)를 탭한 후 **Add/Remove Chains**(체인 표시 관리)를 선택하고,
**"Ethwei Testnet"**을 검색한 다음 토글을 켜세요.

![Keplr의 Add/Remove Chains 화면에서 "Ethwei Testnet" 검색](/keplr-add-chain.png)

## 3. 테스트넷 토큰 받기

테스트넷 포셋(faucet): 준비 중입니다. 그때까지는 커뮤니티에서 테스트넷 ETE를 요청하세요.

## 메인넷

{{< add-to-keplr network="mainnet" >}}

메인넷은 아직 시작되지 않았습니다. 메인넷이 제공되면 이 버튼이 활성화됩니다.

---

## 주소 형식

모든 이더웨이 주소는 `ete`로 시작합니다. 예시:

```
ete1qv2scq5r3qws0gu3h0e7h9c7m2t7h5m0zgm5xq
```

네트워크를 추가하면 이더웨이 테스트넷의 Keplr 주소가 Keplr 아이콘 아래에 표시됩니다.
