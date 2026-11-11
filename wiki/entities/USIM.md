# USIM - Universal Subscriber Identity Module

> **中文譯名**：通用用戶身分模組（俗稱 SIM 卡）  
> **所屬網域**：UE（使用者設備端，與 ME 手機本體共同組成 [[UE]]）

> **一句話直觀定義**：USIM 是手機裡的「防拆身分保險箱」，同時裝著你的永久身分（IMSI / [[SUPI]]）和整個 5G 安全的信任根——長期金鑰 $K$；手機本體只能請它「算出答案」，永遠拿不到 $K$ 本身。

---

## 📌 實體角色概述

### 生活化比喻：銀行保險箱裡的印鑑
把 USIM 想成一個只開小窗口的保險箱：銀行（歸屬網路）把你的印鑑（$K$）鎖在裡面，窗口外的人（手機 ME、基地台、甚至你自己）只能遞進一張考題（RAND、AUTN），保險箱在裡面蓋完章，遞出蓋好的結果（RES、CK、IK）。印鑑本身從不離開保險箱。

### 在 5G 安全架構中的定位
- **實體載體**：USIM 必須位於 UICC（通用積體電路卡）上，UICC 可以是可拔除的實體卡，也可以是不可拔除的嵌入式晶片 [TS 33.501 Rel-18 §6.1.1.1]。
- **SIM 家族演進**：SIM（2G，為 GSM 設計）→ USIM（3G/4G，支援 UMTS/LTE 的認證與金鑰管理）→ ISIM（IMS 多媒體子系統用）→ eSIM / iSIM（嵌入式／整合式，支援遠端配置）[講義: 5G資通-Week3, Week4.pdf, p.29]。
- **身分 vs 容器**：IMSI 是「邏輯身分」，SIM 是存放這個身分的「實體容器」；SIM 內部以 EF_IMSI 檔案儲存 IMSI [講義: 5G資通-Week3, Week4.pdf, p.31]。

---

## 🔐 核心安全職責 (Security Responsibilities)

### 1. 身分管理：保管 IMSI / SUPI 與 SUCI 所需參數
- **IMSI 格式**：最多 15 位數字，`IMSI = MCC (3 位) | MNC (2–3 位) | MSIN (最多 10 位)`，例如 `466-97-123456789`（MCC 466 為台灣）[講義: 5G資通-Week3, Week4.pdf, p.28, 30]。5G 的 [[SUPI]] 通常仍以 IMSI 格式表示 [講義: 5G資通-Week3, Week4.pdf, p.55]。
- **SUCI 所需參數一律存在 USIM**：歸屬網路公鑰（Home Network Public Key）、保護方案識別碼（Protection Scheme Identifier）、公鑰識別碼（Home Network Public Key Identifier），以及「由 USIM 還是 ME 計算 SUCI」的指示（SUCI calculation indication）[TS 33.501 Rel-18 §5.2.5]。
- **誰計算 [[SUCI]]**：由歸屬營運商決定、以 USIM 中的指示表示；沒有指示時預設由 ME 計算。若決定由 USIM 計算，USIM 不得把公鑰等任何計算參數交給 ME [TS 33.501 Rel-18 §5.2.5 NOTE 1, §6.12.2]。

### 2. 金鑰管理：保管信任根 $K$
- **$K$ 與 ARPF 成對存在**：USIM 儲存與 [[ARPF]] 相同的長期金鑰 $K$；認證時由 USIM 以 $K$ 產生金鑰材料（CK、IK）交給 ME [TS 33.501 Rel-18 §6.2.2.2]。講義稱 $K$「絕不可導出」[講義: 5G資通-Week3, Week4.pdf, p.27]。
- **USIM 與 ME 的分工**：ME 用 USIM 給的 CK、IK 產生 $K_{AUSF}$；使用 [[5G-AKA]] 時，由 RES 計算 RES* 的工作也由 ME 執行 [TS 33.501 Rel-18 §6.2.2.2]。
- **$K_{AUSF}$ 存哪裡**：若 USIM 支援 5G 參數儲存，最新的 $K_{AUSF}$ 存在 USIM；否則存在 ME 的非揮發性記憶體 [TS 33.501 Rel-18 §6.2.2.2]。
- 後續整條金鑰樹（$K_{AUSF} \to K_{SEAF} \to K_{AMF} \to \dots$）見 [[Key-Hierarchy]]。

### 3. 認證處理：驗證網路並產生回應
- 收到 RAND 與 AUTN 後，USIM 依 TS 33.102 檢查 AUTN 是否可接受（規格原文："verify the freshness of the received values"），通過後計算 RES，並把 RES、CK、IK 交給 ME [TS 33.501 Rel-18 §6.1.3.2.0 step 7]。講義補充：這個檢查包含驗證 AUTN 中的 MAC（證明對方持有 $K$）與 SQN 是否在可接受範圍 [講義: 5G安全_TS33501-Week4.pdf, p.17]。這一步就是「UE 認證網路」的方向，細節見 [[5G-AKA]]。

### 4. 檔案系統與存取控制
- **三層結構**：MF（Master File，根目錄）→ DF（Dedicated File，分類目錄，如 DF_TELECOM、ADF_USIM）→ EF（Elementary File，實際存放資料的最小單位）[講義: 5G資通-Week3, Week4.pdf, p.33, 35]。
- **存取權限**：EF_IMSI 需通過 PIN1 驗證才能讀取 [講義: 5G資通-Week3, Week4.pdf, p.37]；一般 EF 的安全屬性為 READ 需 PIN1、UPDATE 需 ADM 權限，並定義於 FCP（File Control Parameters）中 [講義: 5G資通-Week3, Week4.pdf, p.43]。

