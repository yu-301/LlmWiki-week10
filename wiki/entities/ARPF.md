# ARPF - Authentication Credential Repository and Processing Function

> **中文譯名**：認證憑證儲存與處理功能  
> **所屬網域**：Home Network（歸屬網路，位於 UDM 實體安全隔離區）

---

## 📌 實體角色概述
ARPF 是 5G 核心網中最深層、機密等級最高的「密碼保險金庫」。它負責長期保管終端用戶的主對稱金鑰 $K$，並負責執行底層密碼學演算法集以產生認證向量（AV）。3GPP 規範明訂 ARPF 必須部署於具備高度實體防護的安全環境中，防止任何未經授權的物理存取或側信道攻擊 [3GPP TS 33.501 Rel-18 §4.3, §5.2.4; 講義: 5G安全_TS33501-Week4.pdf, p.9]。

---

## 🔐 核心安全職責 (Security Responsibilities)

1. **長期密鑰 $K$ 保管**：  
   長期對稱金鑰 $K$ 僅同時存放於用戶的 [[USIM]] 晶片硬體與歸屬網路的 ARPF 中，**終其一生絕對不會在任何實體或無線網路上傳輸** [TS 33.501 Rel-18 §6.1.1; 講義: 5G安全_TS33501-Week4.pdf, p.15]。
2. **認證向量計算 (Vector Generation)**：  
   - 採用標準密碼演算法集（如 **MILENAGE** 或 **TUAK**），依據主金鑰 $K$、隨機數 $RAND$ 與當前序號 $SQN$ 運算出中間參數：$AUTN, CK, IK, XRES$ [TS 33.501 Rel-18 §6.1.1, §6.1.3.2]。
   - **AUTN (Authentication Token)**：組裝 $(SQN \oplus AK) \parallel AMF \parallel MAC$，供 UE 驗證網路端是否真實持有金鑰 $K$。
3. **5G 金鑰升級推導 ($K_{AUSF}$ 與 $XRES^*$)**：  
   - 將傳統中間金鑰 $CK \parallel IK$ 結合服務網路名稱 $SN\text{-}name$，藉由通用 KDF 推導出 5G 專屬的期望回應值 $XRES^*$ 與歸屬金鑰 $K_{AUSF}$ [TS 33.501 Rel-18 §6.1.3.2, Annex A.2, A.4]。
   - **關鍵安全隔離**：$CK$ 與 $IK$ 被強制封閉在 ARPF/UDM 內部，**絕不交給外部拜訪網路**，對外僅交付推導後的 $K_{AUSF}$ [講義: 5G安全_TS33501-Week4.pdf, p.16]。
4. **防重放序列號 ($SQN$) 維護**：  
   維護每一個用戶簽約的序列號 $SQN$，在每次產生認證向量時遞增，確保認證訊息的新鮮度（Freshness）。

---

## 🔄 關聯介面與通訊對象

| 介面 / 服務介面 | 對端實體 | 主要安全信令 / 傳輸內容 | 參考規範條款 |
|---|---|---|---|
| **UDM 內部介面** | [[UDM]] | 接收向量生成請求；回傳 5G HE 認證向量 $(RAND, AUTN, XRES^*, K_{AUSF})$ | TS 33.501 §4.3, §6.1.3.2 |
| **無外部網路暴露** | 無 | 長期金鑰不得以未保護形式離開 UDM/ARPF 的安全環境；規格在 5G AKA 流程中以 "UDM/ARPF" 合稱，對外由 UDM 的 Nudm 服務代表（TS 33.501 未定義 ARPF 自己的服務化 API） | TS 33.501 §5.8.1, §6.1.3.2 |

---

## 📚 參考文獻與條款溯源 (References & Traceability)
- 3GPP TS 33.501 Rel-18:
  - §4.3 (Security entities in the 5G Core network)
  - §5.2.4 (Secure storage and processing of subscription credentials)
  - §6.1.1 (General authentication requirements)
  - §6.1.3.2 (5G AKA authentication procedure)
  - Annex A.2 ($K_{AUSF}$ derivation), Annex A.4 ($RES^* / XRES^*$ derivation)
- 3GPP TS 35.205 ~ TS 35.208: Specification of the MILENAGE Algorithm Set
- 課程講義來源: `raw/lectures/5G安全_TS33501-Week4.pdf` (第 2 章 p.9、第 4 章 p.15-16)
