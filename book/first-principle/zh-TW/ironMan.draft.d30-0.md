# Day 30 | 系列完結篇：給未來工程師的系統設計學習路線與反思 - 系列文章的學習資源、學習主題與心得

## 寫在最後:一場追問「為什麼」的旅程

三十天過去了。

如果你問我這趟旅程最大的收穫是什麼，我會說: **不是學會了多少 AWS 服務的配置技巧，而是重新建立了一套思考系統設計的哲學框架。**

在許久之前，我也曾經是那個迷失在技術清單中的工程師 - 看到新的框架就想嘗試，聽到別人說微服務就覺得單體落伍，遇到性能問題就直覺地想加快取。但隨著不斷的累積工作經驗與爆炸經驗後，讓我明白了一個根本性的轉變:

> **從「怎麼做」(How)到「為什麼」(Why)的思維躍遷，才是架構師與普通工程師最本質的區別。**

這個系列不是一本 AWS 服務使用手冊，而是一場關於系統設計本質的哲學探索。每一個技術決策背後，都藏著更深層的提問:這個系統存在的目的是什麼?它要解決誰的什麼問題?在資源有限的現實中，我們如何做出最務實的權衡?

## 系列內容回顧:從哲學到實踐的完整拼圖

讓我用一個隱喻來串聯這三十天的旅程:**我們一起建造了一座城市**。

### 第一章:奠基哲學 (Day 1-5) - 城市的規劃藍圖

在動工之前，我們先站在空地上，思考這座城市為誰而建、要解決什麼問題。

- **Day 1** 提出了核心命題:系統設計不是技術堆砌，而是目的導向的創造。領域驅動設計(DDD)不是流行術語，而是幫助我們回答「這個系統為何存在」的思維工具。
- **Day 2** 教我們如何將模糊的商業需求轉化為清晰的系統邊界，就像將城市規劃中「我們需要交通」這樣抽象的需求，細化為「需要幾條主幹道、地鐵要覆蓋哪些區域」的具體設計。
- **Day 3-4** 深入 DDD 的核心:如何用抽象建模識別領域邊界，如何用聚合設計保護業務不變式。這是在為城市劃分行政區，確保每個區域有明確的職責與邊界。
- **Day 5** 將視角轉向使用者:User Story 與 Scenario Flow 讓我們從「市民」的角度理解城市，確保設計不是閉門造車。

**核心反思**:**技術服務於目的，而非目的服務於技術**。當我們被 AI 輔助工具包圍時，能夠清晰定義問題、識別領域邊界的能力，反而成為最稀缺的競爭力。

### 第二章:技術決策 (Day 6-9) - 城市的基礎建設

有了藍圖，我們開始面對現實的約束:預算有限、時間緊迫、需求會變。

- **Day 6** 探討 Trade-off 的藝術:在成本、性能、可維護性之間找平衡，就像城市規劃中要在地價、交通便利性、環境品質間做抉擇。Lambda、ECS、EC2 的選擇，從來不是「哪個更好」，而是「哪個最符合當下目的」。
- **Day 7** 將領域模型映射為技術架構:單體、微服務、Serverless 不是時尚符號，而是對應不同業務複雜度與團隊成熟度的務實選擇。
- **Day 8** 將前端設計系統與原子化架構引入，說明了從最小的 UI 組件到完整的業務流程，如何保持設計的一致性與可擴展性。
- **Day 9** 面對高併發場景，學習如何用限流、降級、熔斷等策略保護系統，如同城市的交通管制與應急預案。

**核心反思**:**沒有絕對的最佳解，只有當下情境中的最適解**。每一個技術選擇都伴隨著代價，優秀的架構師不是找到完美方案的人，而是能清晰權衡並承擔決策後果的人。

### 第三章:數據與狀態 (Day 10-11) - 城市的記憶系統

城市需要記憶——交易記錄、用戶行為、系統狀態。如何存儲、如何快速檢索、如何保證一致性?

- **Day 10** 深入快取策略的哲學:時間與空間的權衡、一致性與性能的取捨。快取不只是「讓系統變快」，更是對數據生命週期的深刻理解。
- **Day 11** 探討資料庫設計:從領域模型到 Schema 設計，從 ACID 到最終一致性，每一個選擇都反映了我們對業務本質的理解程度。

**核心反思**:**數據是系統的靈魂，而如何對待數據，體現了我們對業務的尊重程度**。快取雪崩、數據不一致這些技術問題，本質上是我們對業務邏輯理解不夠深刻的體現。

### 第四章:工程實踐 (Day 12-15) - 城市的建設流程

再好的設計，如果無法安全、高效地落地，也只是空中樓閣。

- **Day 12** 建立版本控制與代碼審查文化，確保團隊協作的品質。
- **Day 13** 用 OpenAPI 等工具建立團隊間的契約，讓前後端、不同服務間能夠並行開發而不互相阻塞。
- **Day 14** 將基礎設施代碼化(IaC)，讓系統的部署可重複、可追溯、可回滾。
- **Day 15** 建立 CI/CD 流水線，讓從代碼提交到生產部署的每一步都自動化、可觀測。

**核心反思**:**工程實踐是理想與現實的橋樑**。再優雅的架構，如果團隊無法安全地交付，也無法產生商業價值。這個階段讓我理解:架構師不只設計系統，更要設計讓團隊能夠高效協作的流程與工具鏈。

### 第五章:環境與體驗 (Day 16-18) - 城市的使用體驗

系統不是孤立存在的，它運行在多個環境中，被不同角色的人使用。

- **Day 16** 探討多環境治理:開發、測試、生產環境的隔離與一致性，如同城市的開發區、試驗區與成熟區的規劃。
- **Day 17** 關注開發者體驗(DX):好的內部工具與除錯設計，能讓團隊事半功倍。
- **Day 18** 制定系統驗收準則:如何定義「完成」，如何確保交付品質符合預期。

**核心反思**:**系統的成功不只看最終用戶的體驗，也看團隊的開發體驗**。一個難以除錯、難以部署的系統，再優雅的架構也會在長期維護中逐漸腐化。

### 第六章:品質保證 (Day 19-21) - 城市的安全檢驗

如何確保我們建造的城市是安全的、可靠的、能承受壓力的?

- **Day 19** 從使用者視角進行 UX 測試與可用性驗證，確保系統真正解決了用戶的問題。
- **Day 20** 建立從單元測試到集成測試的完整測試策略，讓系統的每一層都有品質保障。
- **Day 21** 進行性能與壓力測試，識別系統在極限負載下的瓶頸與風險點。

**核心反思**:**測試不是開發完成後的附加步驟，而是設計階段就應該考慮的核心能力**。可測試的系統往往也是設計良好、邊界清晰的系統。

### 第七章:安全與韌性 (Day 22-25) - 城市的防禦體系

面對未知的威脅與故障，系統如何保持韌性?

- **Day 22** 引入零信任架構:從邊界防禦到身份驗證的思維轉變，重新定義安全邊界。
- **Day 23** 建立可觀測性三大支柱(Logs， Metrics， Traces)，讓系統能「對自己說話」，及時發現問題。
- **Day 24** 用 SRE 方法論定義與衡量可靠性:SLI、SLO、錯誤預算等概念，讓可靠性從口號變為可量化的指標。
- **Day 25** 透過混沌工程主動驗證韌性，在受控環境中提前暴露系統的脆弱點。

**核心反思**:**韌性不是被動的容錯，而是主動的設計選擇**。優秀的系統不是從不出錯，而是能夠優雅地處理錯誤、快速恢復並從失敗中學習。

### 第八章:演進與持續改進 (Day 26-29) - 城市的有機成長

系統不是一次性交付的產品，而是持續演進的生命體。

- **Day 26** 建立數據驅動的產品決策機制:從 A/B 測試到北極星指標，讓每一次改進都有數據支撐。
- **Day 27** 識別與償還技術債:用技術債象限與童子軍規則，讓系統在演進中保持健康。
- **Day 28** 關注數據治理與隱私保護:在 GDPR 等法規要求下，如何設計合規的數據生命週期。
- **Day 29-1** 探討架構演進的策略:用絞殺者模式與 BFF，實現從單體到微服務的平滑過渡，避免大爆炸式重寫的災難。
- **Day 29-2** 引入 AI 協作框架:在 AI 時代，架構師如何與 AI 工具協同，放大自己的思考與決策能力。

**核心反思**:**演進比完美更重要**。沒有一次到位的完美架構，只有持續適應業務變化、技術演進的有機系統。架構師的職責不是建造紀念碑，而是培育一個能自我更新的生態系統。

## 參賽心得:從輸出到反思的成長迴路

### 一、為什麼參加鐵人賽?起心動念

