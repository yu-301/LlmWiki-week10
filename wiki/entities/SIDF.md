# SIDF - Subscription Identifier De-concealing Function

> **中文譯名**：用戶身分識別解密功能（訂閱識別碼解隱蔽功能）  
> **所屬網域**：Home Network（歸屬網路，UDM 內部安全微服務）

---

## 📌 實體角色概述
SIDF 是 5G 核心網中專門負責**「解開使用者隱蔽身分」**的最高權限密碼學模組。它座落於歸屬網路（Home Network）之 [[UDM]] 內部，是全網中**唯一持有並有權使用營運商私鑰（Home Network Private Key）**將 [[SUCI]] 解密還原為 [[SUPI]] 的專責實體 [3GPP TS 33.501 Rel-18 §4.3, §6.12.5; 講義: 5G安全_TS33501-Week4.pdf, p.10]。

---

## 🔐 核心安全職責 (Security Responsibilities)

1. **SUCI 解隱蔽運算 (De-concealment)**：  
   當拜訪網路轉發包含 SUCI 的認證請求到達 UDM 時，SIDF 根據 SUCI 中的 `Home Network Public Key ID` 與 `Protection Scheme ID`，調用本機保存的私鑰，執行 [[ECIES]] 解密運算還原出真正的 [[SUPI]] [TS 33.501 Rel-18 §5.8.2, §6.12.5]。
2. **私鑰最高防護（絕不離開歸屬網）**：  
   3GPP 規範明訂：用於用戶隱私保護的歸屬網路私鑰必須在 UDM 中受到防實體攻擊的保護，且隱私演算法必須在 UDM 的安全環境中執行 [TS 33.501 Rel-18 §5.8.2]；講義補充「私鑰絕不離開歸屬網路」[講義: 5G安全_TS33501-Week4.pdf, p.10]。
3. **嚴格存取權限控制**：  
   只有歸屬網路內部的合法網元（如 UDM 本身）被允許向 SIDF 發起解密呼叫，拜訪網路（Serving Network）絕無法繞過 UDM 直接向 SIDF 索取資料 [TS 33.501 Rel-18 §6.12.5]。

---

## 💡 深度架構洞見：若 SIDF 私鑰外洩會發生什麼？

> [!NOTE] **「隱私」與「認證」的獨立解耦設計**  
> 如果某天電信商的 SIDF 私鑰不幸外洩，後果是：攻擊者能將監聽到的所有過去與未來的 SUCI 還原成 SUPI，**身分隱私保護退回到 4G 裸奔水準**。  
> **但是，網路認證依然固若金湯！**  
> 因為 5G 認證所依賴的是燒錄在 USIM 與 [[ARPF]] 內的長期主對稱金鑰 $K$，而 $K$ 完全獨立於 SIDF 的非對稱私鑰。這證明了 5G 在架構上將「用戶身分隱私」與「網路存取認證」完美解耦為兩套互不影響的獨立防線 [講義: 5G安全_TS33501-Week4.pdf, p.10]。

---

## 🔄 關聯介面與通訊對象

| 介面 / 服務介面 | 對端實體 | 主要安全信令 / 傳輸內容 | 參考規範條款 |
|---|---|---|---|
| **UDM 內部呼叫** | [[UDM]] | 接收 SUCI，回傳解密後的 [[SUPI]]（限本機處理） | TS 33.501 §5.8.2, §6.12.5 |
| **Nausf / Nudm 服務化介面** | [[AUSF]] $\to$ [[UDM]] | AUSF 發送 `Nudm_UEAuthentication_Get` 攜帶 SUCI，觸發 UDM 呼叫內部 SIDF | TS 33.501 §6.1.3.2 |

---

## 📚 參考文獻與條款溯源 (References & Traceability)
- 3GPP TS 33.501 Rel-18:
  - §4.3 (Security entities in the 5G Core network)
  - §5.8.2 (Subscriber privacy related requirements to UDM and SIDF)
  - §6.12.5 (Subscription identifier de-concealing function)
- 課程講義來源: `raw/lectures/5G安全_TS33501-Week4.pdf` (第 2 章 p.10、第 3 章 p.12-14)
