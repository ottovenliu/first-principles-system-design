# Day 2-2 | 需求確認 × 系統設計起點(二)：領域邊界與基礎需求確認

昨天我們從用戶的話語中萃取出了功能需求的本質，理解了不同認知模式下系統存在的目的。但當我們準備將這些抽象理解轉化為具體的 AWS 架構時，會發現一個關鍵問題：

**功能需求回答了「做什麼」，但沒有回答「做到什麼程度」。**

當投資交易用戶說「我怕速度慢會虧錢」時，他們真正的意思是延遲要控制在多少毫秒內？當家庭財務用戶說「隨時整理開銷」時，系統要支持多少併發用戶？當健康監控用戶說「不能影響判斷」時，可接受的停機時間是多久？

這些就是**非功能需求**的領域，它們決定了系統的技術邊界和 AWS 服務的具體配置。

## 非功能需求的本體論：從質性到量性的轉化

### 三個層次的技術邊界

**性能邊界**（Performance Boundaries）

- API 每秒請求數（RPS）的上限與下限
- 響應時間的期望值與容忍範圍
- 數據處理吞吐量的峰值與平均值

**容量邊界**（Capacity Boundaries）

- 用戶增長的預期曲線與峰值規劃
- 數據存儲的增長模式與歷史保留
- 計算資源的彈性擴展與成本控制

**韌性邊界**（Resilience Boundaries）

- 可接受的故障恢復時間（RTO）
- 可容忍的數據遺失程度（RPO）
- 系統降級的優雅範圍與核心功能保障

### 從 Domain 事件推導技術規格

每個領域都有其獨特的**技術節奏**，這個節奏決定了系統的性能特徵：

**金融交易領域**：微秒級的競爭節奏

- 市場數據更新：每秒千次級別
- 交易決策延遲：毫秒不可接受
- 持倉計算精度：小數點後 8 位

**家庭管理領域**：日常生活的協作節奏

- 支出記錄頻率：每日數次到數十次
- 同步延遲容忍：秒級可接受
- 數據一致性要求：最終一致性充足

**健康監測領域**：生理週期的監控節奏

- 設備數據同步：分鐘級到小時級
- 異常告警延遲：關鍵指標需即時
- 歷史數據保存：年級別的長期存儲

## API Request Per Second：從用戶行為到技術指標

### RPS 的哲學基礎

RPS 不是憑空估計的數字，而是從用戶行為模式中推導出來的必然結果。每個 API 調用背後都有其用戶意圖，而用戶意圖的頻率分佈決定了系統的負載特徵。

#### 投資交易系統的 RPS 推導

**用戶行為模式分析**：

**行為分層**：

- 普通投資者：每日查看持倉 2-5 次，交易 0-2 筆
- 活躍交易者：每日查看持倉 20-50 次，交易 5-15 筆
- 高頻交易者：每秒查看持倉，每分鐘可能交易

**時間分佈**：

- 市場開盤前 30 分鐘：流量 peak，是平時的 10 倍
- 交易時段：持續高流量，是平時的 5 倍
- 市場收盤後：逐漸回落到基線

**計算邏輯**：

- 普通用戶：5 API calls/day ÷ 8 hours = 0.17 RPS/user
- 活躍用戶：50 API calls/day ÷ 8 hours = 1.74 RPS/user
- 高頻用戶：3600 API calls/hour ÷ 3600 seconds = 60 RPS/user
- 假設用戶分佈：80%普通、15%活躍、5%高頻
- 總 RPS = (0.8 × 0.17 + 0.15 × 1.74 + 0.05 × 60) × total_users
- = (0.136 + 0.261 + 3) × total_users
- = 3.397 × total_users

**AWS 服務選型的 RPS 臨界點**：

- < 100 RPS：API Gateway + Lambda (Serverless 優先)
- 100-1000 RPS：ALB + ECS/EC2 (Container 或 VM)
- 1000-10000 RPS：ALB + Auto Scaling Groups
- > 10000 RPS：Custom Load Balancer + 多 AZ 部署

#### 家庭財務系統的 RPS 推導

**協作行為的時間聚集性**：

**家庭協作模式**：

- 每個家庭平均 3.5 人，其中 2 人頻繁使用
- 使用集中時段：晚餐後(19:00-21:00)，週末購物時段
- 協作衝突：多人同時記錄同一筆支出的概率

