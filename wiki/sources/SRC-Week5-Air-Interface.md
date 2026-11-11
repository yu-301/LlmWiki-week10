# 來源收據：5G 行動空中介面安全 課程講義 (Uu 介面與未受保護訊息導讀)

> **檔案位置**：`raw/lectures/5G空中介面安全-Week5.pdf`  
> **文件性質**：大學部選修專題講座導讀教材（依據 3GPP TS 33.501 / TS 38.323 / TS 38.331 / TR 33.809 及公開學術研究編寫，全篇 35 頁）  
> **收納日期**：2026-11-11  
> **處理階段**：波次 2（空中介面安全、SMC 與偽基地台防禦）

---

## 📌 文件核心貢獻與定位
本講義深入探討 5G 系統中唯一透過電磁波在開放空間傳播的環節——**Uu 空中介面**。講義跳脫「5G 已經完全安全」的單純迷思，直擊蜂巢式網路的結構性限制：**「不能保護（認證前的物理空窗）」與「選擇不保護（營運商政策與開銷取捨）」的本質區別**。詳細剖析了 PDCP 加密與完整性保護機制、NAS/AS SMC 雙層安全建立時序、訊號覆蓋（Overshadowing）、aLTEr 位元翻轉攻擊、Paging 側通道、換手金鑰鏈（$NH/NCC$），以及 TR 33.809 針對偽基地台的研擬對策。

---

## 🗺️ 涵蓋主題與對應 3GPP 規範條款
| 講義章節 | 核心主題 | 對應 3GPP 規範與標準條款 | 熔接目標 Wiki 頁面 |
|---|---|---|---|
| 第 1 章 | 空中介面本質與協定堆疊（Uu 位置、為什麼保護在 PDCP、NAS vs AS） | TS 38.300, TS 38.323, TS 33.501 §6.4, §6.5 | `[[Air-Interface-Security]]`, `[[NAS-vs-AS-Security]]` |
| 第 2 章 | PDCP 保護機制（128-NEA/NIA 輸入輸出、COUNT=HFN\|\|SN、演算法選擇） | TS 38.323 §5.7–§5.8, TS 33.501 Annex D | `[[Air-Interface-Security]]` |
| 第 3 章 | 安全何時生效（階段 0–4 時間軸、Msg1–Msg5 暴露、未保護訊息清單） | TS 38.331, TS 33.501 §6.7.2, §6.7.4 | `[[Security-Mode-Command]]`, `[[False-Base-Station-Defenses]]` |
| 第 4 章 | 未受保護訊息與攻擊面（偽基站、Overshadowing、Paging 側通道、aLTEr、低層控制注入） | TR 33.809, TS 33.501 §6.6, 學術論文 | `[[False-Base-Station-Defenses]]`, `[[User-Plane-Integrity]]` |
| 第 5 章 | 換手與狀態轉換（$K_{gNB}$ 更新、Xn vs N2 換手、$NH/NCC$ 鏈、RRC_INACTIVE、shortMAC-I、雙連結、LTM） | TS 33.501 §6.8.2.1, §6.9.2.3, §6.9.4, §6.10, §6.11, Annex A.9–A.11（LTM 見下方 NOTE） | `[[Key-Hierarchy]]`（換手金鑰接力專節）, `[[Air-Interface-Security]]` |
| 第 6 章 | 現行與研擬對策（TR 33.809 廣播簽章五大障礙、三道實用防線、PLS 與 AI 展望） | TR 33.809, TS 33.501 §5.10 | `[[False-Base-Station-Defenses]]` |
| 第 7 章 | 實驗環境與研究倫理（srsRAN, OAI, Open5GS, Wireshark, 通保法與刑法界線） | 實驗法規與倫理 | 參考背景 |

