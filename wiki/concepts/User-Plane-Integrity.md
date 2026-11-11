# 使用者平面完整性保護 (User Plane Integrity Protection - UP-IP)

> **一句話直觀定義**：UP-IP 是 5G 突破性引入的空中介面使用者資料防竄改機制，依據 SMF 下發之安全策略，對手機與基地台間的資料無線承載（DRB）逐封包計算並校驗 32-bit MAC-I，徹底終結了 4G 時代串流加密的位元翻轉劫持漏洞 [3GPP TS 33.501 Rel-18 §6.6.1; TS 38.323 §5.8; 講義: 5G空中介面安全-Week5.pdf, p.20–21]。

---

## 💡 直觀原理解析 (Beginner-Friendly Explanation)

### 生活化比喻：上鎖卻沒有封條的快遞包裹
想像你寄送一個內附重要合約的快遞包裹：
- **4G 的做法（只有加密，沒有完整性）**：你把合約裝進上鎖的鐵盒（加密）。路上的小偷雖然打不開盒子、看不到合約內容（機密性良好），但他拿出一塊強力磁鐵隔著鐵盒在特定位置掃過，把合約中的「匯款帳號」磁性墨水改成了自己的帳號（位元翻轉）。當收件人打開鐵盒時，根本不知道內容已經被動過手腳！
- **5G 的做法（UP-IP 完整性保護）**：除了鎖上鐵盒，你還在箱子接縫處貼上高科技防偽雷射封條（32-bit MAC-I）。任何人只要隔空微調哪怕一個位元，收件人開箱驗算封條時立刻發現破損，直接將包裹丟進垃圾桶（靜默丟棄），絕不採納！

### 為什麼 4G 只有加密不行？（aLTEr 與 IMP4GT 攻擊啟示）
4G 空中介面普遍採用 **AES-CTR** 或 **SNOW 3G** 等串流密碼（Stream Cipher）。串流密碼具備**密文可延展性（Malleability）**：
$$\text{正常}：C = M \oplus KS \implies M = C \oplus KS$$
$$\text{攻擊}：C' = C \oplus \Delta \implies M' = C' \oplus KS = (M \oplus KS \oplus \Delta) \oplus KS = M \oplus \Delta$$
- **致命弱點**：攻擊者**不需要破解金鑰，也不需要讀懂明文**。只要攻擊者知道目標封包的既定結構（例如 IP 標頭格式或未加密的 DNS 查詢請求位置），攻擊者只要構造微調差值 $\Delta$，就能精確改變解密後的明文！
- **真實世界攻擊案例**：
  - **aLTEr 攻擊（IEEE S&P 2019, Rupprecht et al.）**：研究者在 4G 空中鏈路攔截加密的 DNS 查詢封包，利用位元翻轉將 DNS 伺服器 IP 改為攻擊者的惡意 DNS，將用戶神不知鬼不覺導向釣魚網站 [講義 p.20]。
  - **IMP4GT 攻擊（NDSS 2020, Rupprecht et al.）**：進一步示範雙向冒充，利用未受完整性保護的漏洞偽造上行與下行封包，以受害者身分任意收發資料 [講義 p.21]。

---

## ⚙️ 技術架構與運作機制 (Technical Mechanism)

### 1. SMF 集中控制之 UP Security Policy
依據 **3GPP TS 33.501 §6.6.1**，使用者平面的保護並非全網死板啟用，而是由核心網 [[SMF]] 在建立 PDU Session 時下發給 gNB，對加密（Confidentiality）與完整性（Integrity）各自提供獨立指示，取值三選一 [講義 p.20, 25]：

| Policy 取值 | 規範強制力與運作行為 | 安全與可用性取捨哲學 |
|---|---|---|
| **`Required`** | **必須強制啟用**。若 gNB 硬體處理能力不足或因故無法支援，**必須直接拒絕建立該 PDU Session**。 | **「寧可不連，也不要不安全」**。適用於對安全有極高要求的垂直產業專網、金融交易或車聯網。 |
| **`Preferred`** | **偏好啟用**。若 gNB 具備充足算力則開啟；若 gNB 負載過高或終端能力受限，仍允許退回無保護狀態建立連線。 | 優先追求連線可用性，在環境允許時提供最佳防護。 |
| **`Not needed`** | **明確不啟用**。PDCP 封包不計算亦不附加 MAC-I 欄位。 | 適用於本身已有上層強加密保護（如 HTTPS、VPN）或對延遲極度敏感的超高速流量。 |