> [!WARNING]
> **講義內部不一致：EF_KEYS 到底存什麼？**  
> 講義 p.38 寫 EF_KEYS「儲存行動網路加密金鑰 K、CK、IK」，p.48 也稱 EF_KEYS「承載用於 AKA 計算的主金鑰 (K)」；但 p.48 同頁的表格（引 TS 31.102 §4.2.19）列出 EF_KEYS 的資料結構只有 `[CK (16 bytes)] + [IK (16 bytes)]`。TS 33.501 只規定「USIM 儲存 $K$」[§6.2.2.2]，沒有說 $K$ 放在可讀取的 EF 中。TS 31.102 不在本庫 `raw/` 內，無法逐字查證；簡報中建議只說「$K$ 存在 USIM 內、不可讀出」，不要說「$K$ 存在 EF_KEYS」。

---

## 🔄 關聯介面與通訊對象
| 介面 / 服務介面 | 對端實體 | 主要安全信令 / 傳輸內容 | 參考規範條款 |
|---|---|---|---|
| USIM ↔ ME（UE 內部） | ME（手機本體） | USIM 回傳 RES、CK、IK；ME 據此算 RES*、$K_{AUSF}$ | TS 33.501 §6.1.3.2.0 step 7, §6.2.2.2 |
| N1 NAS（經 ME 轉送） | [[SEAF]] / [[AMF]] | 接收 RAND、AUTN（Authentication Request） | TS 33.501 §6.1.3.2.0 step 6–7 |
| 長期信任關係（不經網路傳送） | [[ARPF]] | 雙方各自持有同一把 $K$ | TS 33.501 §6.2.2.2 |

---

## 🛡️ 安全防禦目標與威脅
| 威脅 | 說明 | 防禦方向 | 出處 |
|---|---|---|---|
| IMSI Catcher | 假基地台攔截明文 IMSI | 5G 以 [[SUCI]] 加密傳送身分 | [講義: 5G資通-Week3, Week4.pdf, p.58] |
| SIM Cloning | 攻擊者複製 SIM 金鑰資料 | USIM/ISIM 晶片防護、HSM 金鑰管理 | [講義: 5G資通-Week3, Week4.pdf, p.58] |
| SIM Swap Fraud | 社交工程騙電信商換卡 | 強化用戶身分驗證（MFA） | [講義: 5G資通-Week3, Week4.pdf, p.58] |
| Side Channel Attack | 分析晶片功耗窺取金鑰 | 晶片級防護與隨機遮罩運算 | [講義: 5G資通-Week3, Week4.pdf, p.58] |
| eSIM 遠端配置風險 | 供應鏈、配置憑證管理、Profile 竄改 | 使用受信任的 SM-DP+，強化簽章驗證 | [講義: 5G資通-Week3, Week4.pdf, p.67] |

> 📰 **真實案例：SK Telecom USIM 資料外洩（講義 W2 p.70 引述之新聞）**  
> 南韓電信龍頭 SK Telecom 證實內部網路遭植入惡意程式，攻擊者竊走用戶 USIM 卡相關資料；公司宣布提供所有用戶申請更換新 SIM 卡，當地媒體估計逾 2,500 萬用戶符合換卡資格。第二次調查確認外流 9.82 GB 檔案，內含 26,957,749 個 IMSI；清查後發現最早的 Web Shell 出現在 2022 年 6 月 15 日，駭客已潛伏近 3 年 [講義: 5G 資通安全技術概論-W2.pdf, p.70]。  
> **與本頁的關聯**：這起事件攻擊的不是空中介面，而是電信商**核心網端**保存的用戶資料；公司的補救方式是讓用戶「換卡」（講義未說明外洩資料是否包含金鑰 $K$，也未說明換卡的技術理由；合理推測是讓外洩的卡片資料失效，此為推論）。這也呼應 [[SIDF]] 頁「私鑰外洩則 SUCI 保護失效」的討論：密碼學保護的前提是金鑰沒有外洩。

> 💡 **簡報可用的重點**：SUCI 保護的是「空中傳送的身分」，USIM 保護的是「金鑰與身分的保存處」。SIM Swap 這類攻擊根本不碰密碼學，而是騙電信商的客服——這提醒觀眾，5G 的密碼設計再好，也擋不住流程與社交工程的漏洞。

---

## 🔗 關聯概念與流程
- 身分：[[SUPI]]、[[SUCI]]、[[ECIES]]
- 金鑰：[[Key-Hierarchy]]、[[ARPF]]
- 流程：[[5G-AKA]]
- 世代演進：[[Cellular-Generations-Security]]

---

## 📚 參考文獻與條款溯源
- 3GPP TS 33.501 Rel-18:
  - §5.2.5 (Subscriber privacy：USIM 必須儲存的 SUCI 參數)
  - §6.1.1.1 (General：USIM shall reside on a UICC)
  - §6.1.3.2.0 (5G AKA：步驟 6–7 USIM 驗證 AUTN 並回傳 RES、CK、IK)
  - §6.2.2.2 (Keys in the UE：Keys in the USIM / Keys in the ME)
  - §6.12.2 (Subscription concealed identifier：USIM / ME 計算 SUCI)
- 課程講義來源: `raw/lectures/5G資通-Week3, Week4.pdf`（p.27–31、p.33、p.35、p.37–38、p.43、p.48、p.55、p.58、p.67）；`raw/lectures/5G 資通安全技術概論-W2.pdf`（p.70，SK Telecom 案例）
- 外部規範（未收錄於 raw/，未逐字查證）：3GPP TS 31.102（USIM Application）