老實說，一開始報名鐵人賽有點衝動。看到主題「Build on AWS」時，我心想:「好啊，我每天都在做這件事，應該有很多可以分享的。」

但當我坐在電腦前準備寫第一篇文章時，才發現 **「做得到」和「說得清楚」是兩回事** 。

就像我一直在文章中強調的 **抽象化的概念** ，這些經驗與心得何嘗不是一種抽象化的概念，然後我就開始頭痛要怎麼寫了 TAT

這三十天的寫作，與其說是在教別人，不如說是 **逼著自己把碎片化的經驗系統化，把隱性的知識顯性化** 。

### 二、最大的挑戰:在深度與廣度間找平衡

寫技術文章最難的不是「寫什麼」，而是「不寫什麼」。

系統設計涉及的面向真的真的真的太廣了 - 架構模式、雲端服務、數據設計、安全合規、團隊協作……每一個主題都可以寫成一本書。三十天的篇幅，要如何取捨?

我最終選擇的策略是:**以「目的導向」為主軸，串聯所有技術細節**。

不是平鋪直敘地介紹 AWS 的每個服務，而是從真實的業務場景出發:投資交易系統需要極低延遲與強一致性，所以我們選擇這樣的架構組合;家庭財務系統成本敏感，所以我們用不同的技術棧;健康監控系統要處理 IoT 數據流，所以我們設計這樣的處理管線。

**讓「為什麼」引導「是什麼」和「怎麼做」** ，這個寫作框架也是我思考系統設計的心智模型。

### 三、意外的收穫:重新發現哲學的價值

在寫 Day 1 時，我幾乎是下意識地引用了亞里士多德的「目的因」概念。當時還有點忐忑:讀者會不會覺得太學術、太抽象?

好端端的工程師怎麼又是談哲學的、又是談商業的，甚至還談交友軟體 XD

但隨著系列展開，我越來越確信:**哲學思維是架構師最強大的武器**。

技術會過時——今天的 Kubernetes 可能是明天的 Docker Swarm;服務會變化——AWS 每年推出上百個新功能。但那些根本性的思考方式是永恆的:如何在約束中找到最適解?如何定義系統的邊界與責任?如何在不確定性中做出決策?

**當你掌握了這些底層的思維工具，面對任何新技術、新場景，都能快速找到思考的抓手**。

### 四、寫作即思考:輸出倒逼輸入的良性迴路

有幾篇文章我重寫了三次以上。

比如 Day 29 講架構演進，第一版寫得很技術化 - 絞殺者模式的步驟、BFF 的代碼範例……但讀起來很乾，缺少靈魂。

後來我停下來問自己:**這篇文章要解決的核心問題是什麼?**

答案逐漸清晰:**不是教大家如何實施絞殺者模式，而是為什麼要選擇「演進」而非「重寫」這條路**。

於是我引入了「大爆炸式重寫」的四大災難、園丁哲學、飛輪效應……用隱喻與故事，把技術決策背後的權衡與人性因素展現出來。

**寫作逼我把「知道」變成「理解」，把「理解」變成「能教會別人」**。這個過程痛苦但收穫巨大。

### 五、如果重來一次，我會怎麼做?

坦白說，如果現在讓我重新規劃這三十天，我會調整一些內容:

1. **更早引入實戰案例**:前五天有點太理論化，如果能更早引入具體的系統案例(比如在 Day 2 就引入投資交易系統)，可能會讓讀者更容易進入情境。

2. **增加「反模式」的討論**:我花了很多篇幅講「應該怎麼做」，但其實「常見的錯誤做法」往往更有教學價值。如果能增加一些「踩坑案例」，會更接地氣。

3. **加強視覺化呈現**:系統架構很適合用圖解說明，但礙於平台限制，很多架構圖沒能充分展示。未來如果有機會，我會嘗試用影片或互動式圖表來輔助說明。

但整體而言，**我很滿意這個系列達成的目標:建立一套從哲學到實踐、從需求到運營的完整思維框架**。

## 給未來學習者的建議

如果你也想深入系統設計領域，基於這三十天的經驗，我有幾點建議:

### 建議一:從「為什麼」開始，而非「怎麼做」

不要急著學工具。在學習 Kubernetes 之前，先問自己:為什麼需要容器編排?在學習微服務之前，先問:我的業務複雜度真的需要拆分嗎?

**技術是為了解決問題而存在的，不理解問題就學工具，就像不知道要去哪裡就買了一輛車**。

### 建議二:建立自己的學習體系，而非追逐熱點

技術圈永遠有新的熱點——今天是 Serverless，明天是邊緣運算，後天是 Web3。如果只是跟風學習，永遠學不完，也學不深。

更好的做法是:**找到一套底層的思維框架(比如領域驅動設計、系統思考、Trade-off 分析)，然後用新技術來驗證與豐富這個框架**。

就像我在這個系列中，用 AWS 服務來具象化領域驅動的概念，但核心的思維框架是可以遷移到任何雲端平台、甚至任何技術領域的。

### 建議三:多寫、多畫、多講

**理解有三個層次**:

1. **知道**:看過某個概念，大致有印象
2. **理解**:能在實際工作中應用
3. **內化**:能用自己的語言解釋給別人聽

大多數人停留在第一層，少數人到第二層，極少數人達到第三層。

而**到達第三層的最有效方法，就是輸出**——寫技術文章、畫架構圖、在團隊內部分享。輸出會暴露你理解的盲點，倒逼你去填補知識漏洞。

### 建議四:重視「軟技能」

系統設計不只是技術問題，更是溝通問題、協作問題、政治問題。

如何說服團隊接受新的架構?如何在不同部門的利益衝突中找到平衡?如何在預算有限的情況下，讓決策者理解技術投資的價值?

這些「軟技能」往往比純技術能力更難培養，但也更稀缺、更有價值。

Day 29 中提到的「政治阻力最小的灘頭堡策略」，就是軟技能與技術判斷的結合。

### 建議五:擁抱不完美，持續迭代

完美主義是學習的大敵。

不要等「準備好了」才開始寫文章、做分享;不要等「完全理解了」才應用新技術。

**在實踐中學習，在錯誤中成長，在反饋中迭代**——這才是真實的成長路徑。

這個系列絕對是不完美的，我自己非常清楚，有些地方寫得不夠深入，有些概念可能解釋得不夠清楚。但正是這種不完美，留下了未來改進的空間，也讓學習成為一個持續的過程，而非一次性的任務。

## 尾聲:架構師的使命

三十天過去，如果要我用一句話總結「架構師的使命」，我會說:

> **架構師的使命，是在混沌中建立秩序，在約束中創造可能，在變化中保持韌性。**

我們不是在建造不朽的紀念碑，而是在培育能自我演進的生命系統。我們的作品會過時、會被替換，但我們傳遞的思維方式、建立的工程文化，會在團隊中延續，成為組織的 DNA。

AI 會越來越強大，它會寫代碼、會設計架構、甚至會自動化運維。但有一件事 AI 做不到: **定義「這個系統應該存在的意義」**。

這正是人類架構師不可替代的價值——我們不只回答「How」，更回答「Why」和「What for」。

感謝這三十天的陪伴。無論你是從第一天就跟著的讀者，還是偶然路過的訪客，希望這個系列能為你的技術旅程帶來一些啟發。

**系統設計的學習沒有終點，但我們可以一起，在這條路上走得更遠、看得更清**。

---

> 「系統的設計，是概念化的仿生學。我們不只是在寫代碼，更是在創造有目的、有生命力的數位生物。」
>
> — Day 1 的開場，也是第 30 天的註腳

---

以下是我在 2024 年 Q3 AI 大爆發的時候設計的學習 a.k.a 複習路線，也是那時候 GPT 聰明的時候，我也是不斷的透過使用 AI 工具來調整學習我的技能樹。

我曾經在很多地方或得到了各位前輩的支持，要謝的人太多了，所以我後續會慢慢補上致謝銘，在詢問過當事人意願後，當然也要謝天啦 哈哈

我很喜歡做軟體開發與系統設計的最主要的原因之一 - 或許沒有之一，就是因為我們的環境相對來說是非常開放與友善的，在大多數的時候，許多前輩都非常樂意回答問題 - 只要不要白目沒做功課就去瞎 XX 亂問。

我很喜歡這種善的循環的概念，正是因為淋過雨，所以願意為其他人撐傘。