> [!WARNING]
> **講義與本庫規格版本的條款差異**：講義 p.26 稱 LTM（L1/L2 觸發移動性）的安全處理定義於「TS 33.501 §6.17」，但本庫收錄的 `3GPP TS 33.501 V18.12.0` Clause 6 只到 §6.16（Security handling in Cellular IoT），全文亦查無 LTM 相關條款。可能是講義引用了不同版本（例如 Rel-19），引用 LTM 時請以講義為出處、勿標註 §6.17。（2026-11-11 Claude Code 查核補記）

---

## 💡 本次 Ingest 萃取之關鍵洞見 (Key Takeaways)
1. **「不能保護」與「選擇不保護」的本質區別**：
   - 「不能保護」屬於協定結構性限制：接收端在階段 0–1 尚未完成認證、手頭無任何金鑰，因此廣播（SSB/MIB/SIB1）、隨機存取（Msg1/Msg2）與初次 RRC 建立無法進行對稱保護 [講義 p.5, 14–17]。
   - 「選擇不保護」屬於營運商政策取捨：例如使用者平面完整性保護（UP-IP）在規格中已齊備，但營運商因傳輸頻寬與 CPU 計算成本常將 UP security policy 設為 `not needed`，因而衍生出 aLTEr 等攻擊面 [講義 p.5, 20–21]。
2. **PDCP 處理順序的核心設計哲學**：
   - 下行方向為「先算完整性標籤 MAC-I，再進行加密」；上行方向則「先解密，再驗證 MAC-I」。此順序使得安全模式命令（SMC）得以在尚未啟用加密的情況下「單獨啟用完整性驗證」，確保演算法選擇等關鍵參數不被竄改 [講義 p.9]。
   - PDCP 標頭（含 PDCP SN）強制明文傳送，使接收端得以先獲取 SN，還原出 32-bit $COUNT = HFN \parallel SN$，進而生成正確的對稱金鑰流 [講義 p.9, 11–12]。
3. **金鑰流重用（Keystream Reuse）是串流密碼最致命的死穴**：
   - 128-NEA 採用串流加密（如 AES-CTR, SNOW 3G, ZUC），若同一把金鑰使用相同 $COUNT$、$BEARER$ 與 $DIRECTION$，兩組密文互斥或（$C_1 \oplus C_2$）將直接消去金鑰流，暴露出兩段明文的互斥或（$M_1 \oplus M_2$）。因此規格嚴格要求 $COUNT$ 即將繞回前必須觸發 intra-cell handover 更新 $K_{gNB}$ [講義 p.10, 12, 24]。
4. **AS SMC 原封重播機制擊潰 Bidding-down 降級攻擊**：
   - 雖然 UE 在初始階段以明文宣告的安全能力可被中間人竄改為弱演算法（如 NEA0/NIA0），但 gNB 在受 $K_{RRCint}$ 完整性保護的 `AS Security Mode Command` 中原封重播其收到的能力清單。UE 驗證完整性通過後比對發現能力遭篡改，即刻中斷連線，使演算法降級攻擊徹底失效 [講義 p.12, 20]。
5. **廣播加數位簽章面臨的五大工程治理難題**：
   - 雖然 TR 33.809 研擬為 MIB/SIB 加入數位簽章，但在工程實務上面臨五大不可忽視的障礙：公鑰預置的雞生蛋困境、缺乏即時聯網環境下的憑證撤銷（CRL/OCSP）、防重播所需的時間戳與未同步時鐘矛盾、廣播無線頻寬開銷、以及終端初始搜尋延遲 [講義 p.27–28]。

---

## 📚 參考來源對照
- `raw/lectures/5G空中介面安全-Week5.pdf` (p.1–35)
- `raw/specs/3GPP TS 33.501 V18.12.0 (Release 18).docx` (§6.4, §6.5, §6.6, §6.7.2, §6.7.3.6, §6.7.4, §6.8.2.1, §6.9.2.3, §6.9.4, §6.10, §6.11, Annex D)
- 3GPP TS 38.323 (NR; PDCP specification)
- 3GPP TS 38.331 (NR; RRC protocol specification)
- 3GPP TR 33.809 (Study on 5G security enhancements against False Base Stations)
