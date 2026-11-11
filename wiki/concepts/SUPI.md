# 用戶永久識別碼 (SUPI - Subscription Permanent Identifier)

> **一句話直觀定義**：5G 網路中每位使用者的「數位身分證號碼」，是全球唯一的永久身分標識，取代了 2G/3G/4G 時代的 IMSI。

---

## 💡 直觀原理解析 (Beginner-Friendly Explanation)

- **生活化比喻**：  
  SUPI 就像你的「身分證字號」。無論你走到哪裡、換了哪隻手機，只要 SIM 卡不變，這個身分號碼終生不變。正因為它永久且唯一，如果直接在街上（空中無線電波）大聲喊出來，壞人（偽基站）就能輕易記錄你的行蹤。
- **為什麼需要它？（痛點與背景）**：  
  在 2G/3G/4G 時代，終端直接使用 IMSI（國際移動用戶識別碼）向網路報到 [講義: 5G安全_TS33501-Week4.pdf, p.6]。由於空中介面缺乏初次加密機制，攻擊者利用 IMSI Catcher（假基地台）發送 `Identity Request`，手機就會交出明文 IMSI，導致使用者的地理位置長期遭到非法追蹤監聽 [講義: 5G安全_TS33501-Week4.pdf, p.6-7]。5G 雖然仍需要一個永久識別碼來綁定帳務與電信訂閱，但制定了嚴格的隱私規範，禁止它以明文形式暴露在無線電波中。

---

## ⚙️ 技術架構與運作機制 (Technical Mechanism)

### 1. SUPI 的兩大資料格式
依據 **3GPP TS 33.501 Rel-18 §6.12.1** 與 **TS 23.003**，SUPI 主要存在兩種編碼形式 [講義: 5G安全_TS33501-Week4.pdf, p.12]：

```text
型態一：IMSI 型 (公眾行動網路 PLMN 常用)
┌──────────────┬──────────────┬────────────────────────┐
│  MCC (3位)   │  MNC (2~3位) │      MSIN (用戶號)     │
└──────────────┴──────────────┴────────────────────────┘
└────── 歸屬網路識別碼 ───────┘ └──── 隱私加密保護區間 ────┘
  (Home Network Identifier)        (Subscription Identifier)

型態二：NAI 型 (RFC 7542 格式，用於專網 SNPN 或 IoT)
┌──────────────────────────────┬────────────────────────┐
│           username           │        @realm          │
└──────────────────────────────┴────────────────────────┘
└────── 隱私加密保護區間 ───────┘ └──── 歸屬網域名稱(明文) ──┘
```

- **IMSI 型**：
  - **MCC (Mobile Country Code)**：行動國碼（例如台灣為 466）。
  - **MNC (Mobile Network Code)**：行動網路碼（電信業者代號，如 01、92）。
  - **MSIN (Mobile Subscription Identification Number)**：用戶識別號碼（真正標識特定個人的部分）。
- **NAI 型 (Network Access Identifier)**：
  - 格式如 `user@realm.example.com`，常用於獨立非公眾網路（SNPN）或免 USIM 憑證認證情境。

### 2. 規格三大鐵律 (Three Golden Rules)
3GPP TS 33.501 對 SUPI 的防護設立了嚴密的規範要求：
1. **不以明文在空中傳送**：SUPI 不應以明文透過空中介面（NG-RAN）傳送（路由資訊 MCC/MNC 除外），而是以 [[ECIES]] 等保護方案封裝為 [[SUCI]]。未認證的緊急通話是規格列出的例外（"privacy protection for SUPI is not required"）[TS 33.501 Rel-18 §5.2.5, §6.12.2; 講義: 5G安全_TS33501-Week4.pdf, p.12]。

> [!NOTE]
> **講義與規格的用語差異**：講義 p.12 寫「SUPI **不得**以明文透過空中介面傳送」，但 TS 33.501 Rel-18 §5.2.5 原文用的是 **should not**（強烈建議）而非 shall not；實際的強制力來自 §6.12.2 對 UE 產生 SUCI 的 shall 規定，以及 null-scheme 只能在三種情況使用的限制。講義 p.5 自己也提醒「很多爭論源自把 should 誤讀成 shall」。（2026-11-11 Claude Code 查核補記）
2. **受限儲存與傳遞**：SUPI 在核心網內必須由歸屬網路的 [[UDM]] / [[ARPF]] 安全儲存，且僅能在受保護的內部介面上傳輸 [TS 33.501 Rel-18 §5.2.4, §6.12.1; 講義: 5G安全_TS33501-Week4.pdf, p.12]。
3. **成功認證後始交付**：當 UE 漫遊至拜訪網路時，拜訪網路（如 [[AMF]] / [[SEAF]]）在認證流程開始時只能拿到加密的 [[SUCI]]；只有在 [[AUSF]] 與 UDM 判定認證完全成功後，歸屬網路才會將解密後的 SUPI 交付給拜訪網路 [TS 33.501 Rel-18 §6.1.3.2; 講義: 5G安全_TS33501-Week4.pdf, p.12, 17]。

---

## 🛡️ 安全防禦目標 (Security Objectives)

| 防禦目標 | 達成的安全機制 | 參考條款 |
|---|---|---|
| **防範身分竊聽 (Eavesdropping)** | 空中介面一律將 SUPI 隱蔽為 [[SUCI]]，攔截者無法讀取真實號碼 | [TS 33.501 Rel-18 §6.12.2] |
| **防範偽基地台定位追蹤 (IMSI Catching)** | 攻擊者即便發送 Identity Request，也只能獲得每次動態變化的 SUCI，無法長期追蹤特定設備 | [TS 33.501 Rel-18 §5.2.5] |
| **防範漫遊拜訪網路詐欺 (Serving Network Fraud)** | 拜訪網路無法在認證前獲取用戶真實身分，防止未經認證擅自扣款或偽造記錄 | [TS 33.501 Rel-18 §6.1.3] |

---

## 🔗 關聯概念與實體
- **衍生概念**：[[SUCI]]（SUPI 的加密型態）、[[ECIES]]（SUPI 加密演算法）
- **關聯網元**：[[SIDF]]（唯一解密 SUPI 的實體）、[[UDM]]（儲存 SUPI 的核心資料庫）、[[ARPF]]（金鑰存放處）
- **涉及流程**：[[5G-AKA]]（主要認證與身分交付流程）

---

## 📚 參考文獻與條款溯源 (References & Traceability)
- 3GPP TS 33.501 Rel-18:
  - §5.2.5 (Subscriber privacy requirements on UE)
  - §6.12.1 (Subscription permanent identifier privacy)
  - §6.1.3.2 (5G AKA authentication procedure)
- 3GPP TS 23.003: §2.2 (Composition of IMSI), §28.7 (Structure of SUPI)
- 課程講義來源: `raw/lectures/5G安全_TS33501-Week4.pdf` (第 1 章 p.6-7、第 3 章 p.12)