```md
# 全方位系統型工程師訓練計畫

從模組化元件設計到企業級系統架構，打造具備策略決策力與實作能力的現代全端 / 架構師人才

---

## 📚 整體課程摘要

這是一套針對「中高階工程師 → 架構師」成長軌跡所設計的實戰導向訓練計畫，涵蓋前端架構、API 設計、資料庫建模、CI/CD、系統可觀測性、效能優化、狀態管理、雲端部署、系統整體設計、架構風格哲學、跨部門協作等 **16 大職能模組**。

課程以企業級系統開發實務為基礎，透過大量演練、模擬提案、Trade-off 分析與設計圖繪製，幫助學員從單一技術角色轉型為具備 **設計能力 × 實作能力 × 決策能力** 的系統型工程師。

---

## 🎯 學習完成後的轉化能力與收穫

| 能力領域                 | 完成後能力轉化                                                                                   |
| ------------------------ | ------------------------------------------------------------------------------------------------ |
| 架構設計思維             | 能獨立提案中大型系統架構，針對模組劃分、資料一致性、可用性、可觀測性與部署策略進行設計與說明。   |
| 模組化與元件治理         | 能設計並維運企業級 Design System 與模組化元件庫，掌握元件抽象、可測試性與跨框架使用策略。        |
| API 與契約治理           | 熟練 RESTful/BFF/Gateway 架構設計，具備 OpenAPI 契約思維與版本演進治理能力。                     |
| 資料庫建模與效能優化     | 能應對 OLTP/OLAP 系統資料結構、特徵資料設計與高併發資料一致性挑戰。                              |
| Trade-off 分析與決策紀錄 | 能針對系統選型提出設計理由，書寫 ADR / Design Rationale，對 SLA/SLO/SLC 與容量模型具備掌握力。   |
| CI/CD 與 DevOps          | 能獨立設計 Git Flow + 多環境 CI/CD 自動化部署流程，熟悉 Docker、GitHub Actions、GitLab CI。      |
| 可觀測性與錯誤排查       | 建立從 metrics/logs/traces 到 alert 策略的觀測性架構，具備 MTTR 下降與 incident drill 實作能力。 |
| 效能優化與 SSR 渲染策略  | 精通 LCP/CLS/TTI 等 Web Vitals 指標優化，能導入 SSR / SSG / CDN / Skeleton 渲染模型。            |
| 狀態管理與同步策略       | 掌握 Redux / NgRx 狀態架構設計與 async sync 策略，支援跨角色 UI 狀態分離與前後端一致性。         |
| 架構風格判斷力           | 熟悉 Clean Architecture、DDD、Hexagonal、CQRS 等設計風格與其適配情境與限制。                     |
| 跨部門產品設計與流程推演 | 能與 PM / UX / Backend 有效協作，具備 MVP 拆解、流程圖繪製與需求轉語意化設計能力。               |
| 面試應對與技術簡報能力   | 每模組皆產出設計圖與成果報告，強化職涯面試應對、技術簡報、系統提案與設計說服力。                 |

---

## 💼 未來職涯對應路徑

| 角色                                           | 能力符合情境                                              |
| ---------------------------------------------- | --------------------------------------------------------- |
| 🎯 資深全端工程師（Senior Fullstack Engineer） | 擁有完整技術鏈與跨端整合能力，能主導架構設計與技術選型    |
| 🔧 系統架構師（System Architect）              | 能進行整體系統設計、技術決策、異常處理與部署策略提案      |
| 📊 技術產品設計師（Tech-Oriented PM）          | 熟悉技術與產品協作語言，具備規格書、流程圖與 MVP 推演能力 |
| 🧩 DevOps 工程師 / Platform Engineer           | 掌握 CI/CD、自動化部署與觀測性整合，能落實持續交付文化    |
| 📣 技術顧問 / 專案主持人                       | 可作為顧問型角色參與企業技術轉型與核心系統建構指導        |

[API \*\*\*\*解決方案設計](https://www.notion.so/API-214d54ae00a680c1b989c8841ea9d51f?pvs=21)

[系統設計簡要-系統整體設計與 trade-off ](https://www.notion.so/trade-off-214d54ae00a680239bc0cfd20ff75a12?pvs=21)

[企業級元件模組化與設計系統實務](https://www.notion.so/215d54ae00a6802596c9fb7ca7ed1408?pvs=21)

[架構風格概念理論](https://www.notion.so/215d54ae00a68057b936fe0d72e8d63d?pvs=21)

[版本控管藝術-Git Flow 與 CI/CD 自動化流程建構](https://www.notion.so/Git-Flow-CI-CD-216d54ae00a680b9acbbda4031b16f95?pvs=21)

[雲端高可用架構設計與實戰](https://www.notion.so/216d54ae00a680b48bedf7205bd4d76b?pvs=21)

[系統可觀測性與錯誤追蹤設計](https://www.notion.so/217d54ae00a68024bea5e267149212b8?pvs=21)

[資料庫 Schema 與 Index 設計](https://www.notion.so/Schema-Index-217d54ae00a68013913df0c2e008f175?pvs=21)

[複雜狀態管理策略](https://www.notion.so/218d54ae00a680b1badfdb2a2dbb3d57?pvs=21)

[UX Flow 與痛點辨識](https://www.notion.so/UX-Flow-218d54ae00a680ffae00d70d0319e8f0?pvs=21)

[A/B 測試與成長分析](https://www.notion.so/A-B-218d54ae00a6800dbef8d97cb45ed5bf?pvs=21)

[跨部門產品需求拆解與流程建構](https://www.notion.so/218d54ae00a680549d18ca6e5bb29994?pvs=21)

[@範本資源](https://www.notion.so/21bd54ae00a680d98654de1a13359b9b?pvs=21)
```

```md
# API 解決方案設計

## 🔁 課程定位

從基礎語意設計、業務建模、契約協議、版本策略、整合設計到部署與治理，建構完整 API 解決方案能力。

目標為能獨立應對企業級 RESTful 架構設計挑戰，並統合技術與策略思維。

## 📆 學期長度

18 週（對應 3 學分，總投入時間 60–80 小時）

## 🎯 結業目標

具備「設計、整合、佈署、治理」整體 API 解決方案能力，並能輸出一份企業級設計提案與系統文件（含契約、錯誤處理、版本控制、可擴展性設計等）

## 📚 教材核心

- 《REST API Design Rulebook》
- 《Designing Web APIs》
- 《API Design Patterns》
- 《The Art of Scalability》
- 《Domain-Driven Design: Tackling Complexity in the Heart of Software》

---

## 🧭 課程全覽

| 週次 | 主題名稱                          | 任務重點 / 進度檢核                        |
| ---- | --------------------------------- | ------------------------------------------ |
| W1   | 課程總覽與 API 的本質             | 架構圖繪製、自我定向學習目標表             |
| W2   | API 設計哲學：從 CRUD 到契約模型  | 撰寫三種 API 思維方式的比較分析文          |
| W3   | 業務語意建模與 API 的語言邏輯     | 對業務流程建模並對應出 API 原型            |
| W4   | URL 命名與資源設計策略            | 完成語意正確的 REST 資源路由設計           |
| W5   | HTTP 語義與錯誤設計哲學           | 錯誤狀態規劃、idempotency 與語意策略       |
| W6   | OpenAPI 作為契約與開發協議        | 撰寫 OpenAPI 契約並建立 Mock server        |
| W7   | Schema 設計與 Contract Testing    | 完成 Schema 驗證 + contract test 任務      |
| W8   | 🔍 小考週｜語意理解與契約整合     | YAML / OpenAPI 檔案實作 + 簡答題           |
| W9   | 系統整合場景：Webhook 與事件導向  | 設計 webhook 流程圖並模擬觸發流程          |
| W10  | gRPC / GraphQL / REST 協議比較    | 撰寫三種協議對應場景分析建議書             |
| W11  | API Versioning 策略與可演進設計   | 設計 /v1/ → /v2/ 共存架構與升級流程        |
| W12  | API 架構模式：BFF / Gateway 整合  | 設計 Gateway 架構圖（含 auth、cache）      |
| W13  | API Governance 策略與監控         | 撰寫 API Policy 文件與監控設計圖           |
| W14  | 專案架構週：從需求建模到 API 定義 | 輸出 Domain Model + API 契約與狀態轉移圖   |
| W15  | 專案實作週：功能驗證與部署測試    | 上傳測試錄影、Postman Collection、驗證報告 |
| W16  | 🔍 期末提案演練｜架構簡報 + demo  | 進行架構提案簡報與 API demo 展示           |
| W17  | 部署、壓測與可擴展性思考          | 撰寫壓力測試報告 + 水平擴展提案            |
| W18  | 🎓 發表與互評週                   | 最終成果發表 + 同儕互評 + 課程總結回饋     |

[W1 | 課程總覽與 API 的本質](https://www.notion.so/W1-API-214d54ae00a68052a2f0daac34fcfb85?pvs=21)

[W2 ｜ API 設計哲學：從 CRUD 到契約模型](https://www.notion.so/W2-API-CRUD-21bd54ae00a680139c3bc0fdacc20ae2?pvs=21)

[W3 | 業務語意建模與 API 的語言邏輯](https://www.notion.so/W3-API-222d54ae00a6809c9cadea88ef45ce49?pvs=21)

[W4 | URL 命名與資源設計策略](https://www.notion.so/W4-URL-229d54ae00a6803698adc26eaebe8928?pvs=21)
```

