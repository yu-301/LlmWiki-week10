# 5G 網路安全架構與認證機制知識庫 - 總索引 (Catalog Index)

> 本文件為知識庫之主目錄，由維護 Agent 在每次 Ingest 或建立新頁面時同步更新。各專題依照 5G 安全架構層級分類，並支援雙向檢索。

---

## 🗺️ 全景導覽 (Overview & Foundation)
- [[wiki/overview|5G 安全架構全景導航]] - 系統性認識 5G 安全域、核心網元角色與金鑰體系的全貌（含「身分保護 → 認證 → 金鑰 → 空中介面保護 → 剩餘空窗」學習路徑）。

---

## 🔑 核心概念 (Concepts)
- [[wiki/concepts/SUPI|SUPI]] - 用戶永久識別碼 (Subscription Permanent Identifier)，5G 終端身分標識與三條規範鐵律 [TS 33.501 §5.2.5, §6.12.1]
- [[wiki/concepts/SUCI|SUCI]] - 用戶隱蔽識別碼 (Subscription Concealed Identifier)，公鑰加密防偽基站追蹤的動態化身 [TS 33.501 §6.12.2]
- [[wiki/concepts/ECIES|ECIES]] - 橢圓曲線整合加密方案，Profile A/B 參數、5 步混合加密演算法與抗量子展望 [TS 33.501 Annex C]
- [[wiki/concepts/Key-Hierarchy|Key-Hierarchy]] - 5G 金鑰推導樹體系架構，通用 KDF 公式、FC 領域分離與情境參數鎖死，以及換手時水平/垂直推導、Xn vs N2 前向安全 [TS 33.501 §6.2, §6.9.2.3, Annex A]
- [[wiki/concepts/EAP-AKA-prime|EAP-AKA-prime]] - EAP-AKA' 認證機制概述，非 3GPP 接入、IETF 框架封裝與 K_SEAF 金鑰收斂 [TS 33.501 §6.1.3.1; RFC 9048]
- [[wiki/concepts/Air-Interface-Security|Air-Interface-Security]] - 5G 空中介面安全架構，PDCP 串流加密 (128-NEA) 與完整性 (128-NIA)、COUNT 繞回防護與金鑰流重用防範 [TS 33.501 §6.5; TS 38.323]
- [[wiki/concepts/User-Plane-Integrity|User-Plane-Integrity]] - 使用者平面完整性保護 (UP-IP)，SMF Policy 三選一、aLTEr/IMP4GT 位元翻轉攻擊防禦與規格部署落差 [TS 33.501 §6.6.1]
- [[wiki/concepts/False-Base-Station-Defenses|False-Base-Station-Defenses]] - 偽基地台與未受保護訊息防禦，階段 0–2 結構性空窗、Overshadowing 訊號覆蓋、Paging 側通道與 TR 33.809 對策 [TR 33.809]；含安全生效時間軸、結構性 vs 政策性、功能下移與 LTM、降級四型、低層控制攻擊、TR 33.809 候選方案、研究方向
- [[wiki/concepts/Research-Ethics|Research-Ethics]] - 行動網路安全研究的實驗環境與倫理界線（srsRAN / OAI / Open5GS、遮蔽室、負責任揭露、台灣相關法規）

---

## 🏛️ 網路網元與功能實體 (Entities & Network Functions)
*涵蓋歸屬網路 (Home Network)、服務網路 (Serving Network) 與終端*
- [[wiki/entities/SIDF|SIDF]] - 用戶身分識別解密功能，UDM 內唯一解密 SUCI 的最高權限密碼模組 [TS 33.501 §4.3, §6.12.5]
- [[wiki/entities/UDM|UDM]] - 統一資料管理，掌控用戶訂閱資料與認證方法裁決的總閘門 [TS 33.501 §4.3, §6.1.2]
- [[wiki/entities/ARPF|ARPF]] - 認證憑證儲存與處理功能，保管長期主金鑰 K 並執行向量運算的密碼金庫 [TS 33.501 §4.3, §6.1.1]
- [[wiki/entities/AUSF|AUSF]] - 認證伺服器功能，代表歸屬網路執行 Home Control 最終比對的最高法官 [TS 33.501 §4.3, §6.1.3.2]
- [[wiki/entities/SEAF|SEAF]] - 安全錨點功能，拜訪網路前線代表，負責效能初篩與錨點金鑰維護 [TS 33.501 §4.3, §6.1.3.2]
- [[wiki/entities/AMF|AMF]] - 接入與移動管理功能，與 SEAF 同處一地；NAS 安全終點，產生 K_NAS、K_gNB、NH 並負責 SUCI 觸發認證與 SUPI 確認 [TS 33.501 §5.5, §6.2.2.1]
- [[wiki/entities/USIM|USIM]] - 通用用戶身分模組，保管 IMSI/SUPI、SUCI 參數與信任根 K；檔案系統、存取權限與 SIM Cloning / SIM Swap 威脅 [TS 33.501 §5.2.5, §6.2.2.2]
- [[wiki/entities/UE|UE]] - 使用者設備＝手機本體 ME + USIM；SUCI 發送、驗證網路（AUTN）、只交 RES* 不交金鑰、檢查 SMC 重播能力 [TS 33.501 §6.2.2.2]

---

