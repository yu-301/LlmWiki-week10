# EAP-AKA' 認證機制 (EAP-AKA Prime)

> **一句話直觀定義**：EAP-AKA' 是將行動網路 USIM 對稱金鑰認證機制封裝於 IETF 標準 EAP（可延伸認證協定）框架中的認證協定，主要用於 5G 與非 3GPP 網路（如 Wi-Fi、有線寬頻）的統一接取認證 [RFC 9048; 3GPP TS 33.501 Rel-18 §6.1.3.1]。

---

## 💡 直觀原理解析 (Beginner-Friendly Explanation)

- **生活化比喻**：**「同一張門禁卡，換上國際標準外殼」**。5G-AKA 是電信業者自家專利規格的讀卡機（走 NAS 訊令）；而 EAP-AKA' 是為這張卡套上全球通用的國際標準轉接頭（EAP 協定包裝）。無論手機是透過 5G 基地台還是路邊的 Wi-Fi 熱點連線，只要插著 SIM 卡，就能以相同的密碼學根基通過驗證。
- **為什麼需要它？**：5G 不只涵蓋蜂巢式行動基地台，還必須原生支援非 3GPP 接取（Non-3GPP Access，如 Wi-Fi 透過 [[N3IWF]]、固定有線網路）。這些網路不具備 3GPP 專屬的 NAS 協定通道，因此必須採用 IETF 國際標準的 EAP 框架進行身分認證。

---

## ⚙️ 技術機制與 5G-AKA 對比

### 核心運作機制
1. **信任根基相同**：與 [[5G-AKA]] 相同，依然以 [[USIM]] 與 [[ARPF]] 中儲存的長期對稱金鑰 $K$ 為唯一信任源頭。
2. **AUSF 轉型為 EAP 伺服器**：在 EAP-AKA' 流程中，[[AUSF]] 擔任標準的 **EAP Server**，完整運作 EAP 狀態機；拜訪網路 [[SEAF]] 則退化為純粹的透通轉送者（Pass-through Authenticator）[TS 33.501 Rel-18 §6.1.3.1]。
3. **金鑰推導收斂**：
   - 認證成功後，EAP 協議產出延伸主工作階段金鑰 $EMSK$（Extended Master Session Key）。
   - 依據 TS 33.501 Annex F / RFC 9048，取 $EMSK$ 的最高有效 256 位元作為 **$K_{AUSF}$**。
   - 隨後同樣推導出拜訪網路錨點金鑰 **$K_{SEAF}$**。

### 5G-AKA vs EAP-AKA' 速查表 [講義 p.19]

| 比較維度 | 5G-AKA | EAP-AKA' |
|---|---|---|
| **協定框架** | 3GPP 自訂之專屬 NAS 訊息交換 | IETF EAP 框架（RFC 9048），NAS 僅作為傳輸載體 |
| **典型應用場景** | 3GPP 蜂巢式行動網路（5G-NR gNB） | 非 3GPP 接取（Wi-Fi、有線）、SNPN 專網 |
| **AUSF 角色** | 認證伺服器（比對 $RES^*$ 與 $XRES^*$） | 完整 EAP 伺服器（執行 EAP 狀態機） |
| **SEAF 角色** | 初篩比對（計算 $HRES^*$ 比對 $HXRES^*$） | 純粹的透通中繼者（Pass-through Authenticator） |
| **訊息回合** | 固定一次挑戰／回應 | 可能包含多次 EAP-Request / EAP-Response 往返 |
| **金鑰收斂終點** | **$K_{SEAF}$（完全相同）** | **$K_{SEAF}$（完全相同）** |

---

## 🛡️ 設計價值：乾淨的解耦與分層

> 💡 **講義核心洞見**：  
> **「不論使用哪一種認證方法，程序結束時都會產生同一把錨點金鑰 $K_{SEAF}$」** [講義: 5G安全_TS33501-Week4.pdf, p.15]。  
> 這是 5G 架構把「**認證方法（Authentication Method）**」與「**金鑰使用（Key Usage）**」乾淨解耦的極佳工程範例。核心網後續的金鑰階層（[[Key-Hierarchy]]）、NAS/AS 加密以及換手流程完全不受底層認證協定影響。

---

## 🔗 關聯概念與流程
- 關聯實體：[[AUSF]], [[SEAF]], [[UDM]], [[ARPF]]
- 主要流程：[[5G-AKA]]
- 金鑰階層：[[Key-Hierarchy]]

---

## 📚 參考文獻與條款溯源 (References & Traceability)
- 3GPP TS 33.501 Rel-18:
  - §6.1.1.2 (EAP framework)
  - §6.1.3.1 (Authentication procedure for EAP-AKA')
- IETF RFC 9048: Improved Extensible Authentication Protocol Method for 3GPP Mobile Network Authentication and Key Agreement (EAP-AKA')
- 課程講義來源: `raw/lectures/5G安全_TS33501-Week4.pdf` (第 4 章 p.15, 18–19)