```md
# 系統設計簡要-系統整體設計與 trade-off

🔁 **課程定位**：

以架構師視角設計整體系統，涵蓋系統模組化、資料一致性處理、可用性與擴展性策略、可觀測性導入與設計決策分析，針對 trade-off 概念進行策略性建構。

📆 **學期長度**：

18 週，建議每週 6–8 小時，總投入約 150 小時（5 學分課程量級）

🎯 **結業目標**：

能獨立設計中大型系統的模組邊界、資料一致性策略、觀測模組、架構圖與 decision record，並能進行 trade-off 分析與容量預估，具備完整架構提案能力。

📚 **教材核心**：

《Fundamentals of Software Architecture》、

《Software Architecture: The Hard Parts》、

《Just Enough Software Architecture》、

《Domain-Driven Design》、

《Implementing Domain-Driven Design》、

《Clean Architecture》、

《Designing Data-Intensive Applications》、

《Kafka: The Definitive Guide》、

《Site Reliability Engineering (SRE)》、

《Observability Engineering》、

《The Art of Scalability》、

《Cloud Design Patterns》、

《Release It!》、

《The Art of Capacity Planning》

🧭 **課程全覽**：

| **週次** | **主題名稱**                | **任務重點 / 進度檢核**                              |
| -------- | --------------------------- | ---------------------------------------------------- |
| W1       | 架構設計導論                | 系統設計 vs 軟體設計區別、系統分層概念啟發           |
| W2       | 架構類型與視覺化            | Monolith / Microservices / Serverless 對比與繪圖實作 |
| W3       | 模組邊界與責任劃分          | Layer / Domain / Service 區分與場景演練              |
| W4       | 架構文件撰寫與提案練習      | ADR / decision log 產出與資料圖整合                  |
| W5       | 強一致性處理                | Transaction 模型設計與實作練習                       |
| W6       | 最終一致性與事件整合        | Event sourcing / Message queue 實務應用              |
| W7       | 非同步流程設計與 Queue 整合 | Kafka / RabbitMQ 等設計模式比較與整合任務            |
| W8       | 可用性設計與風險模式        | Retry / Timeout / Circuit breaker 設計               |
| W9       | 水平擴充策略與快取          | Cache / CDN / Sharding 模式演練                      |
| W10      | 可觀測性導論與模組劃設      | Metrics / Logging / Tracing 系統統合架構設計         |
| W11      | Observability Tools 實作    | Prometheus + Grafana / OpenTelemetry 演練            |
| W12      | Trade-off 框架建構          | 架構選擇對比與報告撰寫技巧                           |
| W13      | Trade-off 實戰練習與回顧    | 面試風格問題演練與多場景架構分析                     |
| W14      | SLA、SLO 與系統健康管理     | 運營視角加入架構評估，導出 SLI 計算與報告            |
| W15      | 系統容量預估模型            | 資源用量建模與性能壓力模擬分析                       |
| W16      | 架構提案整合報告撰寫        | 撰寫一份完整架構設計與決策提案報告                   |
| W17      | Final Defense 演練與評估    | 模擬面試 / 系統問題分析答辯訓練                      |
| W18      | 專案結案發表                | 完整系統設計文件與決策記錄報告 + 同儕評比            |

[**W1 ｜架構設計導論**](https://www.notion.so/W1-214d54ae00a680349a23eaf3ace80fe2?pvs=21)

[W2 ｜系統整體設計與 Trade-off × 架構類型與視覺化](https://www.notion.so/W2-Trade-off-21bd54ae00a6805a9735ece3f0456646?pvs=21)

[W3 | 模組邊界與責任劃分](https://www.notion.so/W3-223d54ae00a6809b9e19e27cee3f1dfb?pvs=21)

[W4 | 架構文件撰寫與提案練習](https://www.notion.so/W4-229d54ae00a6804a8bbdc2cd4be1fb4f?pvs=21)
```

```md
# 企業級元件模組化與設計系統實務

🔁 **課程定位**：

全面強化 Component 設計的底層理解與語意掌握，從 HTML 元素語境 → 渲染機制 → 元件治理 → 模組化維運，全鏈路掌握企業級 UI 設計系統實作技術。

📆 **學期長度**：

**20 週**（每週約 6 小時投入，總計約 120 小時，對應 4 學分）

🎯 **結業目標**：

能獨立設計、測試、治理並部署一套符合企業需求的元件系統，並能針對效能瓶頸、語意誤用與模組策略提出最佳化建議與重構方案。

📚 **教材核心**：

- 《Atomic Design》
- 《Designing Design Systems》
- 《Frontend Architecture for Design Systems》
- 《Refactoring UI》
- 《Clean Code》
- React / Angular / Web Component 官方文檔
- 《深入理解 React 技術內幕》
- **MDN HTML API**
- 《Inclusive Components》by Heydon Pickering
- 《Accessibility for Everyone》

---

🧭 課程全覽（20 週完整進度）

| **週次** | **主題名稱**                           | **任務重點 / 檢核**                                 |
| -------- | -------------------------------------- | --------------------------------------------------- |
| W1       | 為什麼需要設計系統與模組化？           | 繪製 Design System 架構圖與元件分層視覺化           |
| W2       | Atomic Design 與元件分層原則           | 重構任意 UI Flow 為 Atom → Template 元件樹          |
| W3       | HTML 元素語意與組合原理（結構性）      | 用 <section>、<article> 組合產品卡片元件            |
| W4       | HTML 元素語意與組合原理（表單與控制）  | 使用正確的 <label>、<fieldset> 組合表單             |
| W5       | HTML 元素語意與組合原理（互動與媒體）  | 製作無障礙互動元件（按鈕、Carousel）與 img fallback |
| W6       | Design Token 設計與主題切換機制        | 完成一組完整的 token 架構與 dark/light 切換         |
| W7       | Props / State / Context 模組化設計     | 釐清狀態與控制的設計原則並封裝控制型元件            |
| W8       | Virtual DOM、Render Decision Tree 原理 | 自畫 render 流程圖並解釋 reconciliation             |
| W9       | Component Re-render Trigger 與效能策略 | 撰寫性能陷阱重現與 memo 最佳實踐                    |
| W10      | 高複雜元件抽象設計與重用策略           | 拆出抽象 form / table 與封裝複雜 slot 邏輯          |
| W11      | Accessibility 策略與測試工具           | 使用 Axe、Lighthouse 檢測元件可達性                 |
| W12      | Storybook 驅動的文件化與控制文檔生成   | 整合 Controls / DocsPage 與組織 token 展示          |
| W13      | Web Component 與跨框架元件實作         | 撰寫純 HTML 自定義元件並在三大框架中使用            |
| W14      | Snapshot Testing 與 CI 驗證流程        | 撰寫單元 / 視覺測試並整合 GitHub Actions            |
| W15      | Nx / Monorepo 結構規劃與分層策略       | 元件包 / token 包 / util 包分層與共用策略           |
| W16      | Component Governance 與命名策略        | 撰寫元件命名規則、deprecated 流程與內部維運 wiki    |
| W17      | 🎓 Final Project 設計週｜設計系統實作  | 架構一套完整 Design System + 實作 + 測試 + CI       |
| W18      | 🎓 架構提案週｜設計原則與維運簡報      | 發表元件設計、測試、佈署、版本策略與成果頁面        |
| W19      | 🎉 成果展示與互評                      | 同儕互評 + QA 討論 + 改版建議                       |
| W20      | 🎯 Final Review + 課程總結             | 自我評估 + 能力地圖映射 + 學習歷程檔案輸出          |

[**W1 | 為什麼需要設計系統與模組化？**](https://www.notion.so/W1-215d54ae00a6802480afd73920045bcf?pvs=21)

[W2 | Atomic Design 與元件分層原則](https://www.notion.so/W2-Atomic-Design-21bd54ae00a680a0bfc6fb3dd0cade95?pvs=21)

[W3 | HTML 元素語意與組合原理（結構性）](https://www.notion.so/W3-HTML-223d54ae00a680b88407caa49a387d29?pvs=21)
```

