# 2G → 5G 行動通訊安全演進對比 (Security Evolution across Cellular Generations)

> **一句話直觀定義**：從 2G 到 5G，每一代都在補上一代最痛的洞——2G 只驗證你、不驗證基地台；3G 開始雙向認證；4G 把金鑰分層；5G 終於把「你的身分」也藏起來，並把認證的最終決定權交回你的電信商。

---

## 💡 直觀原理解析 (Beginner-Friendly Explanation)

### 生活化比喻：大樓門禁的四次升級
| 世代 | 門禁比喻 | 背後的安全概念 |
|---|---|---|
| 2G | 警衛檢查你的通行證，但你不會檢查警衛是不是真的 | 單向認證 |
| 3G | 你和警衛互對暗號，雙方都確認對方是自己人 | 雙向認證（AKA）＋序號防重放 |
| 4G | 大樓、樓層、房間各用不同鑰匙，丟一把不會全丟 | 金鑰階層化、NAS／AS 分層保護 |
| 5G | 你出示的是一次性代號，不再大聲報出身分證字號；是否放行由你的公司總部決定，不是由這棟樓的警衛自己說了算 | [[SUCI]] 身分隱藏、Home Control |

### 為什麼要看這張演進表？
講義歸納出一條清楚的軌跡：**安全責任逐步從「拜訪網路」移回「歸屬網路」，保護的對象也從「連線內容」擴大到「用戶身分」**[講義: 5G安全_TS33501-Week4.pdf, p.6]。理解這條軌跡，就理解 5G 為什麼要做 SUCI、為什麼要做 Home Control。

---

## ⚙️ 技術對比 (Technical Comparison)

### 1. 認證與身分：四個世代總表
| 世代（年份） | 認證方式 | 身分傳送 | 主要弱點 | 出處 |
|---|---|---|---|---|
| **2G / GSM**（1991） | 單向認證：只有網路驗證用戶 | IMSI 明文 | 網路可被冒充；A5/1、A5/2 強度不足，加密已被實務破解 | [講義: 5G安全_TS33501-Week4.pdf, p.6] |
| **3G / UMTS**（2001） | 引入 AKA：雙向認證，以序號 SQN 防重放；KASUMI 演算法 | IMSI 仍在初次連線時明文傳送 | 身分隱私未解決 | [講義: 5G安全_TS33501-Week4.pdf, p.6] |
| **4G / LTE**（2009） | EPS-AKA；金鑰階層化，NAS 與 AS 分層保護 | IMSI 仍明文 | 認證決定權在拜訪網路；使用者平面無完整性保護 | [講義: 5G安全_TS33501-Week4.pdf, p.6] |
| **5G / NR**（2018） | [[5G-AKA]] / [[EAP-AKA-prime]]；Home Control | [[SUCI]] 隱藏永久身分 | 連線前訊息仍未保護；AKA 可連結性問題未完全解決 | [講義: 5G安全_TS33501-Week4.pdf, p.6；認證方法見 p.15] |

**2G 單向認證長什麼樣子？**（GSM/UMTS 的基本流程）[講義: 5G資通-Week3, Week4.pdf, p.57]
1. UE → 網路：IMSI
2. 網路 → SIM：RAND（隨機挑戰）
3. SIM → 網路：SRES（Signed Response）
4. 網路比對，完成認證

對照 5G：UE 先產生 SUCI（加密的身分），網路端解密出 SUPI 後才執行 5G-AKA [講義: 5G資通-Week3, Week4.pdf, p.57]。差別一眼可見：2G 的流程裡，**手機從頭到尾沒有驗證過網路**，這就是假基地台能存在的根本原因。

### 2. 加密與完整性演算法：四個世代總表
| 世代 | 加密演算法 (Ciphering) | 完整性演算法 (Integrity) | 出處 |
|---|---|---|---|
| **GSM (2G)** | A5/1、A5/2、A5/3 | **無**（只靠加密） | [講義: 5G資通-Week3, Week4.pdf, p.80] |
| **UMTS (3G)** | UEA1 (KASUMI)、UEA2 (SNOW 3G) | UIA1 (KASUMI)、UIA2 (SNOW 3G) | [講義: 5G資通-Week3, Week4.pdf, p.80] |
| **LTE (4G)** | EEA1 (SNOW 3G)、EEA2 (AES)、EEA3 (ZUC) | EIA1 (SNOW 3G)、EIA2 (AES)、EIA3 (ZUC) | [講義: 5G資通-Week3, Week4.pdf, p.80] |
| **5G NR** | NEA1 (SNOW 3G)、NEA2 (AES-CTR)、NEA3 (ZUC) | NIA1 (SNOW 3G)、NIA2 (AES-CMAC)、NIA3 (ZUC) | [講義: 5G資通-Week3, Week4.pdf, p.80] |

