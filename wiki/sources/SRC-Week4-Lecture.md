# 來源收據：3GPP TS 33.501 — 5G 行動網路安全架構 課程講義（Week4 導讀）

> **檔案位置**：`raw/lectures/5G安全_TS33501-Week4.pdf`  
> **文件性質**：大學部選修專題講座導讀教材，依 ETSI TS 133 501 V19.6.0 / 3GPP Release 19 編寫，全篇 34 頁。它不是規格翻譯，而是「導讀」：目標是讓讀者能自己打開 TS 33.501 找到章節並看懂 [講義 p.5]  
> **收納日期**：2026-11-10（階段一），2026-11-11（波次 1 與後續引用）  
> **收據建立**：2026-11-11，由舊收據 `SRC-TS33501-Week4` 拆出（原收據同時涵蓋規格與本講義，改為一份來源一張收據）。規格部分見 [[SRC-TS33501]]

---

## 📌 文件核心貢獻與定位
本講義提供從 2G/3G/4G 到 5G 安全演進的宏觀視角，並針對痛點說明「為什麼 5G 這樣設計」。在用戶隱私（SUPI / SUCI / ECIES）、主要認證（5G-AKA / EAP-AKA'）、金鑰階層與 NAS/AS 分層上有清楚的取捨分析與參數速查表。

> [!NOTE]
> **版本差異**：本講義依 Rel-19 撰寫，本庫的規格基準是 Rel-18 V18.12.0（[[SRC-TS33501]]）。條款編號或細節若有出入，以 Rel-18 為準。

---

## 🗺️ 章節、頁碼與熔接去向
| 章節 | 頁碼 | 核心主題 | 處理階段 | 熔接目標 wiki 頁面 |
|---|---|---|---|---|
| 給讀者的話 | p.5 | 讀規格的三個提醒（shall / should / may、版本、規格≠部署） | 引用 | [[SUPI]]（should vs shall NOTE） |
| 第 1 章 為什麼 5G 需要新的安全架構 | p.6–8 | 四個世代的安全演進、IMSI Catcher、5G 三個結構性改變、仍未解決的問題 | 階段一；後續 Week3/4、W2、簡報引用 | [[overview]], [[SUPI]], [[SUCI]], [[4G-vs-5G-AKA]], [[Cellular-Generations-Security]], [[AUSF]], [[AMF]], [[User-Plane-Integrity]], [[NSA-vs-SA-Security]] |
| 第 2 章 5G 安全架構總覽 | p.9–11 | 六大安全域、五個關鍵網路功能、SIDF 私鑰外洩思考、規格文件地圖 | 階段一 | [[SEAF]], [[AUSF]], [[UDM]], [[ARPF]], [[SIDF]], [[SUCI]] |
| 第 3 章 用戶身分隱私：SUPI 與 SUCI | p.12–14 | SUPI 兩種型態與三條鐵律、SUCI 欄位結構、ECIES 五步、Profile A/B、實作陷阱 | 階段一 | [[SUPI]], [[SUCI]], [[ECIES]], [[SIDF]] |
| 第 4 章 主要認證：5G-AKA 與 EAP-AKA' | p.15–19 | 認證方法、參數速查表、5G-AKA 十四步、AUTN 結構、兩道檢查、可連結性、三個回應值、EAP-AKA' 差異、與 4G EPS-AKA 對比 | 波次 1 | [[5G-AKA]], [[EAP-AKA-prime]], [[4G-vs-5G-AKA]], [[AUSF]], [[SEAF]], [[UDM]], [[ARPF]], [[UE]], [[USIM]], [[AMF]] |
| 第 5 章 金鑰階層與推導 | p.20–22 | 為什麼需要金鑰樹、階層結構、通用 KDF 與 FC、情境綁定、金鑰生命週期、安全上下文與 ngKSI | 波次 1 | [[Key-Hierarchy]], [[SEAF]] |
| 第 6 章 NAS 與 AS 安全機制 | p.23–26 | 兩層分工、NAS SMC、演算法家族與密碼敏捷性、Null 演算法爭議、防降級、UP 完整性、NH/NCC | 波次 2 及之後引用 | [[NAS-vs-AS-Security]], [[Security-Mode-Command]], [[AMF]], [[UE]], [[Air-Interface-Security]], [[Cellular-Generations-Security]], [[User-Plane-Integrity]] |
| 第 7 章 服務化架構（SBA）安全 | p.27–28 | TLS、OAuth 2.0、SEPP 與 N32 | **未 ingest**（與簡報主題關聯低） | — |
| 第 8 章 已知弱點與研究方向 | p.29–30 | 形式化驗證（Basin 等）、AKA 可連結性、偽基站、部署落差、尚未解決的問題、十個重點 | 部分引用 | [[5G-AKA]]（可連結性與 Basin 更正）, [[ECIES]]（後量子） |
| 附錄 A–C | p.31–34 | 中英術語對照、縮寫表、規格與延伸閱讀 | 未 ingest | — |