```md
# 架構風格概念理論

🔁 課程定位：

本課程面向具有中高階系統設計經驗者，將涵蓋業界主流架構風格（Clean Architecture、DDD、Hexagonal、CQRS、Event Sourcing、TDD、BDD 等），並輔以架構風格與應用情境對應訓練，培養可繪製與解釋企業級架構設計圖的能力。

學期長度：20 週（含假期與彈性整合週） / 共計 5 學分

每週預估投入時間：約 6 小時

🎯 結業目標：

- 具備多種現代架構風格的設計思維與依賴策略掌握
- 能針對各架構風格，分別列出 3 種主要 + 2 種次要適配情境
- 針對任一系統情境繪製完整設計圖並說明 trade-off 原則

📚 教材核心：

- 《Clean Architecture》by Robert C. Martin
- 《Domain-Driven Design》by Eric Evans
- 《Implementing Domain-Driven Design》by Vaughn Vernon
- 《Architecture Patterns with Python》
- 《The Art of Scalability》by Abbott & Fisher
- 《Fundamentals of Software Architecture》by Neal Ford & Mark Richards
- 《DDD Distilled》by Vaughn Vernon
- 官方資源：CQRS & Event Sourcing Reference， Hexagonal Architecture 實例專案

🧭 課程全覽：

| **週次** | **主題名稱**                             | **任務重點 / 進度檢核**                               |
| -------- | ---------------------------------------- | ----------------------------------------------------- |
| W1       | 課程導論與現代架構演進概覽               | 認識各架構哲學與應用戰場、繪製演進圖譜                |
| W2       | Clean Architecture 深入原則              | Entities / UseCases / Adapters / Framework 概念設計稿 |
| W3       | Clean Architecture 主要應用情境演練      | ERP / 訂單系統 / 多模組 CMS 架構圖分析                |
| W4       | Clean Architecture 次要應用 + Trade-off  | Ticket / 資料服務為例分析設計限制與延展方案           |
| W5       | DDD 核心建模邏輯                         | Entity / VO / Aggregates / Bounded Context 設計演練   |
| W6       | DDD 的語意建模實戰                       | Logistics / 交易平台 / 多租戶帳務系統分析             |
| W7       | Context Mapping 與 ACL / 跨界限策略      | 建立 Anti-Corruption Layer 圖解與設計思維練習         |
| W8       | Hexagonal Architecture（六邊形架構）     | Ports / Adapters 拆解與實例說明                       |
| W9       | Hexagonal 應用情境分析                   | 多入口應用、Scheduler、外部 CLI 整合範例              |
| W10      | CQRS + Event Sourcing 理論與實作範式     | Command / Query 隔離設計圖、事件流程設計              |
| W11      | CQRS 實務應用領域演練                    | 秒殺活動系統 / IoT 控制平台設計說明稿                 |
| W12      | TDD / BDD 設計哲學與驅動開發方法         | 測試層責任與應用層驅動開發策略圖解                    |
| W13      | TDD / BDD 在架構決策中的角色             | 案例討論 + BDD 的開發流程分工實戰稿                   |
| W14      | Screaming / Plugin / Layered Patterns    | 如何建構語意清晰與模組擴展友善系統                    |
| W15      | 各風格適配矩陣與 trade-off 策略圖整理    | 建立設計決策矩陣與案例對應表                          |
| W16      | [設計演練週 1] Clean / DDD 獨立架構提案  | 實例繪製 + 設計說明簡報準備                           |
| W17      | [設計演練週 2] CQRS / Hexagonal 架構草圖 | 多模組整合下的選型與設計稿製作                        |
| W18      | 假期週與補進度整合週                     | 彈性安排與補完閱讀任務                                |
| W19      | Final Design Jam ｜三種情境模擬設計      | 每人輸出一套架構決策與說明                            |
| W20      | 🎓 成果發表會 + 架構問答環節             | 簡報提案 + Peer Review + 問答演練                     |

📌 備註：

- 各風格皆對應 3+2 應用場景，資料與討論素材將於每週補充
- 實作需求僅限於圖解與文件產出，無需程式碼開發
- 架構圖可使用 C4 / Mermaid / Whimsical / Diagrams.net 製作
- Final 成果需涵蓋任一實戰情境的架構草圖 + 設計 rationale（說明文件）

[**W1 ｜課程導論與現代架構演進概覽**](https://www.notion.so/W1-215d54ae00a6801e9426c531ac88c634?pvs=21)

[ W2 ｜ Clean Architecture 深入原則](https://www.notion.so/W2-Clean-Architecture-21cd54ae00a680ba9edaf1cd8f18b3d9?pvs=21)

[W3 | Clean Architecture 主要應用情境演練 ](https://www.notion.so/W3-Clean-Architecture-223d54ae00a680fea688d438caa02fa3?pvs=21)
```

```md
# 版本控管藝術-Git Flow 與 CI/CD 自動化流程建構

🔁 **課程定位**：

涵蓋 Git 分支策略、CI/CD 自動化建構與部署、容器化實作、流程設計策略與 DevOps 組織文化實踐，符合企業級應用與高階技術職面試需求。

📆 **課程長度**：18 週

🎓 **學分建議**：4 學分

📚 **教材核心**：

- 《DevOps Handbook》
- 《Accelerate》
- 《CI/CD for Monorepos at Scale》
- 《Docker: Up & Running》
- GitHub Actions / GitLab CI / Jenkins 官方文檔
- AWS / GCP CI/CD 部署案例集

---

✅ 週次課表與整合說明（共 18 週）

| **週次** | **主題名稱**                          | **重點內容與整合補充說明**                                                            |
| -------- | ------------------------------------- | ------------------------------------------------------------------------------------- |
| W1       | Git 基礎與 commit 架構                | 熟練 reset/revert/staging；版本還原流程                                               |
| W2       | Git Flow 與 Trunk-Based 開發策略      | 分支策略比較，實作 PR 流程範本                                                        |
| W3       | 團隊 PR 流程設計與審查規範            | 撰寫審查條件、Merge request 守則                                                      |
| W4       | Git 衝突管理與版本恢復技巧            | 多人合作情境模擬與版本回復                                                            |
| W5       | CI/CD 流程與架構設計                  | 圖解 Build-Test-Deploy Pipeline                                                       |
| W6       | CI/CD 選型策略（平台 + 架構）         | 比較 GitHub Actions / Jenkins / GitLab CI                                             |
| W7       | Task vs Job 邊界與流程分界設計        | **原子任務拆分原則、retry 模式與 job 邏輯邊界，對應 CI 工具中 jobs/steps 的差異實作** |
| W8       | GitHub Actions CI 流程實作            | 編寫 Lint、Unit test、Code Coverage 之 step                                           |
| W9       | CD 流程與環境部署（含 Secrets）       | 多環境 matrix 設計、環境變數與密鑰管理                                                |
| W10      | Build 快取與 Job 條件控制             | cache、workflow_run、matrix 條件分支                                                  |
| W11      | Pre-merge 驗證與差異檢查設計          | Required Status Check / Snapshots                                                     |
| W12      | Dockerfile 與映像建構策略             | multi-stage 建構、語言模板撰寫練習                                                    |
| W13      | Docker Compose 整合測試流程           | 模擬 Redis + DB + App 測試流程整合                                                    |
| W14      | 映像推送與版本治理策略                | 撰寫 tag 策略、Git tag vs Docker tag 對應                                             |
| W15      | 容器自動部署與回滾策略設計            | ECS / Cloud Run / Firebase Hosting 實作                                               |
| W16      | DevOps 指標（DORA）與文化治理         | MTTR / deploy frequency 對應現有流程現況                                              |
| W17      | Final Project：CI/CD 流程統整實作     | 實作個人或團隊專案的完整流程，自評 bottleneck 與彈性調整                              |
| W18      | 架構簡報 + Flow map 發表 + 職涯對照表 | 技術簡報 + 能力地圖 + 職涯成熟度回饋                                                  |

[W1 ｜ Git 基礎與 Commit 架構：掌握版本還原與狀態管理技術](https://www.notion.so/W1-Git-Commit-216d54ae00a6803aa3c8c133a3d90b32?pvs=21)

[W2 ｜ Git Flow 與 Trunk-Based 策略比較實作](https://www.notion.so/W2-Git-Flow-Trunk-Based-21dd54ae00a6807abca5cf40192f1cc8?pvs=21)

[W3 | 團隊 PR 流程設計與審查規範](https://www.notion.so/W3-PR-223d54ae00a68033a043e0c2547da31f?pvs=21)
```

