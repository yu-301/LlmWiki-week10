# SEAF - Security Anchor Function

> **中文譯名**：安全錨點功能  
> **所屬網域**：Serving Network（服務網路 / 拜訪網路，實務上與 AMF 併置）

---

## 📌 實體角色概述
SEAF 是 5G 安全架構中拜訪網路（Serving Network）的「前線安全代表」。它實體上與 [[AMF]]（存取與行動性管理功能）共存於同一節點。SEAF 負責代表拜訪網路向歸屬網路發起認證請求、執行終端回應的初步篩檢，並在認證通過後持有**錨點金鑰 $K_{SEAF}$**，做為拜訪網路推導所有後續金鑰的唯一信任源頭 [3GPP TS 33.501 Rel-18 §4.3; 講義: 5G安全_TS33501-Week4.pdf, p.9]。

---

## 🔐 核心安全職責 (Security Responsibilities)

1. **代表拜訪網路發起認證**：  
   當終端透過無線介面發送 `Registration Request` 時，SEAF 攔截該請求，為其附加上自身拜訪網路的官方服務名稱 **$SN\text{-}name$**（格式為 `5G:mnc<MNC>.mcc<MCC>.3gppnetwork.org`），向歸屬網路 [[AUSF]] 請求認證 [TS 33.501 Rel-18 §6.1.3.2; 講義: 5G安全_TS33501-Week4.pdf, p.16]。  
   > **防偽冒機制**：$SN\text{-}name$ 會被深度綁入後續所有金鑰與認證回應的推導，徹底防止漫遊網路冒用其他業者的身分 [講義: 5G安全_TS33501-Week4.pdf, p.16]。
2. **高效能初篩比對 ($HRES^*$ 比對)**：  
   - 終端回傳 $RES^*$ 後，SEAF 依照 **TS 33.501 Annex A.5** 計算單向雜湊：
     $$HRES^* = \text{SHA-256}(RAND \parallel RES^*)[128..255]$$
   - 將計算結果與 AUSF 預先提供的 $HXRES^*$ 進行本地比對。若不符，SEAF 判定認證失敗（從服務網路角度），但**依規格仍須繼續把 $RES^*$ 送給 AUSF**，收到 AUSF 回覆後才拒絕 UE 或發起 Identification 程序 [TS 33.501 Rel-18 §6.1.3.2.2]。

   > [!WARNING]
   > 講義 p.17 寫「若不符，SEAF 可立刻拒絕。這是效率設計」，與 TS 33.501 Rel-18 §6.1.3.2.2（"The SEAF shall proceed with step 10"）不符。HXRES* 的價值在於讓拜訪網路有自己的判定依據、不必盲信，而不是省下跨網往返。（2026-11-11 Claude Code 查核補記）
   - **安全哲學**：講義精闢總結——**「SEAF 的比對是效率，AUSF 的比對是權威」** [講義: 5G安全_TS33501-Week4.pdf, p.18]。
3. **錨點金鑰維護與 $K_{AMF}$ 推導**：  
   - 認證完全通過後，SEAF 接收由 AUSF 核發的錨點金鑰 $K_{SEAF}$ 與真正的用戶身分 [[SUPI]] [TS 33.501 Rel-18 §6.1.3.2]。
   - 在同一次註冊週期中，SEAF 透過安全上下文識別碼（[[ngKSI]]）重複使用該錨點狀態，避免終端每次喚醒都必須重新執行龐大的全網認證 [講義: 5G安全_TS33501-Week4.pdf, p.9, 22]。
   - 結合用戶真實身分 $SUPI$ 與防架構降級參數 $ABBA$，推導出行動管理金鑰 $K_{AMF}$，交由 AMF 啟動 NAS 安全模式命令（NAS SMC）[TS 33.501 Rel-18 Annex A.7; 講義: 5G安全_TS33501-Week4.pdf, p.17]。

---

## 🔄 關聯介面與通訊對象

| 介面 / 服務介面 | 對端實體 | 主要安全信令 / 傳輸內容 | 參考規範條款 |
|---|---|---|---|
| **N1 (NAS 介面)** | [[UE]] | 傳送 `NAS Authentication Request` $(RAND, AUTN, ngKSI, ABBA)$；接收 `NAS Authentication Response` $(RES^*)$ | TS 33.501 §6.1.3.2 |
| **Nausf_UEAuthentication** | [[AUSF]] | 發送認證請求 `Nausf_UEAuthentication_Authenticate`；接收 5G Serving AV；轉發 $RES^*$；接收最終裁定、$K_{SEAF}$ 與 $SUPI$ | TS 33.501 §6.1.3.2 |
| **內部調用** | [[AMF]] | 將推導出的 $K_{AMF}$ 交付給 AMF 核心控制層 | TS 33.501 §6.2.1 |

---

## 📚 參考文獻與條款溯源 (References & Traceability)
- 3GPP TS 33.501 Rel-18:
  - §4.3 (Security entities in the 5G Core network)
  - §6.1.3.2 (5G AKA authentication procedure)
  - Annex A.5 ($HXRES^* / HRES^*$ derivation)
  - Annex A.7 ($K_{AMF}$ derivation)
- 課程講義來源: `raw/lectures/5G安全_TS33501-Week4.pdf` (第 2 章 p.9、第 4 章 p.16-18)