- **5G 直接沿用 4G 演算法**：規格寫明 128-NEA1/2/3 分別與 4G 的 128-EEA1/2/3 完全相同（"is identical to"），128-NIA1/2/3 也分別與 128-EIA1/2/3 相同 [TS 33.501 Rel-18 Annex D.2.1.2–D.2.1.4, D.3.1.2–D.3.1.4]。
- **哪些是手機必備**：UE **必須**實作 NEA0、128-NEA1、128-NEA2 與 NIA0、128-NIA1、128-NIA2；128-NEA3 與 128-NIA3（ZUC）則是**可選** [TS 33.501 Rel-18 §5.2.2, §5.2.3]。
- **四種演算法核心的特性**：KASUMI 為區塊加密、已部分淘汰；SNOW 3G 為流加密、穩定成熟；ZUC 為流加密、中國自主演算法；AES 為區塊加密、NIST 國際標準 [講義: 5G資通-Week3, Week4.pdf, p.81]。三套 5G 演算法的數學基礎刻意不同，是「密碼敏捷性」的具體做法，細節見 [[Air-Interface-Security]]。

> [!WARNING]
> **講義 Week3/4 的演算法編號前後矛盾**：p.78 的表格把 AES 標為「EEA4/EIA4」，但同一份講義 p.80 寫 EEA2 = AES、EEA3 = ZUC。規格寫明 128-NEA2 與 128-EEA2 相同、128-NEA3 與 128-EEA3 相同 [TS 33.501 Rel-18 Annex D.2.1.3–D.2.1.4]，而 Week4 講義 p.24 也寫 128-NEA2 的核心是 AES-CTR、128-NEA3 是 ZUC [講義: 5G安全_TS33501-Week4.pdf, p.24]。**以 p.80 為準，p.78 的「EEA4/EIA4」應視為誤植**（AES 對應的是 EEA2/EIA2）。

### 3. 使用者平面完整性：4G 沒有、5G 可選
- 4G 使用者平面只有加密、沒有完整性保護；5G 允許依 SMF 下發的 UP security policy，對每一個 DRB 逐一啟用完整性保護 [講義: 5G安全_TS33501-Week4.pdf, p.7–8; TS 33.501 Rel-18 §6.6.1]。
- 規格要求 UE **必須支援**使用者資料的完整性保護，但**是否使用**是可選的 [TS 33.501 Rel-18 §5.2.3]。細節與攻擊實例見 [[User-Plane-Integrity]]。

> [!WARNING]
> **講義 Week3/4 p.84 的過時說法**：該頁寫「User Plane：數據流量加密（無完整性）」。這描述的是 4G，不是 5G；TS 33.501 Rel-18 §6.6.1 明訂 UP security policy 可要求啟用 UP integrity protection。簡報請勿引用 p.84 這句。

---

## 🛡️ 安全防禦目標 (Security Objectives)：每一代補了什麼洞
| 升級 | 補上的洞 | 對應的攻擊 |
|---|---|---|
| 2G → 3G | 手機開始驗證網路（雙向認證）、SQN 防重放 | 假基地台冒充網路、重放舊認證資料 |
| 3G → 4G | 金鑰分層、NAS / AS 分開保護 | 單一金鑰外洩波及全部通訊 |
| 4G → 5G | 身分加密（SUCI）、Home Control、使用者平面可做完整性保護 | IMSI Catcher 追蹤、漫遊詐欺、位元翻轉竄改（如 aLTEr） |
| 5G 仍未解決 | 連線建立前的訊息（Paging、SIB 等）仍未受完整性保護 | 偽基地台的剩餘攻擊面，見 [[False-Base-Station-Defenses]] |

（2G→4G 各列依據 [講義: 5G安全_TS33501-Week4.pdf, p.6–8]）

---

## 🔗 關聯概念與流程
- 身分保護：[[SUPI]]、[[SUCI]]、[[USIM]]
- 認證：[[5G-AKA]]、[[4G-vs-5G-AKA]]
- 空中介面：[[Air-Interface-Security]]、[[User-Plane-Integrity]]
- 剩餘風險：[[False-Base-Station-Defenses]]

---

## 📚 參考文獻與條款溯源 (References & Traceability)
- 3GPP TS 33.501 Rel-18:
  - §5.2.2 (User data and signalling data confidentiality：UE 必須／可選實作的 NEA)
  - §5.2.3 (User data and signalling data integrity：UE 必須／可選實作的 NIA、UP 完整性可選)
  - §6.6.1 (UP security policy)
  - Annex D.2.1.2–D.2.1.4、D.3.1.2–D.3.1.4 (128-NEA/NIA 與 128-EEA/EIA 相同)
- 課程講義來源:
  - `raw/lectures/5G資通-Week3, Week4.pdf`（p.57、p.78、p.80、p.81、p.84）
  - `raw/lectures/5G安全_TS33501-Week4.pdf`（p.6–8、p.15、p.24）
