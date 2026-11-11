# 用戶隱蔽識別碼 (SUCI - Subscription Concealed Identifier)

> **一句話直觀定義**：SUPI 透過非對稱加密封裝後的「一次性動態化身」，在無線電波中傳送以徹底防止偽基地台竊聽真實身分。

---

## 💡 直觀原理解析 (Beginner-Friendly Explanation)

- **生活化比喻**：  
  如果 [[SUPI]] 是你的真實身分證字號，SUCI 就是你每次進入公共場合時領取的「動態拋棄式號碼牌」。每次連線號碼牌都長得完全不同，旁觀者（中間人攻擊者）就算把每次出現的號碼牌全抄下來，也無法猜出背後是誰，更無法判斷兩張號碼牌是否來自同一個人。
- **為什麼需要它？（破解三十年的密碼學困境）**：  
  在 4G 以前，保護身分面臨一個「雞生蛋、蛋生雞」的死結：**要加密通訊，UE 與網路必須先有共享密鑰；但要找到專屬共享密鑰，網路必須先知道你是誰** [講義: 5G安全_TS33501-Week4.pdf, p.7]。  
  5G 引入非對稱加密（公鑰密碼學）打破此僵局：電信營運商的「公開金鑰（Home Network Public Key）」預先燒錄在 USIM 內。手機尚未認證前，直接使用該公鑰把身分加密成 SUCI，全世界只有持有對應私鑰的歸屬網路 [[SIDF]] 才能解密還原 [TS 33.501 Rel-18 §6.12.2; 講義: 5G安全_TS33501-Week4.pdf, p.7]。

---

## ⚙️ 技術架構與運作機制 (Technical Mechanism)

### 1. SUCI 欄位結構詳解
依據 **3GPP TS 33.501 Rel-18 §6.12.2** 與 **TS 23.003 §28.7.3**，SUCI 並非將整串身分一鍋端加密，而是由明文路由欄位與密文負載組成的精巧結構 [講義: 5G安全_TS33501-Week4.pdf, p.12-13]：

```text
SUCI 結構 (IMSI 型)：
┌───────────┬─────────────────────────┬───────────────────┬──────────────────────┬─────────────────────────┬────────────────────────────────────────────┐
│ SUPI Type │ Home Network Identifier │ Routing Indicator │ Protection Scheme ID │ Home Network PubKey ID  │               Scheme Output                │
│  (0=IMSI) │        (MCC + MNC)      │     (1~4位數字)   │     (0=null, 1=A, 2=B)│         (0~255)         │ (臨時公鑰 || 密文 MSIN || MAC 驗證標籤)    │
└───────────┴─────────────────────────┴───────────────────┴──────────────────────┴─────────────────────────┴────────────────────────────────────────────┘
│<───────────────────────────────── 明文傳輸 ──────────────────────────────────>│<─────────────── ECIES 密碼學輸出 ─────────>│
```

| 欄位名稱 | 傳輸狀態 | 規格功能與說明 | 參考條款 |
|---|---|---|---|
| **SUPI Type** | 明文 | 標註原始身分類型（`0` = IMSI, `1` = NAI） | TS 33.501 §6.12.2 |
| **Home Network Identifier** | 明文 | 歸屬網路識別碼（MCC + MNC 或 realm）。讓拜訪網路知道請求該路由給哪家業者 | TS 33.501 §6.12.2 |
| **Routing Indicator** | 明文 | 1~4 位數字，用於在歸屬網路內部精準路由到特定的 [[UDM]] / [[AUSF]] 伺服器實例 | TS 33.501 §6.12.2, §6.12.5 |
| **Protection Scheme ID** | 明文 | 指示所使用的保護方案（`0x0` = null-scheme, `0x1` = Profile A, `0x2` = Profile B） | TS 33.501 Annex C.1 |
| **Home Network Public Key ID** | 明文 | 標明使用的是第幾把營運商公鑰（0~255），支援公鑰定期輪替（Key Rotation） | TS 33.501 §6.12.2 |
| **Scheme Output** | **已加密** | 唯一受加密保護的核心載體：包含臨時橢圓曲線公鑰、加密後的 MSIN/username 及 MAC 標籤 | TS 33.501 Annex C.1 |

### 2. 深度思維：為什麼不連 MCC/MNC 一起加密？
- **工程與路由取捨**：若將 MCC 與 MNC 也加密，拜訪網路（Serving Network）在收到訊息時將完全無從得知該將訊息轉發至哪一個國家的哪一家電信業者。要做到全匿名路由，必須建立全球分散式匿名路由骨幹，這在跨國電信治理與連線效能上極不實際 [講義: 5G安全_TS33501-Week4.pdf, p.13]。
- **資訊外洩代價**：攻擊者監聽空中介面，雖然能知道「有一位來自某國某電信的用戶出現在此地」，但**絕無法得知具體是哪一位用戶**。