```md
# 雲端高可用架構設計與實戰

🔁 **課程定位：**

培養能獨立設計並實作企業級 AWS 與 GCP 雲端架構的工程師，具備高可用性（HA）、災難復原（DR）、IaC、自動化部署、成本優化與雲端安全能力。課程最終目標亦涵蓋一張雲端架構師等級證照（SAA / ACE）。

📆 **學期長度：** 20 週（建議每週 4–6 小時）

🎓 **學分數：** 5 學分

🎯 **結業目標：**

- 能獨立設計具 HA / DR 的雲端架構圖與成本模型
- 實作 AWS / GCP 核心資源（VPC、EC2、RDS、Lambda、GCS、Cloud Run 等）
- 掌握 Terraform / CDK 進行模組化架構部署
- 熟悉 DevOps 整合、Serverless 設計與跨平台整合思維
- 完成至少一張 AWS SAA 或 GCP ACE 證照考取準備

📚 **教材核心：**

- 《The Art of Scalability》
- 《Cloud Architecture Patterns》
- AWS 官方白皮書：Well-Architected Framework / HA Design / DR Strategy
- GCP 官方架構圖庫與案例
- Terraform / AWS CDK / Google Cloud Deployment Manager
- 工具：AWS Console， GCP Console， GitHub Actions， CodePipeline， CloudWatch， Cloud Monitoring

---

🗺️ 課程全覽

| **週次** | **主題名稱**                   | **任務重點 / 進度檢核**                             |
| -------- | ------------------------------ | --------------------------------------------------- |
| W1       | 雲端架構設計導論與企業應用藍圖 | AWS vs GCP 架構哲學與設計原則比對                   |
| W2       | 雲端核心資源設計               | EC2 vs GCE、S3 vs GCS、VPC 原則與對照               |
| W3       | 高可用性與多區部署策略         | Auto Scaling， ALB/ELB， GCP Region/Zone 配置       |
| W4       | 容錯與災難復原（DR）設計       | RTO/RPO、資料同步、跨區 Failover                    |
| W5       | 資料庫與分散式儲存設計         | RDS / DynamoDB / Spanner 等 HA 設計比較             |
| W6       | 監控與可觀測性導入             | CloudWatch / Cloud Monitoring 與 SLA 對應           |
| W7       | IAM 與帳號權限架構治理         | Organizations， Policies， Least Privilege 實踐     |
| W8       | 混合雲與 VPN 架構              | Direct Connect / Interconnect 與混合架構案例        |
| W9       | 🔍 小考週                      | 繪製雲端架構圖 + 說明高可用與容錯策略               |
| W10      | 成本控制與資源最佳化設計       | RI / Savings Plan、Billing 報表與效能比分析         |
| W11      | 基礎 IaC 實作 I（Terraform）   | 建立 VPC / EC2 / S3 Module                          |
| W12      | 進階 IaC 實作 II（CDK）        | 多環境部署、State 管理與資源依賴治理                |
| W13      | DevOps 自動化部署實踐          | GitHub Actions / CodePipeline × Terraform 整合      |
| W14      | 雲端安全設計與資安防護         | IAM Policies， WAF， Shield， KMS， Secrets Manager |
| W15      | Serverless 與事件驅動設計      | Lambda / Cloud Functions， Pub/Sub， EventBridge    |
| W16      | 專案實作 I：AWS                | 架構設計 + IaC + Monitoring + 成本預估              |
| W17      | 專案實作 II：GCP               | 架構設計 + IaC + DR/HA 策略 + 成本預估              |
| W18      | 多雲整合與轉換策略             | 跨平台架構對照、Hybrid Design 模式                  |
| W19      | 🔍 證照準備週                  | SAA / ACE 模擬考 + 題型解析與知識補強               |
| W20      | 🎓 成果發表會                  | 提案簡報 + 架構圖 + 成本模型 + 問答演練             |

[W1 ｜雲端架構導論與企業應用藍圖](https://www.notion.so/W1-216d54ae00a68019a50ac41c8bc5fa1c?pvs=21)

[W2 | 雲端核心資源設計](https://www.notion.so/W2-21dd54ae00a6805ea0ebcfc20adc5b54?pvs=21)

[W3 | 高可用性與多區部署策略](https://www.notion.so/W3-223d54ae00a680e18337e2775382d3b0?pvs=21)
```

```md
# 系統可觀測性與錯誤追蹤設計

🔁 **課程定位**：

針對高併發、多模組、微服務架構下的可觀測性設計進行系統訓練，整合 metrics/logs/traces 的治理策略、實作技能與異常排查能力，具備從零建構 observability solution 的架構眼光與落地能力。

📆 **學期長度**：18 週

📚 **建議學分**：4 學分

🎯 **結業目標**：

- 能設計跨模組、具業務語意的 observability 架構
- 建立從 trace 傳遞、log 設計到 MTTR 減少的可追蹤性解法
- 具備 incident drill、故障排查與 SLO 設計的實戰能力

**🤖 核心教材**:

《Observability Engineering》（Charity Majors）

《Site Reliability Engineering》（Google SRE Book） Ch14–17

Jaeger / Tempo / Grafana / Loki / Prometheus / OpenTelemetry

Datadog / CloudWatch / X-Ray / Firebase Performance

《Monitoring Distributed Systems》（O’Reilly）

---

🧭 課程全覽：週次｜主題名稱｜學習目的與檢核焦點

| **週次** | **主題名稱**                               | **學習目標與任務檢核**                                  |
| -------- | ------------------------------------------ | ------------------------------------------------------- |
| W1       | 課程導入與可觀測性為何重要                 | 定義 observability vs monitoring，導出三支柱模型        |
| W2       | 可觀測性架構全景圖與現代工具鏈分類         | 比較 Grafana Stack / ELK / APM / Cloud-native 套件      |
| W3       | Metrics 設計：指標分類與量測策略           | 建構 request rate， error rate， latency 等核心指標     |
| W4       | Logs 設計：語意、結構化與搜尋性            | 實作 log format 標準（如 trace_id、tag、語意）          |
| W5       | Traces 與 Span：分布式系統中的流程鏈路設計 | 實作 openTelemetry / Jaeger 基礎應用                    |
| W6       | Dashboards 設計原則與資料可讀性規劃        | 設計業務導向儀表板（Grafana 或 Datadog）                |
| W7       | Logs/Traces 整合與 root cause 排查實作     | 跨服務錯誤模擬 → 排查流程與記錄提交                     |
| W8       | 小考週：三支柱概念理解 + logs/traces 實作  | YAML 定義、log span 關聯、incident 判讀                 |
| W9       | 警報策略設計：Threshold、Anomaly、Mute     | 撰寫多層級 alert rule，設計 alert fatigue 緩解機制      |
| W10      | SLO / SLI / SLA：業務語意化的可觀測策略    | 制定以用戶體驗為核心的可衡量目標                        |
| W11      | Trace ID 傳遞機制與 HTTP context 延續設計  | 模擬跨微服務 trace context 傳遞策略                     |
| W12      | Serverless 與無 agent 架構的觀測方案       | 對應 Lambda / Cloud Function / Firebase 的解法實作      |
| W13      | Observability as Design Component          | 💡 觀測性視角設計：將 observability 作為架構圖組件處理  |
| W14      | Log 語意與事件模板治理                     | 💡 事件分類與上下文結構治理，建立 log span template     |
| W15      | Incident Drill 導入與演練準備              | 案例模擬：request 停滯 + 錯誤無 log → 排查演練          |
| W16      | Incident Drill 實作與報告                  | 撰寫完整錯誤報告 + observability gap 與建議調整策略     |
| W17      | Final Project 整合與效能分析               | 彙整 logs/metrics/traces 成一份分析報告與可觀測策略輸出 |
| W18      | 架構簡報 + 成果互評                        | 10 分鐘報告：系統觀測架構設計解說 + 成果評量與自評問卷  |

[W1 ｜課程導入與可觀測性核心概念建立](https://www.notion.so/W1-217d54ae00a6802f837acf6486be3f7c?pvs=21)

[W2 | 可觀測性架構全景圖與現代工具鏈分類](https://www.notion.so/W2-21ed54ae00a6800489dcecee0484bb22?pvs=21)
```

