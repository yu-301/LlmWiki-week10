# 投影片 → wiki 頁面對照表

> 簡報目錄：`slides/index.html`。本表收錄兩份簡報：
> - [第一份](#第一份slidesidentityhtml)：`slides/identity.html`「5G 怎麼保護你的身分與連線安全」（15 張；原檔名 `slides/index.html`，2026-11-11 改名）
> - [第二份](#第二份slidesfake-btshtml)：`slides/fake-bts.html`「假基地台：為什麼這個問題修了 30 年？」（15 張）

---

## 第一份：slides/identity.html

> 簡報：`slides/identity.html`「5G 怎麼保護你的身分與連線安全」（15 張）  
> 規則：投影片內容只取自 `wiki/`。本表列出每張投影片的每個說法對應到哪個 wiki 頁面的哪一段。  
> 查核日期：2026-11-11（Claude Code）。查核結果與統計見 `log.md`。

| # | 投影片標題 | 段落 | 主要說法 | 對應 wiki 頁面（段落） | 查核 |
|---|---|---|---|---|---|
| 1 | 封面 | — | 四個主題：藏身分、互相驗證、鎖電波、還沒做到的事 | `overview`（認證三部曲、第四步、仍存在的空窗） | ✅ 2/2 |
| 2 | 你的手機，每天都在空中「喊話」 | 一、為什麼 | 有線機房鎖門 vs 電波四散；防竊聽、防竄改、防重送；身分隱私 | `Air-Interface-Security`（生活化比喻、一句話定義）、`SUCI`（安全防禦目標） | ✅ 3/3 |
| 3 | 假基地台：一個修了三十年的問題 | 一、為什麼 | IMSI Catcher 四步驟；明文 IMSI 與長期追蹤；雞生蛋困境；公鑰解法 | `4G-vs-5G-AKA`（4G 缺陷場景）、`SUPI`（為什麼需要它）、`SUCI`（破解三十年的困境）、`overview` | ✅ 6/6 |
| 4 | 登場角色：一次連線有哪幾方？ | 二、基本概念 | UE = ME + USIM；K 不出卡；基地台讀不到 NAS；AMF＋SEAF 同節點；AUSF 法官；UDM 含 SIDF、ARPF；當地網路≠你的電信商 | `UE`、`USIM`、`AMF`（比喻、§6.2.2.1）、`SEAF`、`AUSF`、`UDM`、`SIDF`、`ARPF`、`NAS-vs-AS-Security`（總表） | ✅ 7/7 |
| 5 | 從 2G 到 5G：每一代補一個洞 | 二、基本概念 | 四代門禁比喻表；5G 演算法與 4G 相同；5G 的進步在架構 | `Cellular-Generations-Security`（比喻表、演算法總表） | ✅ 3/3 |
| 6 | 身分證字號不能大聲喊：SUPI → SUCI | 三、機制 ① | SUPI 永久唯一；SUCI 一次性；公鑰在 USIM；466/97/123456789；MCC/MNC 明文、MSIN 加密；隱私與路由的取捨 | `SUPI`（比喻、格式）、`SUCI`（欄位結構、為什麼不連 MCC/MNC 一起加密）、`USIM`（IMSI 格式範例） | ✅ 6/6 |
| 7 | SUCI 是怎麼做出來的？ | 三、機制 ① | ECIES 四步；每次換臨時金鑰＝不可連結性；只有 SIDF 能解；私鑰外洩後果；認證仍安全 | `ECIES`（比喻、五步演算法）、`SIDF`（若 SIDF 私鑰外洩、隱私與認證解耦）、`SUCI` | ✅ 6/6 |
| 8 | 5G-AKA：手機與網路互相考試 | 三、機制 ② | 三階段；只給雜湊、留答案；USIM 驗 MAC 與 SQN；只回 RES*、不傳金鑰；K 只在 USIM 與 ARPF；假基地台 MAC failure 中斷 | `5G-AKA`（比喻、兩道檢查流程圖）、`UE`（認證處理）、`USIM`、`ARPF` | ✅ 6/6 |
| 9 | 誰說了算？Home Control | 三、機制 ② | 4G MME 自行結案、漫遊詐欺；SEAF 初篩得到判定依據；RES* 必送回 AUSF；只有 AUSF 說成功才算；成功前拿不到 SUPI 與錨點金鑰；效率 vs 權威 | `4G-vs-5G-AKA`、`SEAF`（初篩與 WARNING）、`AUSF`、`5G-AKA` | ✅ 6/6 |
| 10 | 金鑰階層：不要一把萬能鑰匙用到底 | 三、機制 ③ | 金鑰樹；金鑰分離、生命週期、信任邊界；情境綁定；NH 鏈與搬家換鎖 | `Key-Hierarchy`（為什麼不一把萬能金鑰、金鑰樹、情境鎖死、換手金鑰接力） | ✅ 5/5 |
| 11 | 先約好密碼本，再把每個封包鎖好 | 三、機制 ④ | 手機宣告能力（未保護）；SMC 原封重播＋完整性；先驗章再加密；保險箱／封條／流水號；NAS 與 AS 兩層 | `Security-Mode-Command`（比喻、三道防線）、`UE`（安全模式與演算法）、`Air-Interface-Security`（比喻）、`NAS-vs-AS-Security`、`AMF` | ✅ 6/6 |
| 12 | 有了鑰匙之前，什麼都保護不了 | 四、限制 ① | 不能保護的訊息；偽造廣播、覆蓋、偽造拒絕；撥打掛斷的 Paging 側通道；密碼學保護內容不保護「有沒有發生」；廣播簽章三障礙；治理成本高於密碼學 | `False-Base-Station-Defenses`（未保護訊息總表、四大攻擊、廣播簽章障礙、學術界結論） | ⚠️ 4/6 → 修正後 6/6 |
| 13 | 能力存在，不等於已經啟用 | 四、限制 ② | 4G 只加密、aLTEr 竄改 DNS；5G UP 完整性由政策決定；量測研究指出普遍沒開、效能成本；GSMA 5G 圖示；台灣 NSA 開台；NSA 由 4G 核心網控制；推論 | `User-Plane-Integrity`（比喻、aLTEr、部署落差）、`NSA-vs-SA-Security`（1a、比較表、推論 NOTE） | ⚠️ 5/6 → 修正後 6/6 |
| 14 | 最弱的一環，常常不在空中 | 四、限制 ③ | SIM Swap 與 MFA；SK Telecom 案例數字；私鑰外洩；密碼學前提是金鑰沒被偷；僅 5G、關 2G、TLS／VPN | `USIM`（威脅表、SK Telecom 案例）、`SIDF`、`UE`（使用者可以自己做的事）、`False-Base-Station-Defenses`（三道防線） | ✅ 6/6 |
| 15 | 四道防線，三個提醒 | 五、總結 | 四道防線；三個提醒；帶走一句話 | `overview` 與第 2–14 張各頁 | ⚠️ 2/3 → 修正後 3/3 |

**合計：首次查核 73 / 77 = 94.8%**；4 項不符已全部修正（3 項補 wiki、1 項改投影片）。

## 不符項目明細
| 投影片 | 不符說法 | 類型 | 處理 |
|---|---|---|---|
| 12 | 「密碼學保護的是內容，不是有沒有發生」 | wiki 沒有支撐 | 回 raw 確認出自 Week5 p.19 後，補進 `False-Base-Station-Defenses`（Paging 側通道段） |
| 12 | 「技術可行，但部署與治理成本遠高於密碼學本身」 | wiki 沒有支撐 | 回 raw 確認出自 Week5 p.27–28 後，補進 `False-Base-Station-Defenses`（學術界的結論） |
| 13 | 「量測研究指出商用網路普遍沒開」 | wiki 沒有支撐（wiki 原寫「多數電信商設為 Not needed 或 Preferred」，未提量測研究） | 回 raw 確認出自 Week5 p.21 後，改寫 `User-Plane-Integrity` 部署落差段 |
| 15 | 「NSA 可能沒用上 5G 核心網」 | 內容與 wiki 不符（wiki 寫 NSA **一定**由 4G 核心網控制，不確定的是「5G 核心網的安全機制」） | 改投影片措辭為「NSA 下由 4G 核心網控制，5G 核心網的安全機制可能沒用上」 |

## wiki 頁面使用次數
| wiki 頁面 | 被引用的投影片 |
|---|---|
| `overview` | 1, 3, 15 |
| `Air-Interface-Security` | 2, 11 |
| `SUPI` | 3, 6 |
| `SUCI` | 2, 3, 6, 7 |
| `ECIES` | 7 |
| `4G-vs-5G-AKA` | 3, 9 |
| `Cellular-Generations-Security` | 5 |
| `UE` | 4, 8, 11, 14 |
| `USIM` | 4, 6, 8, 14 |
| `AMF` | 4, 11 |
| `SEAF` | 4, 9 |
| `AUSF` | 4, 9 |
| `UDM` | 4 |
| `SIDF` | 4, 7, 14 |
| `ARPF` | 4, 8 |
| `5G-AKA` | 8, 9 |
| `Key-Hierarchy` | 10 |
| `Security-Mode-Command` | 11 |
| `NAS-vs-AS-Security` | 4, 11 |
| `False-Base-Station-Defenses` | 12, 14 |
| `User-Plane-Integrity` | 13 |
| `NSA-vs-SA-Security` | 13 |

未被簡報使用的 wiki 頁面：`EAP-AKA-prime`（對初學者過深）、各來源收據（`SRC-*`）。

---

## 第二份：slides/fake-bts.html

> 簡報：`slides/fake-bts.html`「假基地台：為什麼這個問題修了 30 年？」（15 張），延伸第一份第 3、12 張，避免重複第一份已講過的內容。  
> 查核日期：2026-11-11（Claude Code）。查核結果與統計見 `log.md`。

| # | 投影片標題 | 段落 | 主要說法 | 對應 wiki 頁面（段落） | 查核 |
|---|---|---|---|---|---|
| 1 | 封面 | — | 延伸第一份；四個主題 | `False-Base-Station-Defenses` | ✅ 2/2 |
| 2 | 上一份的答案，只解了一半 | 一、重新定義 | SUCI 保護身分；5G-AKA＋Home Control 讓假基地台偽造不了認證；5G 沒有在建立安全上下文前認證細胞；假基地台仍可做的四件事；覆蓋攻擊的落腳點 | `False-Base-Station-Defenses`（問題的本質）、`SUCI`、`5G-AKA` | ✅ 5/5 |
| 3 | 病根：2G 從沒想過要驗證基地台 | 一、重新定義 | 2G 認證四步；手機沒驗證網路；四代假基地台能力表 | `Cellular-Generations-Security`（2G 單向認證、補洞表）、`SUCI`、`False-Base-Station-Defenses` | ✅ 6/6 |
| 4 | 安全是「分階段」打開的 | 二、解剖空窗 | 階段 0–4 保護狀態；階段 2 的 RRC 空窗與可被利用的方式 | `False-Base-Station-Defenses`（安全生效時間軸）、`NAS-vs-AS-Security`（啟用順序與空窗期）、`Security-Mode-Command` | ✅ 6/6 |
| 5 | 「不能保護」vs「選擇不保護」 | 二、解剖空窗 | 9 類結構性未保護訊息；1 類政策性；兩者的差別 | `False-Base-Station-Defenses`（未保護訊息總表、分類結論）、`User-Plane-Integrity` | ⚠️ 4/5 → 修正後 5/5 |
| 6 | 為什麼低層訊息塞不下保護？ | 二、解剖空窗 | MIB、MAC CE、DCI 的限制；功能下移趨勢；LTM；課堂提問 | `False-Base-Station-Defenses`（總表、功能下移的趨勢）、`Air-Interface-Security`（MAC-I 32-bit） | ✅ 6/6 |
| 7 | 不必架假基地台：訊號覆蓋攻擊 | 三、攻擊工具箱 | 傳統假基地台；覆蓋的原理；傳統偵測看不到；5G SA 仍可被覆蓋的訊息 | `False-Base-Station-Defenses`（訊號覆蓋攻擊）、`4G-vs-5G-AKA`（傳統假基地台描述） | ⚠️ 3/5 → 修正後 5/5 |
| 8 | 降級攻擊四型，只有一型被完全防住 | 三、攻擊工具箱 | 跨世代、演算法、資料保護、偽造拒絕四型與現況 | `False-Base-Station-Defenses`（降級四型總表）、`Security-Mode-Command` | ⚠️ 4/5 → 修正後 5/5 |
| 9 | 身分還是會從側門漏出去 | 三、攻擊工具箱 | 呼叫時機由識別碼公開算出；Msg3 帶 5G-S-TMSI；識別碼不輪替成追蹤標籤；AKA 兩種失敗訊息可區分 | `False-Base-Station-Defenses`（Paging 側通道、時間軸）、`5G-AKA`（可連結性 WARNING） | ✅ 4/4 |
| 10 | 最底層的攻擊 | 三、攻擊工具箱 | 隨機存取、DCI、MAC CE、參考訊號四類；被動攻擊無法偵測 | `False-Base-Station-Defenses`（低層控制程序攻擊） | ✅ 5/5 |
| 11 | 幫廣播加數位簽章：五個障礙 | 四、為什麼修不好 | 五個障礙；只有第 4 個與密碼學有關；學術界結論；後量子更難 | `False-Base-Station-Defenses`（簽章障礙、學術界結論、研究方向） | ✅ 8/8 |
| 12 | 標準組織在研究什麼？TR 33.809 | 四、為什麼修不好 | TR 不是 TS、不具強制力；四種候選方案與狀態 | `False-Base-Station-Defenses`（TR 33.809 候選方案表） | ✅ 5/5 |
| 13 | 30 年修不好的三個原因 | 四、為什麼修不好 | 結構性；治理成本；相容與取捨（USIM 換卡、涵蓋率、效能成本）；部署比加密難 | `False-Base-Station-Defenses`、`SUCI`、`5G-AKA`、`User-Plane-Integrity` | ✅ 7/7 |
| 14 | 下一步：研究方向與界線 | 五、未來與總結 | AI/ML、PLS、低層輕量保護、6G；四條倫理界線；做得到≠可以做 | `False-Base-Station-Defenses`（研究方向）、`Research-Ethics` | ✅ 8/8 |
| 15 | 30 年的問題，5G 解了一半 | 五、未來與總結 | 已解的一半；未解的一半；卡在治理與部署；無線層不可信 | `False-Base-Station-Defenses`、`SUCI`、`Security-Mode-Command`、`UE` | ✅ 7/7 |

**合計：首次查核 80 / 84 = 95.2%**；4 項不符已全部修正（1 項補 wiki、3 項改投影片）。

### 第二份不符項目明細
| 投影片 | 不符說法 | 類型 | 處理 |
|---|---|---|---|
| 5 | 結構性清單寫「RRCSetupRequest／**Setup**／Reject」，wiki 總表只寫「RRCSetupRequest / Reject」 | wiki 沒有支撐 | 改投影片，與 wiki 一致 |
| 7 | 5G SA 仍可被覆蓋的訊息中列了「RRC 連線建立初期的訊息」，wiki 原本沒有 | wiki 沒有支撐 | 回 raw 確認出自 Week5 p.19，補進 `False-Base-Station-Defenses`（5G SA 受害面） |
| 7 | 傳統假基地台寫「手機依細胞選擇規則接入」，wiki 沒有這個說法；且頁尾未標出描述來源頁 | wiki 沒有支撐／頁面標註不完整 | 改為 `4G-vs-5G-AKA` 的寫法「手機就會主動接入」，頁尾補標 `4G-vs-5G-AKA` |
| 8 | 結語「分清楚哪些已解決，就是理解假基地台問題的核心」——wiki 沒有這個說法 | wiki 沒有支撐 | 改為 wiki 原句「唯一已被完整防住、其餘三類仍有結構性缺口」 |

### 第二份 wiki 頁面使用次數
| wiki 頁面 | 被引用的投影片 |
|---|---|
| `False-Base-Station-Defenses` | 1–15（全部） |
| `SUCI` | 2, 3, 13, 15 |
| `5G-AKA` | 2, 9, 13 |
| `Cellular-Generations-Security` | 3 |
| `NAS-vs-AS-Security` | 4 |
| `Security-Mode-Command` | 4, 8, 15 |
| `User-Plane-Integrity` | 5, 13 |
| `Air-Interface-Security` | 6 |
| `4G-vs-5G-AKA` | 7 |
| `Research-Ethics` | 14 |
| `UE` | 15 |

### 與第一份的重複控管
| 第一份已講 | 第二份的處理 |
|---|---|
| IMSI Catcher 四步驟、雞生蛋困境（第一份第 3 張） | 只在第 2、15 張以一句話帶過，改講「2G 為什麼沒驗證基地台」（第 3 張） |
| 未保護訊息清單、撥打掛斷側通道、簽章三障礙（第一份第 12 張） | 改講完整時間軸、結構性 vs 政策性、識別碼不輪替與 AKA 可連結性、簽章五障礙全解 |
| aLTEr、NSA（第一份第 13 張） | 不重講；第 5、8 張只提「政策性」與「資料保護降級」 |
| 手機設定建議（第一份第 14 張） | 不重講；改為研究方向與倫理界線（第 14 張） |