### 頁碼 → wiki 頁面（wiki 中實際引用到的頁碼）
> 由腳本掃描各頁的 `[講義: 5G安全_TS33501-Week4.pdf, p.X]` 引用產生（2026-11-11），並人工排除同一行中屬於其他講義的頁碼。

| 頁碼 | 引用頁面 |
|---|---|
| p.6 | [[4G-vs-5G-AKA]], [[Cellular-Generations-Security]], [[NSA-vs-SA-Security]], [[SUPI]], [[overview]] |
| p.7 | [[4G-vs-5G-AKA]], [[AMF]], [[AUSF]], [[Cellular-Generations-Security]], [[NSA-vs-SA-Security]], [[SUCI]], [[SUPI]], [[User-Plane-Integrity]], [[overview]] |
| p.8 | [[Cellular-Generations-Security]] |
| p.9 | [[ARPF]], [[AUSF]], [[SEAF]], [[UDM]] |
| p.10 | [[SIDF]], [[SUCI]] |
| p.12 | [[SIDF]], [[SUCI]], [[SUPI]] |
| p.13 | [[ECIES]], [[SIDF]], [[SUCI]] |
| p.14 | [[ECIES]], [[SIDF]], [[SUCI]] |
| p.15 | [[5G-AKA]], [[ARPF]], [[AUSF]], [[Cellular-Generations-Security]], [[EAP-AKA-prime]], [[UDM]] |
| p.16 | [[5G-AKA]], [[ARPF]], [[AUSF]], [[SEAF]], [[UDM]], [[UE]] |
| p.17 | [[5G-AKA]], [[AMF]], [[AUSF]], [[SEAF]], [[SUCI]], [[SUPI]], [[UE]], [[USIM]] |
| p.18 | [[5G-AKA]], [[AUSF]], [[EAP-AKA-prime]], [[SEAF]], [[UDM]], [[UE]] |
| p.19 | [[4G-vs-5G-AKA]], [[5G-AKA]], [[AUSF]], [[EAP-AKA-prime]] |
| p.20–22 | [[Key-Hierarchy]]（p.22 另有 [[SEAF]]） |
| p.23 | [[AMF]], [[NAS-vs-AS-Security]], [[Security-Mode-Command]], [[UE]] |
| p.24 | [[Air-Interface-Security]], [[Cellular-Generations-Security]], [[Security-Mode-Command]] |
| p.25 | [[Security-Mode-Command]], [[User-Plane-Integrity]] |
| p.29 | [[5G-AKA]], [[ECIES]] |

---

## ⚠️ 本講義與 TS 33.501 Rel-18 的已知分歧（查核時發現）
| 頁碼 | 講義寫法 | 規格 | 處理 |
|---|---|---|---|
| p.12 | 「SUPI **不得**以明文透過空中介面傳送」 | §5.2.5 原文為 **should not** | [[SUPI]] 已加 NOTE |
| p.17 | HRES* 不符時「SEAF 可立刻拒絕」 | §6.1.3.2.2：SEAF 仍須把 RES* 送給 AUSF，等回覆才拒絕 | [[5G-AKA]]、[[SEAF]] 已加 WARNING |

---

## 💡 萃取之關鍵洞見 (Key Takeaways)
1. **身分隱私與可路由性的權衡**：SUPI 加密成 SUCI 時只加密 MSIN，必須保留明文的 MCC/MNC，否則拜訪網路無法把認證請求路由回歸屬網路 [講義 p.12–13]。
2. **認證與隱私的獨立性**：SIDF 私鑰外洩只影響身分隱私（退回 4G 水準），不影響認證本身，因為認證依賴長期金鑰 $K$，而 $K$ 從未離開 USIM 與 ARPF [講義 p.10]。
3. **不可連結性的關鍵依賴**：ECIES 每次都要產生全新的臨時金鑰對；若亂數產生器有瑕疵導致金鑰可預測或重複，SUCI 機制形同虛設 [講義 p.13–14]。
4. **Home Control 的權威與效率**：XRES*（AUSF 的權威答案）、HXRES*（交給 SEAF 的雜湊）、RES*（UE 的證明）。「SEAF 的比對是效率，AUSF 的比對是權威」[講義 p.18]（失敗時的處理見上方分歧表）。
5. **金鑰樹的情境鎖死**：每往下一層就多綁一個情境參數（SN-name、SUPI 與 ABBA、NAS COUNT、演算法識別碼、PCI 與 ARFCN）[講義 p.21]。
6. **認證方法與金鑰使用解耦**：不論 5G-AKA 或 EAP-AKA'，最後都產生同一把錨點金鑰 $K_{SEAF}$ [講義 p.15]。

---

## 📚 參考來源對照
- `raw/lectures/5G安全_TS33501-Week4.pdf`（使用 p.5–25、p.29；第 7 章與附錄未使用）
- 對應規格收據：[[SRC-TS33501]]