```md
# 資料庫 Schema 與 Index 設計

🔁 **課程定位**：

針對資深全端工程師（FSE）所設計，從業務建模、查詢優化到 AI 導向資料設計，涵蓋企業級交易、資料一致性、性能與可維運性挑戰。適合作為核心資料工程實力養成之用。

📆 **學期長度**：22 週（含假期與預備週），總學分：5 學分

🎯 **結業目標**：

- 能獨立設計可擴展、高可用之資料表結構與索引策略
- 應對多表寫入、高併發、時間序列與 OLAP/OLTP 負載問題
- 搭配業務建模思維，設計具 AI 訓練/預測能力之資料結構

📚 **教材核心**：

- 《Designing Data-Intensive Applications》
- 《High Performance MySQL》 / 《PostgreSQL Internals》
- 《Building Microservices》
- Uber / Airbnb Schema & Data Flow 實例
- Google Feature Store / Michelangelo 架構白皮書

---

🧭 課程全覽

| **週次** | **主題名稱**              | **任務重點 / 進度檢核**                           |
| -------- | ------------------------- | ------------------------------------------------- |
| W1       | 資料建模核心與課程總覽    | 業務轉資料的視角建立，表結構設計全貌              |
| W2       | 正規化與去正規化策略      | 從 1NF–3NF 到 schema 優化與表拆解決策             |
| W3       | 業務流程 → 聚合建模       | 聚合根與資料一致性邊界設定（搭配 DDD）            |
| W4       | 表格與物件設計哲學        | 命名、型別、上下文分離與擴展性邊界考量            |
| W5       | 交易建模與一致性方案      | ACID、eventual consistency、boundary 標示法       |
| W6       | Index 設計 I              | BTree / Hash / bitmap index 機制與應用場景        |
| W7       | Index 設計 II             | 複合索引、Partial、Covering Index 實作與選型      |
| W8       | 查詢效能優化              | Explain Plan 解讀、慢查詢追蹤、反模式範例         |
| W9       | 分區 / 分表設計策略       | 分區 key、水平 / 垂直切分與資料熱點處理           |
| W10      | 時序資料設計策略          | TTL、事件流儲存、歷史資料落地與歸檔架構           |
| W11      | 資料遷移與版本治理        | Schema Migration 策略 + Version Table 設計        |
| W12      | 查詢與資料轉換治理        | ETL / ELT 專用 schema 與中介層轉換策略            |
| W13      | SQL vs NoSQL 融合         | Mongo、Redis 結構設計與整合架構                   |
| W14      | 冷熱資料分層設計          | Hot/Cold Path 設計與快取策略對應                  |
| W15      | 多表寫入與高併發處理      | 股票交易範例：補償策略、多表一致性實作            |
| W16      | Outbox 與事件一致性模式   | 非同步資料落地 / 消息傳遞一致性設計               |
| W17      | AI 資料設計與特徵倉儲策略 | Feature Store、標記結構、ETL 資料版本管理         |
| W18      | 表結構重構與技術債清理    | 歷史遷移、兼容查詢設計與 refactor 方法論          |
| W19      | 期中總結與專題題目核發    | Schema 設計方向確認與團隊分組                     |
| W20      | 商業場景建模演練          | 訂票系統 / 活動報名系統的資料建模實戰             |
| W21      | 假期週（Reading Week）    | DIA 第 10-12 章 + Michelangelo/FeastureStore 研究 |
| W22      | 資料一致性驗證與測試      | 合約驗證、資料快照驗證與同步模擬                  |
| W23      | 期末預備 I                | Schema + Index 設計、使用情境文檔                 |
| W24      | 期末預備 II               | 壓測分析、快取策略、觀測與報告生成                |
| W25      | 🎓 結業發表週             | 專案發表、互評、總結與回饋                        |

[W1 ｜資料建模核心與 Schema 設計全貌](https://www.notion.so/W1-Schema-217d54ae00a6807e9076c85cafbd3adc?pvs=21)

[W2 | 正規化與去正規化策略設計](https://www.notion.so/W2-21ed54ae00a68058b25df10440a6f5f7?pvs=21)
```

```md
# 複雜狀態管理策略

🔁 **課程定位**：

本課程針對**資深前端與全端開發者**，系統化建構複雜狀態管理的工程能力與架構設計能力，對應實戰應用場景包括企業級業務系統、跨角色協作場景與前後端同步一致性挑戰。

📚 **學期長度與學分**：4 學分｜ 18 週密集訓練課程

🎯 **結業目標**：

- 精通 Redux 與 NgRx 架構原理與狀態流設計
- 能夠獨立設計符合大型系統需求的狀態管理方案
- 能理解並實作前後端一致性策略（如 Optimistic Update、Server Sync）

📘 **核心教材**：

- 《Redux Essentials》（Redux 官方文檔 + RTK）
- 《NgRx Component Store & Effects Patterns》
- 《Fullstack React》、Angular 官方指南
- 實戰專案（訂單管理系統、多角色看板、線上交易同步）

---

🧭 課程週次與主題安排（預覽版）

| **週次** | **主題名稱**                                  | **重點任務 / 檢核**                             |
| -------- | --------------------------------------------- | ----------------------------------------------- |
| W1       | 課程導論：狀態的本質與系統定位                | 解構 UI → Data → Logic → State 的關係模型       |
| W2       | Redux 與 NgRx 思想源流與設計哲學              | Redux vs NgRx 架構設計圖解析                    |
| W3       | Redux 架構與 RTK 實作精要                     | 建立 Store、Slice、Selector、Middleware         |
| W4       | NgRx 架構與 Store 模組設計                    | 建構 Action、Reducer、Selector、Effect          |
| W5       | Component Store 與 ViewModel 模型             | NgRx 輕量替代方案、元件解耦實作                 |
| W6       | 表單與複雜輸入場景管理                        | FormGroup 狀態、dirty/valid/state sync          |
| W7       | 多模組狀態分區與邊界治理                      | Bounded Context、Slice 切割策略                 |
| W8       | 🔍 小考週｜狀態同步策略設計與 Debug Flow 實作 | 問題模擬與 DevTool 排錯演練                     |
| W9       | 多角色 UI 狀態分流（角色/權限/視圖）          | 探討 Contextual State 與可共用/隔離設計         |
| W10      | 業務驅動狀態機構建（如訂單 / 任務流程）       | 對應業務流程建構狀態圖與管理策略                |
| W11      | API 回應同步策略（CRUD / debounce / loading） | async pipe / entity adapter / loading indicator |
| W12      | Optimistic UI 與離線狀態處理策略              | rollback， patch， staging state 設計           |
| W13      | 跨設備與實時協作場景（Socket / SSE）          | 可觀測狀態、同步設計圖、錯誤回復策略            |
| W14      | NgRx DevTools / Redux DevTools 深度應用       | 自訂 Middleware / meta-reducer 實作             |
| W15      | 狀態與效能：Memoization 與 Virtual State      | Selector Memo / Lazy Load / Store 分段載入      |
| W16      | 🔍 最終演練週｜狀態管理架構設計與模擬提案     | 架構圖 + 任務流設計 + 優化說明文件              |
| W17      | 專案週｜實作中型系統（複雜狀態與前後端 sync） | 評量：整合度、維護性、解耦能力                  |
| W18      | 🎓 成果發表週｜狀態管理設計競演與互評         |                                                 |

[W1 ｜狀態的本質與系統定位](https://www.notion.so/W1-218d54ae00a680f8b9cafa6cfe10fcc2?pvs=21)

[W2 | Redux 與 NgRx 思想源流與設計哲學](https://www.notion.so/W2-Redux-NgRx-21ed54ae00a680f79d55d4ca6fa789c2?pvs=21)
```