## 🔄 認證與信令流程 (Procedures & Call Flows)
- [[wiki/procedures/5G-AKA|5G-AKA]] - 5G 主要認證與金鑰協商流程（14 步信令時序、AUTN 結構與兩道檢查、三階比對體系）[TS 33.501 §6.1.3.2]
- [[wiki/procedures/Security-Mode-Command|Security-Mode-Command]] - NAS 與 AS 安全模式控制程序（兩階段演算法協商、能力原封重播防降級、先整後密原理）[TS 33.501 §6.7.2, §6.7.4]
- `[[SUCI-Deconcealment]]` - SUPI 隱蔽與 SIDF 解密流程 *(待後續深化)*

---

## ⚖️ 橫向對比分析 (Comparisons & Evolution)
- [[wiki/comparisons/4G-vs-5G-AKA|4G-vs-5G-AKA]] - 4G EPS-AKA 與 5G-AKA 認證安全橫向對比（Home Control、身分隱私、SN-name 綁定與完整性防護）[TS 33.401; TS 33.501 §6.1, §6.12]
- [[wiki/comparisons/Cellular-Generations-Security|Cellular-Generations-Security]] - 2G → 5G 安全演進對比（認證方式、身分傳送、各世代加密/完整性演算法、每一代補的洞）[TS 33.501 §5.2.2, §5.2.3, Annex D]
- [[wiki/comparisons/NSA-vs-SA-Security|NSA-vs-SA-Security]] - NSA 與 SA 組網的安全差異：手機顯示「5G」不代表用上 5G 核心網的 SUCI / Home Control（含推論標註）[TS 33.501 §1, §6.10.1]
- [[wiki/comparisons/NAS-vs-AS-Security|NAS-vs-AS-Security]] - NAS 安全與 AS 安全分層橫向對比（通訊端點、信任邊界隔離、更新頻率、啟用順序與空窗期分析）[TS 33.501 §6.4, §6.5]

---

## 📚 原始資料吸納紀錄 (Source Receipts)
- [[wiki/sources/SRC-TS33501-Week4|SRC-TS33501-Week4]] - 5G 行動網路安全架構 課程講義 (TS 33.501 導讀，全篇 34 頁)
- [[wiki/sources/SRC-W2-Network-Overview|SRC-W2-Network-Overview]] - 5G 行動通訊網路概論 投影片（輕量 ingest，全篇 97 頁；AMF、NSA/SA、SK Telecom 案例）
- [[wiki/sources/SRC-Week3-Week4-Identity-Keys|SRC-Week3-Week4-Identity-Keys]] - 5G 網路系統之安全認證與身分識別以及密鑰管理 投影片（輕量 ingest，全篇約 102 頁；含 9 項講義錯誤清單）
- [[wiki/sources/SRC-Week5-Air-Interface|SRC-Week5-Air-Interface]] - 5G 行動空中介面安全 課程講義 (Uu 介面與未受保護訊息導讀，全篇 35 頁；波次 2 已完成並查核)

---

## 🩺 出處查核狀態 (Traceability Audit Status)
> 詳細抽查紀錄見 `log.md`。✅ = 已由 Claude Code 抽查並修正。
- 階段一（SUPI、SUCI、ECIES、SIDF、UDM、ARPF、AUSF、SEAF）：✅ 2026-11-11 重查，15/23 通過，8 項已修正
- 波次 1（5G-AKA、Key-Hierarchy、EAP-AKA-prime、4G-vs-5G-AKA）：✅ 2026-11-11 重查，7/12 通過，5 項已修正
- 波次 2（Week5 空中介面）：✅ 2026-11-11 查核，0/3 通過，3 項已修正
- Week3/4 投影片（USIM、Cellular-Generations-Security、來源收據，及 ECIES / SUCI 補記）：✅ 2026-11-11 逐頁即時查核，57/61 首次通過，4 項已修正
- W2 投影片（AMF、NSA-vs-SA-Security、來源收據，及 USIM / Air-Interface 補記）：✅ 2026-11-11 逐頁即時查核，57/64 首次通過，7 項已修正
- 簡報前補查（overview、False-Base-Station-Defenses、User-Plane-Integrity、Security-Mode-Command 中簡報會用到的部分）＋新建 UE：✅ 2026-11-11，修正 8 處無出處或誇大說法
- 第二份簡報前補 wiki（False-Base-Station-Defenses 補 7 塊、新建 Research-Ethics、修正 5G-AKA 的 Basin 誤植與 FBS 的 SDR / AdaptOver 年份 / 「完全失效」）：✅ 2026-11-11 逐頁即時查核，30/30 新增說法通過
- ⚠️ 已知講義與規格分歧（Week3/4 另見其來源收據的 9 項錯誤清單）：SEAF 初篩失敗處理（講義 p.17 vs §6.1.3.2.2）、SUPI 明文限制用語（should vs shall）、LTM 條款 §6.17（V18.12.0 無此條）

---

## 🎞️ 教學簡報 (Slides)
- `slides/index.html` - 簡報目錄頁（連到兩份簡報）
- `slides/identity.html` - 第一份「5G 怎麼保護你的身分與連線安全」15 張（原 `slides/index.html`；簡報 ↔ wiki 查核 73/77 首次通過）
- `slides/fake-bts.html` - 第二份「假基地台：為什麼這個問題修了 30 年？」15 張（簡報 ↔ wiki 查核 80/84 首次通過）
- `slides/slide-wiki-map.md` - 兩份簡報的投影片 → wiki 頁面對照表與查核明細