### 3. 特殊例外：Null-Scheme (ID = 0)
依據 **3GPP TS 33.501 Annex C.1、C.2**，當 Protection Scheme ID 為 `0x0` 時，即採用不加密的 null-scheme（輸出等於輸入）。
- **使用場景**：規格限定 UE **只能**在三種情況使用 null-scheme：(1) 進行未認證緊急通話且對該 PLMN 沒有 5G-GUTI；(2) 歸屬網路設定使用 null-scheme；(3) 歸屬網路未在 USIM 配置公鑰 [TS 33.501 Rel-18 §6.12.2; §5.2.5; 講義: 5G安全_TS33501-Week4.pdf, p.14]。
- **風險**：null-scheme 不具備不可連結性（Unlinkability），傳輸的實質就是明文 SUPI，毫無身分隱私保護 [TS 33.501 Annex C.2 NOTE 2]。

### 4. 終端運算實體位置：USIM vs. ME 執行
依據 **3GPP TS 33.501 Rel-18 §6.12.2**：
- **由營運商決定、由 USIM 指示**：SUCI 由 USIM 或 ME 計算，取決於「歸屬營運商的決定」，此決定以存在 USIM 中的 SUCI calculation indication 表示；若沒有此指示，預設由 ME 計算 [TS 33.501 Rel-18 §5.2.5 NOTE 1, §6.12.2]。
- **USIM 計算時**：USIM 不得把公鑰、公鑰識別碼、保護方案識別碼等任何計算參數交給 ME。
- **ME 計算時**：營運商須在 USIM 中配置允許的保護方案優先序清單；若公鑰或優先序清單未配置，ME 以 null-scheme 計算 [TS 33.501 Rel-18 §6.12.2]。

> [!NOTE]
> 舊版本頁面寫「若 USIM 硬體支援，ME 必須指示 USIM 計算」，與規格不符：決定權在營運商，不在 USIM 硬體能力。（2026-11-11 Claude Code 查核修正）

---

## 🛡️ 安全防禦目標 (Security Objectives)

1. **使用者身分隱私性 (Subscriber Identity Confidentiality)**：阻止空中無線電監聽者透過靜態識別碼辨識用戶。
2. **不可連結性 (Unlinkability)**：藉由 [[ECIES]] 每次生成全新的臨時金鑰對，同一個 SUPI 每次產生的 SUCI 皆不同，使攻擊者無法把兩次通訊關聯至同一人 [TS 33.501 Annex C.2 NOTE 1; 講義: 5G安全_TS33501-Week4.pdf, p.13]。
3. **密文完整性保護**：Scheme Output 內嵌 MAC 認證碼，防止中間人竄改密文內容 [TS 33.501 Annex C.3.2]。

> [!WARNING]
> **講義 Week3/4 對 SUCI 安全特性的兩個過度宣稱**：
> 1. **「前向保密 (Forward Secrecy)」**：p.74 把「每次註冊都生成新臨時金鑰對」稱為前向保密。但 Week4 講義 p.10 指出，若 SIDF 的私鑰外洩，「全網用戶過去與未來的 SUCI 都可被還原成 SUPI」——這恰好說明 SUCI **沒有**前向保密：臨時金鑰對帶來的是**不可連結性**，而非前向保密 [講義: 5G安全_TS33501-Week4.pdf, p.10, 13]。
> 2. **「抗重放」**：p.74–75 稱「每次加密輸出唯一」「每次隨機金鑰重新生成，防止重放」。UE 每次產生新 SUCI，並不能阻止攻擊者把**錄下的舊 SUCI** 再送一次；TS 33.501 也沒有把 SUCI 列為防重放機制。真正擋住重放的是後續的 [[5G-AKA]]（RAND 與 SQN）[講義: 5G安全_TS33501-Week4.pdf, p.17]。
> （2026-11-11 Claude Code，Week3/4 ingest 補記）

---

## 🔗 關聯概念與流程
- **加密核心**：[[ECIES]]（SUCI 的非對稱混合加密算法）
- **關聯網元**：[[SIDF]]（歸屬網路解密實體）、[[UDM]]、[[UE]]
- **對應身分**：[[SUPI]]（原始永久身分）
- **認證流程**：[[5G-AKA]]（註冊請求首個攜帶的參數）

---

## 📚 參考文獻與條款溯源 (References & Traceability)
- 3GPP TS 33.501 Rel-18:
  - §6.12.2 (Subscription concealed identifier specification)
  - §6.12.5 (Subscription identifier de-concealing function)
  - Annex C.1 & C.2 (Protection schemes for concealing SUPI, Null-scheme)
- 3GPP TS 23.003: §28.7.3 (SUCI structure and coding)
- 課程講義來源: `raw/lectures/5G安全_TS33501-Week4.pdf` (第 1 章 p.7、第 3 章 p.12-14)