**計算邏輯**：

- 平日晚間：2 users/family × 5 records/2hours = 2.5 RPS/family/1000families
- 週末高峰：2.5 × 3 = 7.5 RPS/1000families
- 年度報稅季：7.5 × 2 = 15 RPS/1000families

#### 健康監控系統的 RPS 推導

**IoT 設備的數據推送模式**：

**設備類型與頻率**：

- 智能手環：心率每 30 秒，步數每 10 分鐘
- 血壓計：手動測量，每日 1-3 次
- 血糖機：手動測量，每日 2-6 次
- 智能體重計：手動測量，每週 2-3 次

**計算邏輯**：

- 手環：(120 + 6) API calls/day = 126/day = 0.0015 RPS/user
- 血壓計：3 API calls/day = 0.000035 RPS/user
- 血糖機：6 API calls/day = 0.00007 RPS/user
- 體重計：0.4 API calls/day = 0.000005 RPS/user
- 總計：≈ 0.002 RPS/user
- 但考慮數據同步的批次處理和異常情況重試：
- 實際 RPS = 0.002 × retry_factor × batch_factor ≈ 0.01 RPS/user

### API 設計的哲學：從介面到契約

API 不只是技術介面，更是**領域概念的程式化表達**。每個 API 端點都應該對應到領域中的一個有意義的操作。

**RESTful 的領域語意映射**：

**投資交易領域**：

- POST /portfolios/{id}/orders - 提交交易訂單
- GET /portfolios/{id}/positions - 查詢持倉狀態
- PATCH /accounts/{id}/risk-preferences - 更新風險偏好
- GET /market/quotes/{symbol} - 獲取市場報價

**家庭財務領域**：

- POST /families/{id}/expenses - 記錄支出
- GET /families/{id}/budget/status - 查詢預算狀況
- PUT /families/{id}/members/{member_id}/permissions - 設定權限
- GET /families/{id}/reports/{period} - 生成財務報告

**AWS API Gateway 的限流策略**：

```yaml
ThrottlingSettings:
  BurstLimit: 5000 # 突發流量上限
  RateLimit: 2000 # 穩定流量限制

UsagePlan:
  BasicTier:
    Throttle: { BurstLimit: 100, RateLimit: 50 }
    Quota: { Limit: 1000, Period: DAY }
  PremiumTier:
    Throttle: { BurstLimit: 1000, RateLimit: 500 }
    Quota: { Limit: 10000, Period: DAY }
```

## Capacity Planning：增長模式的系統思維

### 業務增長的模式識別

不同的業務有不同的增長曲線，這直接影響容量規劃策略：

**線性增長模式（家庭財務管理）**

- 特徵：用戶增長穩定，使用模式可預測
- 增長函數：capacity(t) = baseline + growth_rate × t
- AWS 策略：Reserved Instance + Predictable Scaling
- 成本優化：長期承諾換取成本節省

**指數增長模式（社交分享平台）**

- 特徵：病毒式傳播，流量波動極大
- 增長函數：capacity(t) = baseline × (1 + growth_rate)^t
- AWS 策略：Serverless + Auto Scaling + Spot Instance
- 成本優化：按需付費，快速彈性擴縮

**週期性增長模式（投資交易系統）**

- 特徵：與外部週期（市場開放時間）高度相關
- 增長函數：capacity(t) = baseline + seasonal_factor × sin(2π × t/period)
- AWS 策略：Scheduled Scaling + Predictive Scaling
- 成本優化：預測性擴容，避免突發流量衝擊

### 存儲容量的生命週期管理

**數據的時間價值衰減**：

每種數據都有其存在的時間意義和訪問模式：

**投資交易數據的生命週期**：

- 實時數據（1 小時內）：極高訪問頻率 → S3 Standard
- 當日數據（24 小時內）：高訪問頻率 → S3 Standard
- 歷史數據（30 天內）：中等訪問頻率 → S3 IA
- 季度報告（1 年內）：低訪問頻率 → S3 Glacier
- 法規歸檔（7 年內）：極低訪問頻率 → S3 Deep Archive

**健康監測數據的生命週期**：

