# AMF - Access and Mobility Management Function

> **中文譯名**：接入與移動管理功能  
> **所屬網域**：Serving Network（服務網路／拜訪網路）

> **一句話直觀定義**：AMF 是手機在 5G 核心網的「第一個窗口」，負責你的註冊與移動管理；它和 [[SEAF]] 住在同一個節點裡，拿到認證成果 $K_{AMF}$ 後，負責保護手機與核心網之間的 NAS 訊令，並把基地台要用的金鑰 $K_{gNB}$ 發下去。

> ⚠️ **命名衝突提醒**：網路功能 AMF（Access and Mobility Management Function）和 AUTN 裡的 AMF 欄位（Authentication Management Field）同名但完全無關 [講義: 5G安全_TS33501-Week4.pdf, p.17]。本頁講的是前者。

---

## 📌 實體角色概述

### 生活化比喻：機場的入境櫃台
AMF 像入境櫃台：旅客（手機）一到就先找它報到（Registration）。櫃台旁邊坐著一位「海關代表」（[[SEAF]]），負責向你的母國（[[AUSF]]）確認身分；確認通過後，海關代表把一把通行鑰匙（$K_{AMF}$）交給櫃台，櫃台再用它刻出你和櫃台之間專用的對講機密碼（NAS 金鑰），以及發給各登機門（基地台）的門禁卡（$K_{gNB}$）。

### 主要功能（講義整理）[講義: 5G 資通安全技術概論-W2.pdf, p.79]
- 無線接入網路控制介面（**N2**）終點
- **NAS 加密與完整性保護**，**N1** 介面終點
- 接入安全金鑰管理所需之安全定位點（Anchor）功能與安全上下文管理
- 註冊、連線、可聯繫（reachability）與移動管理
- 用戶與 SMF 間的會話管理訊息轉發與代理、合法偵聽
- 次要功能：支援非 3GPP 接入（例如 Wi-Fi），作為 N3IWF 的 N2 介面終點

### 4G 的對應角色：MME
4G 的 MME 同樣負責 UE 註冊與註銷、安全金鑰管理、相互認證與加密／完整性保護管理 [講義: 5G 資通安全技術概論-W2.pdf, p.63]。關鍵差異在**認證裁決權**：4G 由拜訪網路的 MME 自行比對結案，5G 改由歸屬網路的 [[AUSF]] 做最終判定 [講義: 5G安全_TS33501-Week4.pdf, p.7]，詳見 [[4G-vs-5G-AKA]]。

---

## 🔐 核心安全職責 (Security Responsibilities)

### 1. 身分管理
- AMF 必須支援用 [[SUCI]] 觸發主要認證，並負責分配與重新分配 5G-GUTI（暫時識別碼）給 UE [TS 33.501 Rel-18 §5.5.3]。
- AMF 必須能向 UE 與歸屬網路確認 [[SUPI]]，**確認失敗就必須拒絕服務** [TS 33.501 Rel-18 §5.5.3]。

### 2. 金鑰推導與管理
- **SEAF 與 AMF 同處一地**：規格明寫 "The SEAF is co-located with the AMF"。SEAF 在認證完成後立即由 $K_{SEAF}$ 產生 $K_{AMF}$ 交給 AMF，之後刪除 $K_{SEAF}$ [TS 33.501 Rel-18 §6.2.2.1]。
- **AMF 產生的金鑰** [TS 33.501 Rel-18 §6.2.2.1]：
  - $K_{NASint}$、$K_{NASenc}$：保護 NAS 層
  - $K_{gNB}$：傳給 gNB，作為空中介面金鑰的根
  - $NH$（連同 NCC）：傳給 gNB，用於換手的前向安全（見 [[Key-Hierarchy]]）
  - $K_{N3IWF}$：傳給 N3IWF，用於非 3GPP 接入
- **AMF 之間移交**：跨 AMF 移動時，AMF 可依政策由 $K_{AMF}$ 推導 $K_{AMF}'$ 交給下一個 AMF [TS 33.501 Rel-18 §6.2.2.1]。

