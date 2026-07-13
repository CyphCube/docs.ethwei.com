---
title: "钱包与快速上手"
weight: 2
---

# 钱包与快速上手

以太维使用标准的 Cosmos 钱包。[Keplr](https://www.keplr.app/) 是推荐的钱包，既有桌面浏览器扩展，也有
手机应用。

## 1. 安装 Keplr

**在电脑上**——安装 Keplr 扩展，适用于
[Chrome / Brave](https://chrome.google.com/webstore/detail/keplr/dmkamcknogkgcdfhhbddcghachkejeap)
或 [Firefox](https://addons.mozilla.org/firefox/addon/keplr/)。

**在手机上**——从
[App Store](https://apps.apple.com/app/keplr/id1567851089)（iOS）或
[Google Play](https://play.google.com/store/apps/details?id=com.chainapsis.keplr)（Android）安装 Keplr 应用。

创建或导入钱包，并备份好你的助记词。

## 2. 添加以太维测试网

有两种方式添加以太维测试网——选择更方便的一种即可。

### 方式 A——在 Keplr 中搜索

**在电脑上**——打开 Keplr 扩展，点击顶部的菜单（☰），选择 **Add/Remove Chains**，搜索
**“Ethwei Testnet”**，然后打开开关。

**在手机上**——打开 Keplr 应用，点击菜单（☰），选择 **Add/Remove Chains**（管理链可见性），搜索
**“Ethwei Testnet”**，然后打开开关。

![在 Keplr 的 Add/Remove Chains 界面搜索 “Ethwei Testnet”](/keplr-add-chain.png)

### 方式 B——一键按钮

**在电脑上**——点击下方按钮。Keplr 会请求你批准该网络。接受后，在 Keplr 内切换到 **Ethwei Testnet**。

**在手机上**——在 Keplr 应用内置的浏览器中打开本页面（Keplr → 浏览器标签页，然后输入
`docs.ethwei.com/zh/docs/wallet`），再点击下方按钮。手机版 Safari 和 Chrome 无法直接与 Keplr 应用通信，
因此该按钮只能在 Keplr 内置浏览器中生效。

{{< add-to-keplr network="testnet" >}}

## 3. 获取测试网代币

测试网水龙头：即将上线。在此之前，可以到社区里申请测试网 ETE。

## 主网

{{< add-to-keplr network="mainnet" >}}

主网尚未启动。主网可用后，此按钮将会生效。

---

## 地址格式

所有以太维地址都以 `ete` 开头。例如：

```
ete1qv2scq5r3qws0gu3h0e7h9c7m2t7h5m0zgm5xq
```

添加网络后，你在以太维测试网上的 Keplr 地址会显示在 Keplr 图标下方。