### 2. PDCP 層 32-bit MAC-I 封裝
當 UP 完整性策略設為 `Required` 時：
1. **發送端運算**：PDCP 層以 $K_{UPint}$、32-bit $COUNT$、$BEARER$、$DIRECTION$ 與資料本體輸入 `128-NIA` 演算法，計算出 32 bits 的 MAC-I 並附於封包末端 [TS 38.323 §5.8]。
2. **下行先整後密**：MAC-I 計算完畢後，隨同資料載荷一起送入 `128-NEA` 引擎以 $K_{UPenc}$ 加密。
3. **接收端驗證**：接收端 PDCP 解密後以相同參數驗算 $XMAC\text{-}I$。比對若有任何 1 bit 不合，該資料封包被**立即靜默丟棄**。

---

## ⚖️ 規格理想 vs 商用部署落差 (Spec vs Deployment Gap)

> 💡 **講義核心警語**：  
> **「能力存在不等於已啟用」**。讀規格只完成了一半，另一半是實際量測 [講義 p.21]。

儘管 3GPP 規格在 5G 中定義了解決 aLTEr 的密碼機制，**多項量測研究指出，商用網路普遍未對一般流量啟用 UP 完整性保護** [講義: 5G空中介面安全-Week5.pdf, p.21]，其主要工程原因包括：
1. **嚴重的頻寬與 CPU 效能代價**：在 5G 高達數 Gbps 的超高吞吐率下，對「每一個」資料封包逐一計算 32-bit MAC-I，會使基地台與手機基頻晶片承受極其沉重的 CPU 負荷與發熱耗電 [講義 p.21, 25]。
2. **裝置相容性**：早期規格對支援 UP 完整性保護的資料速率有上限 [講義: 5G空中介面安全-Week5.pdf, p.21]；現行 Rel-18 則要求 UE 在任何資料速率（含其支援的最高速率）都必須支援使用者資料的完整性保護 [TS 33.501 Rel-18 §5.2.3]。

---

## 🛡️ 安全防禦目標 (Security Objectives)

1. **阻斷 Layer 2 位元翻轉攻擊**：徹底瓦解 aLTEr、IMP4GT 等利用串流密碼可延展性竄改 DNS 或特定協定標頭的手法。
2. **終結非授權封包注入**：防止偽造的使用者平面封包滲入核心網，保護用戶帳單免遭幽靈流量盜刷。
3. **關鍵基礎設施專網保證**：在智慧工廠（IIoT）、遠距醫療手術與自動駕駛車聯網中，提供實體電磁波層級的零篡改保證。

---

## 🔗 關聯概念與流程
- 空中安全本體：[[Air-Interface-Security]]
- 流程啟用控制：[[Security-Mode-Command]]
- 偽基站與空窗：[[False-Base-Station-Defenses]]
- 橫向架構對照：[[4G-vs-5G-AKA]]
- 金鑰推導層級：[[Key-Hierarchy]]

---

## 📚 參考文獻與條款溯源 (References & Traceability)
- 3GPP TS 33.501 Rel-18:
  - §6.6.1 (UP security policy)
  - §6.6.2 (UP security activation mechanism)
  - §6.6.4 (UP integrity mechanisms)
  - Annex D (Crypto algorithms: 128-NIA1/2/3)
- 3GPP TS 38.323: §5.8 (Integrity protection and verification)
- 學術研究文獻:
  - D. Rupprecht et al., "Breaking LTE on Layer Two," IEEE S&P 2019 (aLTEr Attack)
  - D. Rupprecht et al., "IMP4GT: IMPersonation Attacks in 4G NeTworks," NDSS 2020
- 課程講義來源: `raw/lectures/5G空中介面安全-Week5.pdf` (第 4 章 p.20–21) 與 `5G安全_TS33501-Week4.pdf` (第 1 章 p.7、第 6 章 p.25)