### 3. NAS 安全控制
- 5G AKA 主要認證成功後，由 AMF 發起 NAS 安全模式命令（NAS SMC）[TS 33.501 Rel-18 §6.1.1.1]，流程見 [[Security-Mode-Command]]。
- **演算法支援**：AMF 必須支援 NEA0、128-NEA1、128-NEA2 與 NIA0、128-NIA1、128-NIA2，128-NEA3／128-NIA3 為可選 [TS 33.501 Rel-18 §5.5.1, §5.5.2]。
- **NAS 加密是「可選但應盡量使用」**：規格寫 NAS 加密 "optional to use"，但 "should be used whenever regulations permit"；NAS 完整性與防重放則是 AMF **必須支援**的 [TS 33.501 Rel-18 §5.5.1, §5.5.2]。
- **NIA0 停用**：在法規不要求支援未認證緊急通話的部署中，AMF 必須停用 NIA0 [TS 33.501 Rel-18 §5.5.2]。

---

## 🔄 關聯介面與通訊對象
| 介面 / 服務介面 | 對端實體 | 主要安全信令 / 傳輸內容 | 參考規範條款 |
|---|---|---|---|
| N1（NAS） | [[UE]] | Registration Request（SUCI / 5G-GUTI）、NAS SMC、受 $K_{NASint}$ / $K_{NASenc}$ 保護的 NAS 訊令 | TS 33.501 §6.1.2, §6.7.2；[講義: W2, p.79] |
| N2 | gNB | 傳送 $K_{gNB}$、$\{NH, NCC\}$ 與 UE 安全能力 | TS 33.501 §6.2.2.1；[講義: 5G空中介面安全-Week5.pdf, p.12] |
| 節點內部 | [[SEAF]] | 接收 $K_{AMF}$ 與 ngKSI | TS 33.501 §6.1.3.2.0, §6.2.2.1 |
| N14（AMF 間） | 另一個 AMF | 跨 AMF 移動時傳 $K_{AMF}'$ | TS 33.501 §6.2.2.1, §6.9.3；「N14 訊息」一詞見 §6.9.2.3.3 NOTE 4 |

---

## 🛡️ 安全防禦目標
- **服務網路不能自說自話**：AMF 身旁的 SEAF 拿不到 $K_{SEAF}$ 與 SUPI，直到 [[AUSF]] 判定成功（Home Control）[TS 33.501 Rel-18 §6.1.3.2.0]。
- **核心網訊令不讓基地台看見**：NAS 保護在 UE 與 AMF 之間端到端，跨越基地台，所以一台被入侵的 gNB 讀不到核心網訊令 [講義: 5G安全_TS33501-Week4.pdf, p.23]。分層理由見 [[NAS-vs-AS-Security]]。

---

## 🔗 關聯概念與流程
- 同節點夥伴：[[SEAF]]
- 金鑰：[[Key-Hierarchy]]
- 流程：[[5G-AKA]]、[[Security-Mode-Command]]
- 對比：[[4G-vs-5G-AKA]]、[[NAS-vs-AS-Security]]、[[NSA-vs-SA-Security]]

---

## 📚 參考文獻與條款溯源
- 3GPP TS 33.501 Rel-18:
  - §5.5.1, §5.5.2, §5.5.3 (Requirements on the AMF)
  - §6.1.1.1 (General：5G AKA 後由 AMF 發起 NAS SMC)
  - §6.1.3.2.0 (5G AKA)
  - §6.2.2.1 (Keys in network entities：Keys in the SEAF / Keys in the AMF)
  - §6.9.3 (Key handling in mobility registration update：跨 AMF)
- 課程講義來源:
  - `raw/lectures/5G 資通安全技術概論-W2.pdf`（p.63、p.79）
  - `raw/lectures/5G安全_TS33501-Week4.pdf`（p.7、p.17、p.23）
  - `raw/lectures/5G空中介面安全-Week5.pdf`（p.12）
