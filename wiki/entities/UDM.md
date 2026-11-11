# UDM - Unified Data Management

> **中文譯名**：統一資料管理功能  
> **所屬網域**：Home Network（歸屬網路）

---

## 📌 實體角色概述
UDM 是 5G 核心網歸屬網路中的用戶資料總管，扮演類似 4G HSS（Home Subscriber Server）的核心角色。在安全架構中，UDM 負責託管用戶的訂閱資料，並在實體架構中內嵌包含 [[SIDF]]（身分隱私解密）與 [[ARPF]]（憑證與認證向量運算）兩大核心安全功能，是決定用戶認證權限的總閘門 [3GPP TS 33.501 Rel-18 §4.3; 講義: 5G安全_TS33501-Week4.pdf, p.9]。

---

## 🔐 核心安全職責 (Security Responsibilities)

1. **認證方法裁決 (Authentication Method Selection)**：  
   當收到認證請求時，UDM 依據簽約資料（Subscription Data）裁決該用戶應使用何種認證方式（如主要認證 [[5G-AKA]]、[[EAP-AKA-prime]] 或專網的 EAP-TLS）[TS 33.501 Rel-18 §6.1.2; 講義: 5G安全_TS33501-Week4.pdf, p.9, 16]。
2. **協調身分還原與向量產生**：  
   - 收到 [[SUCI]] 後，調用內部 [[SIDF]] 解密還原出 [[SUPI]] [TS 33.501 Rel-18 §6.12.5]。
   - 指示內部 [[ARPF]] 根據長期主金鑰 $K$ 與目前序號 $SQN$，執行 MILENAGE/TUAK 演算法計算認證向量 [TS 33.501 Rel-18 §6.1.3.2]。
3. **交付 5G 歸屬環境認證向量 (5G HE AV)**：  
   透過服務化 API `Nudm_UEAuthentication_Get` 將 5G Home Environment Authentication Vector（內含 $RAND, AUTN, XRES^*, K_{AUSF}$）以及還原後的 $SUPI$ 回傳給 [[AUSF]] [TS 33.501 Rel-18 §6.1.3.2; 講義: 5G安全_TS33501-Week4.pdf, p.16]。
4. **序號同步管理 (SQN Synchronization)**：  
   若 UE 發現序號超出合理範圍回報同步失敗（附帶 $AUTS$ 參數），SEAF 經 AUSF 把 RAND/AUTS 送回 UDM/ARPF，由其重新同步伺服器端的 $SQN$ 並產生新的認證向量 [TS 33.501 Rel-18 §6.1.3.3.2; 講義: 5G安全_TS33501-Week4.pdf, p.18]。

---

## 🔄 關聯介面與通訊對象

| 介面 / 服務介面 | 對端實體 | 主要安全信令 / 傳輸內容 | 參考規範條款 |
|---|---|---|---|
| **Nudm_UEAuthentication** | [[AUSF]] | 接收認證向量請求（含 SUCI、SN-name）；回傳 5G HE AV 與解密後之 SUPI | TS 33.501 §6.1.3.2 |
| **內部整合調用** | [[SIDF]] / [[ARPF]] | 呼叫 SIDF 進行解密；呼叫 ARPF 進行向量運算 | TS 33.501 §4.3, §5.8.2 |
| **Nudr_DataRepository** | UDR (資料儲存庫) | 存取長期簽約資訊、認證方法設定與金鑰相關元數據 | TS 23.501 / TS 33.501 §6.1 |

---

## 📚 參考文獻與條款溯源 (References & Traceability)
- 3GPP TS 33.501 Rel-18:
  - §4.3 (Security entities in the 5G Core network)
  - §5.8.2 (Subscriber privacy related requirements to UDM and SIDF)
  - §6.1.2 (Authentication methods)
  - §6.1.3.2 (5G AKA authentication procedure)
- 課程講義來源: `raw/lectures/5G安全_TS33501-Week4.pdf` (第 2 章 p.9、第 4 章 p.15-16)