- 當前指標（1 天內）：用戶每日查看 → S3 Standard
- 週期趨勢（30 天內）：週期性分析 → S3 IA
- 年度記錄（1 年內）：年度體檢對比 → S3 Glacier
- 長期歷史（終生）：醫療研究價值 → S3 Deep Archive

**S3 存儲類別的成本效益分析**：

**存儲成本（每 GB/月）**：

- Standard: $0.023
- IA: $0.0125 (存儲便宜 45%，但檢索成本高)
- Glacier: $0.004 (存儲便宜 83%，但檢索延遲數小時)
- Deep Archive: $0.00099 (存儲便宜 96%，但檢索延遲 12 小時)

**訪問模式的成本臨界點**：

- 每月訪問>1 次 → Standard
- 每月訪問<1 次但>0.1 次 → IA
- 每月訪問<0.1 次但>0.01 次 → Glacier
- 每月訪問<0.01 次 → Deep Archive

## Recovery Strategy：韌性的哲學

### 故障的本體論思考

在分散式系統中，故障不是異常，而是常態。不同領域對故障的哲學態度，決定了災難恢復策略的設計。

#### 金融交易：零容忍的故障哲學

**哲學基礎**：時間就是金錢，任何延遲都是損失

- RTO 要求：< 30 秒（用戶無法忍受更長等待）
- RPO 要求：= 0（任何交易資料遺失都不可接受）

**AWS 實現策略**：

- Multi-AZ RDS：同步複製，自動故障轉移
- DynamoDB Global Tables：多區域強一致性
- Lambda@Edge：邊緣計算減少延遲
- Route 53：DNS 故障轉移，30 秒內切換

**成本影響**：基礎設施成本增加 200-300%

#### 家庭財務：實用主義的故障哲學

**哲學基礎**：生活要繼續，完美不是必需

- RTO 要求：< 4 小時（一般情況下可等待）
- RPO 要求：< 1 小時（可重新輸入遺失的記錄）

**AWS 實現策略**：

- RDS Automated Backup：每日備份，點對時間恢復
- S3 Cross-Region Replication：跨區域複製
- CloudFormation：Infrastructure as Code 快速重建
- SNS：故障通知，設定用戶期待

**成本影響**：基礎設施成本增加 30-50%

#### 健康監控：預防優於治療的故障哲學

**哲學基礎**：健康數據關乎生命，但連續性比完整性更重要

- RTO 要求：< 2 小時（影響日常監控但不致命）
- RPO 要求：< 30 分鐘（短期數據遺失可接受）

**AWS 實現策略**：

- IoT Device Shadow：設備本地快取狀態
- DynamoDB Point-in-time Recovery：自動備份
- Timestream：時序數據的自動備份與壓縮
- CloudWatch：監控告警，主動發現問題

**成本影響**：基礎設施成本增加 50-80%

### Disaster Recovery 的架構模式

#### Pilot Light 模式：最小可行備援

**設計哲學**：核心功能的備用燈
**適用場景**：RTO 要求中等，成本敏感

**實現方式**：

- 關鍵數據持續複製到備用區域
- 計算資源保持最小配置（或完全關閉）
- 故障時快速啟動並擴容

**AWS 實現**：

- RDS Cross-Region Read Replica
- S3 Cross-Region Replication
- AMI + Launch Template 預配置
- Route 53 健康檢查觸發切換

**成本特徵**：平時成本低，切換時間中等

#### Warm Standby 模式：溫備的平衡藝術

**設計哲學**：隨時準備但不浪費
**適用場景**：RTO 要求較高，成本可控

**實現方式**：

- 備用環境保持最小運行狀態
- 關鍵組件已啟動但負載較低
- 故障時快速擴容到完整容量

**AWS 實現**：

- EC2 Instance 保持運行但使用小型實例
- Auto Scaling Group min=1, max=production_size
- ELB 健康檢查自動流量切換
- Lambda 預熱 Container 避免冷啟動

**成本特徵**：持續運行成本，但切換快速

#### Multi-Site Active-Active 模式：無縫的完全冗餘

**設計哲學**：永不停機的安全感
**適用場景**：零停機要求，成本不敏感

**實現方式**：

- 多個區域同時提供完整服務
- 負載均衡分散到各個站點
- 任一站點故障，其他站點透明接管

**AWS 實現**：

- Route 53 Latency-based Routing
- CloudFront Global Distribution
- DynamoDB Global Tables
- Multi-Region VPC Peering