```md
# UX Flow 與痛點辨識

🔁 **課程定位：**

強化產品開發中「以使用者為中心」的流程視角，掌握從使用者需求出發的體驗設計策略，結合 UX Flow 分析、使用者研究、痛點洞察與迭代優化技巧，適用於 PM、UX Designer 與全端工程師。

📚 **學分數：** 4 學分

📆 **學期長度：** 18 週（每週建議投入 3 ～ 5 小時）

🎯 **結業目標：**

- 熟悉 UX Flow 分析與可視化設計
- 能針對使用者場景辨識痛點並提出可行改進方案
- 能獨立設計使用流程、動線圖與交互模型
- 能配合設計 / 技術團隊落實需求驗證與使用者測試

📘 **教材核心：**

- 《Don't Make Me Think》by Steve Krug
- 《The Design of Everyday Things》by Don Norman
- 《Lean UX》by Jeff Gothelf
- Nielsen Norman Group – UX Research Methods
- UXPin， Figma， Whimsical
- 使用者訪談 / 可用性測試範本（實務彙編）

---

🧭 課程全覽

| **週次** | **主題名稱**                        | **任務重點 / 進度檢核**                 |
| -------- | ----------------------------------- | --------------------------------------- |
| W1       | 認識 UX Flow                        | 理解 UX Flow 與業務流程差異             |
| W2       | 使用者導向設計觀                    | 學習 Don Norman 使用者模型              |
| W3       | 使用情境分析（Scenario Mapping）    | 建構任務流程與痛點假設                  |
| W4       | 使用者流程圖實作（Task Flow）       | 建立第一版 UX Flow 圖                   |
| W5       | 使用者訪談方法與技巧                | 設計問題清單與訪談流程                  |
| W6       | 使用者行為觀察（Usability Test）    | 撰寫觀察紀錄與分析報告                  |
| W7       | 痛點洞察與 JTBD（Jobs to be Done）  | 建立問題敘述與行為動機圖                |
| W8       | UX Flow 優化模型（Flow Efficiency） | 優化重工路徑與瓶頸視覺化                |
| W9       | 🔍 小考週：UX Flow 實作與反饋       | 任務流程圖製作 + 痛點假設說明           |
| W10      | Figma / Whimsical UX Flow 實作      | 團隊共同建構可互動流程                  |
| W11      | 使用者故事與流程整合                | 建立 UX Flow × User Story 對應          |
| W12      | A/B Test 與 UX 實驗設計             | 撰寫測試假設與實驗變數配置              |
| W13      | 可用性測試分析與指標（SUS / TCR）   | 分析痛點數據並導入優化建議              |
| W14      | UX Flow × 開發邏輯對應              | 與開發對接邏輯點與參數傳遞節點          |
| W15      | 專案整合：建立一個完整流程設計案    | 練習從使用者訪談 → 痛點 → Flow 完成設計 |
| W16      | 成果簡報與回饋準備                  | 撰寫使用者需求對應圖與建議簡報          |
| W17      | 🎓 專案簡報發表會（演練）           | 發表 UX Flow 與痛點優化計畫             |
| W18      | 🎓 同儕回饋與課程總結               | 反思學習成長、互評與自評任務            |

[W1 ｜認識 UX Flow 與業務流程差異](https://www.notion.so/W1-UX-Flow-218d54ae00a680c0b035dd33933ca1e7?pvs=21)

[W2 | 使用者導向設計觀](https://www.notion.so/W2-21ed54ae00a6809eb601d192d42f3c9b?pvs=21)
```

```md
# A/B 測試與成長分析

🔁 課程定位： 培養具備資料驅動決策能力的產品人與工程師，能從假設建構、實驗設計、結果判讀到成長策略規劃，獨立主導跨部門的實驗流程與成長模型設計，達成產品成效優化。

學期長度：18 週（建議每週投入 3~5 小時） 學分數：4 學分

結業目標：

- 能獨立設計與執行產品實驗，並正確詮釋統計顯著性
- 熟悉 A/B 測試的假設建構、樣本計算與分析報告撰寫
- 能設計與推演 Growth Loop 串連策略性實驗週期
- 熟悉可用工具（Amplitude、GA4、Mixpanel）並能實作儀表板
- 能以管理語言報告成效，並具備實驗設計哲學與倫理理解

教材核心：

- 《Trustworthy Online Controlled Experiments》
- 《Lean Analytics》
- 《Growth Hacking》by Sean Ellis
- 《Experimentation Works》by Stefan Thomke
- Amplitude / Mixpanel / Google Optimize / GA4
- 儀表板與實驗報告模板（實務彙編）

🧭 課程全覽：

| **週次** | **主題名稱**                | **任務重點 / 進度檢核**                  |
| -------- | --------------------------- | ---------------------------------------- |
| W1       | 什麼是 A/B 測試與 Growth？  | 案例導入、成長與假設測試的區別           |
| W2       | 成長指標與框架              | HEART / AARRR / North Star 指標設計      |
| W3       | 假設建構哲學與驗證模型      | 撰寫具備商業邏輯的假設與驗證邏輯         |
| W4       | 樣本數計算與顯著性原理      | power analysis、信賴區間與錯誤類型       |
| W5       | 常見錯誤與 p-hacking 避免   | 測試陷阱與統計迷思澄清                   |
| W6       | 實驗設計流程與倫理          | 對照組設計、變因控管、實驗影響評估       |
| W7       | 測案情境設計哲學            | 設計適切問題、關聯業務目標、減少偽陽性   |
| W8       | Growth Loop 策略推演        | 單次 A/B → 週期性優化模型構建            |
| W9       | 🔍 小考週                   | 假設撰寫、樣本計算、實驗流程設計筆試     |
| W10      | 分析工具實作 I（Amplitude） | 建構 funnel 與轉換儀表板                 |
| W11      | 分析工具實作 II（GA4）      | 事件追蹤與目標轉換設定                   |
| W12      | 工具整合與資料可信性        | 埋點一致性、UTM 標籤與資料清洗           |
| W13      | 實驗資料分析與解讀報告      | 平均 / 中位 / 區間 / 比例轉換與視覺圖解  |
| W14      | 多變數測試與疊加實驗設計    | MVT、A/B/n 與連續實驗設計注意事項        |
| W15      | 跨部門溝通與報告技巧        | 以管理語言簡報轉換率、成長模型與建議     |
| W16      | 模擬實作 I                  | 自選實驗主題，撰寫完整測案與驗證設計文件 |
| W17      | 模擬實作 II                 | 出具數據分析報告與成長策略建議書         |
| W18      | 🎓 成果發表會               | 簡報展示 + 成果提案 + 同儕回饋           |

[W1 ｜什麼是 A/B 測試與 Growth？](https://www.notion.so/W1-A-B-Growth-218d54ae00a680238f6dfca2b503af54?pvs=21)

[W2 | 成長指標與框架](https://www.notion.so/W2-21ed54ae00a68081a914dd9538aa1a81?pvs=21)
```

```md
# 跨部門產品需求拆解與流程建構

🔁 **課程定位：**

融合產品經理核心技能與資深開發者溝通能力，強化 AI 時代下 PM × Design × Dev 的協作能力。聚焦跨部門溝通語言、需求拆解技術、流程設計邏輯與 MVP 推演方法。

📚 **學分數：** 4 學分

📆 **學期長度：** 18 週（建議每週投入 3 ～ 5 小時）

🎯 **結業目標：**

- 能清晰拆解產品需求並繪製產品流程圖
- 熟練 MVP 設計與優先順序決策技巧
- 能與設計與開發有效協作並轉譯需求語言
- 熟悉常見 PM 工具與產品開發流程語言（user story / flow / spec / C4 model）

📘 **教材核心：**

- 《Inspired》by Marty Cagan
- 《User Story Mapping》by Jeff Patton
- 《Lean UX》by Jeff Gothelf
- 《Sprint》by Jake Knapp (GV)
- C4 Model / Event Storming / Miro
- 訪談腳本與需求採集手冊（實務彙編）

---

🧭 課程全覽

| **週次** | **主題名稱**               | **任務重點 / 進度檢核**                  |
| -------- | -------------------------- | ---------------------------------------- |
| W1       | 產品角色與協作模型         | 繪製 PM × Design × Dev 協作藍圖          |
| W2       | 產品需求是什麼？           | 定義需求與非需求、業務 vs 技術語言       |
| W3       | 使用者故事與 Persona 設計  | 建立使用者視角與目標導向需求語言         |
| W4       | User Story Mapping 工作坊  | 實作產品故事線與 MVP 拆解                |
| W5       | 需求訪談技巧與工具         | 撰寫訪談腳本，導出關鍵問題與需求         |
| W6       | MVP 構建與優先順序邏輯     | RICE / MoSCoW / Kano 等方法實作          |
| W7       | 流程圖繪製與系統邏輯拆解   | 使用 Miro / Lucidchart 繪製多角色流程    |
| W8       | 跨部門需求協作模型練習     | 演練 Dev × PM × Design 共識流程          |
| W9       | 🔍 小考週                  | 需求理解與故事地圖繪製（筆試 + 圖像）    |
| W10      | 文件規格撰寫（PRD / Spec） | 學習規格書撰寫語法與格式                 |
| W11      | 產品週期與迭代計畫設計     | 以 Sprint 為單位推演開發節奏             |
| W12      | 從目標到指標（OKR / KPI）  | 將需求目標化並對應商業指標               |
| W13      | 產品需求演化與版本治理     | 進行需求版本控制與歷程追溯               |
| W14      | 與技術人員的翻譯技術       | 練習需求轉技術語言（C4 model 概覽）      |
| W15      | 專案模擬演練 I             | 實戰需求訪談與流程建構（以模擬專案為例） |
| W16      | 專案模擬演練 II            | 規格書撰寫與角色共識建立                 |
| W17      | 期末簡報準備               | 製作產品發想、流程、需求拆解簡報         |
| W18      | 🎓 成果發表會              | 完成簡報發表 + 同儕互評 + 自評報告       |

[W1 ｜產品角色與協作模型](https://www.notion.so/W1-218d54ae00a6804fa2d8db08b3870be8?pvs=21)

[W2 | 產品需求是什麼？](https://www.notion.so/W2-21ed54ae00a6802cb453c236af6a9b8e?pvs=21)
```
