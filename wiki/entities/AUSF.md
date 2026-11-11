# AUSF - Authentication Server Function

> **中文譯名**：認證伺服器功能  
> **所屬網域**：Home Network（歸屬網路）

---

## 📌 實體角色概述
AUSF 是 5G 核心網歸屬網路中代表營運商執行認證裁決的「最高法官」。它是 5G 實現 **Home Control（歸屬網路控制權）** 的核心執行者，終結了 4G 時代拜訪網路得以自行判定認證成敗的漏洞。不論用戶漫遊至世界何處，認證的最終判定權與錨點金鑰的核發權，全由 AUSF 牢牢把守 [3GPP TS 33.501 Rel-18 §4.3, §6.1.3.2; 講義: 5G安全_TS33501-Week4.pdf, p.7, 9]。

---

## 🔐 核心安全職責 (Security Responsibilities)

1. **Home Control 最終裁決者**：  
   在 [[5G-AKA]] 流程中，拜訪網路 [[SEAF]] 收到終端回應的 $RES^*$ 後，必須將其原封不動送交 AUSF。AUSF 將 $RES^*$ 與自身妥善保存的 $XRES^*$ 執行完全位元比對。**只有 AUSF 裁定成功，本次認證才具法律與計費效力** [TS 33.501 Rel-18 §6.1.3.2; 講義: 5G安全_TS33501-Week4.pdf, p.17]。
2. **認證向量轉換與初篩雜湊 ($HXRES^*$) 生成**：  
   - AUSF 自 [[UDM]]/[[ARPF]] 取得包含真實答案 $XRES^*$ 的 5G HE AV。
   - AUSF **扣留 $XRES^*$ 不交給拜訪網路**，改以單向雜湊運算：
     $$HXRES^* = \text{SHA-256}(RAND \parallel XRES^*)[128..255]$$
   - 將 $(RAND, AUTN, HXRES^*)$ 組裝為 5G Serving Environment AV 發給 [[SEAF]]，使拜訪網路具備初篩錯誤回應的能力，但無法冒充歸屬網路偽造成功認證 [TS 33.501 Rel-18 §6.1.3.2, Annex A.5; 講義: 5G安全_TS33501-Week4.pdf, p.17, 18]。
3. **錨點金鑰推導與釋放 ($K_{SEAF}$)**：  
   AUSF 根據 $K_{AUSF}$ 與服務網路名稱 $SN\text{-}name$，推導出拜訪網路的錨點金鑰 $K_{SEAF}$ [TS 33.501 Rel-18 Annex A.6]。**AUSF 在認證尚未成功前會死守 $K_{SEAF}$ 與 [[SUPI]]**，直至比對無誤後才將兩者同步下發給 SEAF [TS 33.501 Rel-18 §6.1.3.2; 講義: 5G安全_TS33501-Week4.pdf, p.17]。
4. **支援多元 EAP 認證**：  
   若網路裁定採用 [[EAP-AKA-prime]] 或非 3GPP 專網憑證認證（EAP-TLS），AUSF 則轉型為完整的 **EAP 認證伺服器 (EAP Server)**，終止 EAP 協定狀態機 [TS 33.501 Rel-18 §6.1.3.1; 講義: 5G安全_TS33501-Week4.pdf, p.19]。

---

## 🔄 關聯介面與通訊對象

| 介面 / 服務介面 | 對端實體 | 主要安全信令 / 傳輸內容 | 參考規範條款 |
|---|---|---|---|
| **Nausf_UEAuthentication** | [[AMF]] / [[SEAF]] | 接收拜訪網認證發起請求；回傳認證挑戰 $(RAND, AUTN, HXRES^*)$；接收 $RES^*$ 並回傳最終結果、$K_{SEAF}$ 與 $SUPI$ | TS 33.501 §6.1.3.2 |
| **Nudm_UEAuthentication** | [[UDM]] | 請求產生認證向量（攜帶 SUCI 與 SN-name）；接收 5G HE AV $(RAND, AUTN, XRES^*, K_{AUSF})$ | TS 33.501 §6.1.3.2 |

---

## 📚 參考文獻與條款溯源 (References & Traceability)
- 3GPP TS 33.501 Rel-18:
  - §4.3 (Security entities in the 5G Core network)
  - §6.1.3.1 (EAP-based authentication method)
  - §6.1.3.2 (5G AKA authentication procedure)
  - Annex A.5 ($HXRES^* / HRES^*$ derivation)
  - Annex A.6 ($K_{SEAF}$ derivation)
- 課程講義來源: `raw/lectures/5G安全_TS33501-Week4.pdf` (第 1 章 p.7、第 2 章 p.9、第 4 章 p.15-18)