**成本特徵**：成本最高，但體驗最佳

## 從六個案例看非功能需求的具體化

### 案例 1：投資交易系統的完整技術規格

**性能需求**：

**API 響應時間**：

- 市場數據查詢：< 50ms (P99)
- 交易提交：< 100ms (P99)
- 持倉查詢：< 200ms (P99)

**併發要求**：

- 正常時段：500 concurrent users
- 開盤高峰：2000 concurrent users
- 系統上限：5000 concurrent users

**吞吐量**：

- 正常：1000 RPS
- 高峰：5000 RPS
- 緊急：10000 RPS (5 分鐘內)

**容量規劃**：

**用戶增長預測**：

- 當前：10000 active users
- 6 個月：25000 active users
- 12 個月：50000 active users

**數據增長**：

- 每用戶每日：100KB 交易數據
- 年增長率：500GB/year
- 5 年總容量：2.5TB + 增長緩衝 50% = 3.75TB

**計算資源**：

- CPU 密集：風險計算、技術分析
- Memory 密集：實時數據快取
- Network 密集：市場數據同步

**災難恢復**：

**關鍵指標**：

- RTO: 30 秒
- RPO: 0 秒
- 可用性: 99.99% (8.76 小時/年停機)

**實現策略**：

- Primary: us-east-1 (Multi-AZ)
- Secondary: us-west-2 (Hot Standby)
- Data: DynamoDB Global Tables + RDS Read Replica
- DNS: Route 53 Health Check + 自動故障轉移

### 案例 6：旅遊分享平台的技術規格對比

**性能需求**：

**API 響應時間（與交易系統的對比）**：

- 內容上傳：< 3 秒 (vs 交易的 100ms，要求寬鬆 300 倍)
- 社交互動：< 1 秒 (vs 交易的 50ms，要求寬鬆 20 倍)
- 內容瀏覽：< 500ms (vs 交易的 200ms，相近但容忍度高)

**併發模式**：

- 創作高峰：週末旅遊時段
- 瀏覽高峰：平日晚間分享時段
- 季節性：旅遊淡旺季差異 5-10 倍

**容量規劃哲學差異**：

**增長模式**：

- 交易系統：穩定增長，可預測容量需求
- 旅遊系統：爆發增長，病毒式傳播可能

**數據特性**：

- 交易系統：結構化數據，精確計算
- 旅遊系統：多媒體數據，存儲密集

**成本敏感度**：

- 交易系統：性能優先，成本其次
- 旅遊系統：成本敏感，性能適度

**災難恢復哲學**：

**數據價值觀**：

- 交易系統：每筆交易都是金錢，遺失不可接受
- 旅遊系統：創意內容可重新創作，遺失影響有限

**恢復策略**：

- 交易系統：Multi-AZ + 即時備份 + 零 RPO
- 旅遊系統：跨區域備份 + 定期快照 + 小時級 RPO

**用戶期待**：

- 交易系統：專業工具，完美運行是基本要求
- 旅遊系統：生活工具，偶爾故障可理解

## 明天的抽象建模預告

通過今天的技術邊界確認，我們已經從模糊的業務需求推導出了具體的技術規格。我們知道了：

- 多少 RPS：從用戶行為推導的量化指標
- 多大容量：從增長模式預測的存儲需求
- 多高可用性：從業務價值決定的故障容忍度

但這些技術規格依然是孤立的指標。明天我們將進入「抽象建模」階段，學習如何將這些技術約束轉化為系統的概念模型、行為模型、資料模型和架構模型。

**關鍵問題包括**：

- 如何用 DDD 的概念建模映射到 AWS 服務？
- 如何用 EventStorming 的行為建模設計 API 流程？
- 如何用資料建模決定 SQL vs NoSQL 的選擇？
- 如何用架構建模權衡 Serverless vs Container 的取捨？

## 今日的技術哲學總結

- 非功能需求是功能需求的量化邊界
- RPS 不是猜測，而是從用戶行為推導的必然結果
- 容量規劃是對業務增長模式的技術映射
- 災難恢復策略反映了領域對風險的哲學態度

記住：每個技術指標背後都有其業務意義。我們不是在優化技術指標，而是在優化用戶存在狀態轉化的效率和可靠性。
