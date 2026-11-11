# UE - User Equipment

> **中文譯名**：使用者設備（就是你的手機）  
> **所屬網域**：UE（使用者端）

> **一句話直觀定義**：UE 是「手機本體（ME）＋ USIM 卡」的組合；USIM 負責保管秘密與做最核心的運算，ME 負責其餘的推導、跟網路講話，以及檢查網路有沒有騙它。

---

## 📌 實體角色概述

### 生活化比喻：旅客與護照晶片
UE 像一位出國的旅客：**護照晶片（[[USIM]]）**裡存著只有發照國家才知道的秘密，海關問問題時由晶片在內部算出答案；**旅客本人（ME，Mobile Equipment，手機本體）**負責排隊、遞交文件、確認櫃台是不是真的海關，並把晶片給的答案整理成海關要的格式。

### UE 的兩個部分 [TS 33.501 Rel-18 §6.2.2.2]
| 部分 | 保管 / 負責 | 出處 |
|---|---|---|
| **USIM** | 長期金鑰 $K$（與 [[ARPF]] 相同）；以 $K$ 產生金鑰材料交給 ME；若營運商有配置，也儲存用來隱藏 SUPI 的歸屬網路公鑰 | TS 33.501 §6.2.2.2（Keys in the USIM） |
| **ME（手機本體）** | 用 USIM 給的 CK、IK 產生 $K_{AUSF}$；使用 5G AKA 時由 RES 計算 RES*；往下推導整條金鑰樹 | TS 33.501 §6.2.2.2（Keys in the ME） |

規格的原則是：網路端每一把金鑰，在 UE 端都有一把對應的金鑰 [TS 33.501 Rel-18 §6.2.2.2]。

---

## 🔐 核心安全職責 (Security Responsibilities)

### 1. 身分管理：開口第一句話就要保護身分
- 初次註冊時，UE 在 Registration Request 中放 [[SUCI]]（若已有暫時識別碼則放 5G-GUTI）；此時尚無任何安全保護 [講義: 5G安全_TS33501-Week4.pdf, p.16]。
- SUCI 由 USIM 或 ME 計算，由營運商決定 [TS 33.501 Rel-18 §5.2.5, §6.12.2]（詳見 [[USIM]]）。
- 連線建立後改用暫時識別碼 5G-GUTI，取代 SUCI 用於後續通訊 [講義: 5G資通-Week3, Week4.pdf, p.71]。

### 2. 認證處理：UE 也要驗證網路
- **驗證網路（UE 認證網路）**：USIM 先驗證 AUTN 中的 MAC——通過即證明對方持有 $K$；再檢查 SQN 是否在可接受範圍——防止重放 [講義: 5G安全_TS33501-Week4.pdf, p.17]。
- **兩種失敗回應**：MAC 驗證失敗回覆 MAC failure（對方很可能是偽基地台）；SQN 超出範圍回覆 Synchronisation failure 並附 AUTS 供網路重新同步 [講義: 5G安全_TS33501-Week4.pdf, p.18]。
- **只交答案、不交鑰匙**：UE 在 Authentication Response 中只帶 RES*，**從不傳送任何金鑰** [講義: 5G安全_TS33501-Week4.pdf, p.17]。流程見 [[5G-AKA]]。

### 3. 金鑰推導與管理
- ME 由 CK、IK 推導 $K_{AUSF}$，再往下推導 $K_{SEAF}$、$K_{AMF}$ 等，與網路端各自算出同一組金鑰 [講義: 5G安全_TS33501-Week4.pdf, p.17; TS 33.501 Rel-18 §6.2.2.2]，整棵樹見 [[Key-Hierarchy]]。
- 認證成功後，UE 保存最新的 $K_{AUSF}$：USIM 支援 5G 參數儲存則存在 USIM，否則存在 ME 的非揮發性記憶體 [TS 33.501 Rel-18 §6.2.2.2]。

### 4. 安全模式與演算法：防止被騙去用弱密碼
- UE 在 Registration Request 中宣告自己支援的演算法清單；這則訊息此時沒有保護，可能被竄改 [講義: 5G空中介面安全-Week5.pdf, p.12]。
- 收到 NAS Security Mode Command 時，UE 檢查兩件事：完整性驗證是否通過、訊息中**重播的安全能力**是否與自己送出的一致；任一失敗即拒絕 [講義: 5G安全_TS33501-Week4.pdf, p.23]。流程見 [[Security-Mode-Command]]。
- **選擇權在網路端**：gNB 從「UE 支援的」與「本地優先序清單」的交集中挑選，UE 不能要求使用特定演算法 [講義: 5G空中介面安全-Week5.pdf, p.12]。
- UE **必須**實作 NEA0、128-NEA1、128-NEA2 與 NIA0、128-NIA1、128-NIA2 [TS 33.501 Rel-18 §5.2.2, §5.2.3]。

### 5. 使用者可以自己做的事
- 多數手機有「僅 5G」模式可防跨世代降級，但預設不開 [講義: 5G空中介面安全-Week5.pdf, p.20]；部分裝置提供「2G 關閉」選項 [講義: 5G空中介面安全-Week5.pdf, p.28]。
- 把無線層當作不可信的通道，一律使用 TLS、加密通訊軟體、VPN [講義: 5G空中介面安全-Week5.pdf, p.28]（見 [[False-Base-Station-Defenses]]）。

---

## 🔄 關聯介面與通訊對象
| 介面 / 服務介面 | 對端實體 | 主要安全信令 / 傳輸內容 | 參考規範條款 |
|---|---|---|---|
| UE 內部 | [[USIM]] ↔ ME | USIM 回傳 RES、CK、IK；ME 算 RES*、$K_{AUSF}$ | TS 33.501 §6.1.3.2.0 step 7, §6.2.2.2 |
| N1（NAS） | [[AMF]] / [[SEAF]] | Registration Request（SUCI）、Authentication Request / Response、NAS SMC | TS 33.501 §6.1.3.2.0, §6.7.2 |
| Uu（空中介面） | gNB（基地台） | AS SMC、受 PDCP 保護的 RRC 訊令與使用者資料 | TS 33.501 §6.7.4；[[Air-Interface-Security]] |

---

## 🔗 關聯概念與流程
- 組成：[[USIM]]
- 身分：[[SUPI]]、[[SUCI]]
- 流程：[[5G-AKA]]、[[Security-Mode-Command]]
- 金鑰：[[Key-Hierarchy]]
- 威脅與自保：[[False-Base-Station-Defenses]]、[[NSA-vs-SA-Security]]

---

## 📚 參考文獻與條款溯源
- 3GPP TS 33.501 Rel-18:
  - §5.2.2, §5.2.3 (UE 必須實作的加密與完整性演算法)
  - §5.2.5, §6.12.2 (SUCI 由 USIM 或 ME 計算)
  - §6.1.3.2.0 (5G AKA)
  - §6.2.2.2 (Keys in the UE：Keys in the USIM / Keys in the ME)
  - §6.7.2, §6.7.4 (NAS / AS SMC)
- 課程講義來源:
  - `raw/lectures/5G安全_TS33501-Week4.pdf`（p.16–18、p.23）
  - `raw/lectures/5G空中介面安全-Week5.pdf`（p.12、p.20、p.28）
  - `raw/lectures/5G資通-Week3, Week4.pdf`（p.71）
