# 使用 Container Insights 監控 Amazon ECS 容器 CPU 使用率指南

## 概述

Amazon ECS Container Insights 提供了全面的監控功能，可以幫助您識別和追蹤容器的 CPU 使用情況。本文檔將指導您如何使用 Container Insights 找出 CPU 使用率最高的容器，並設置相應的監控和警報。

## 啟用 Container Insights

在開始監控之前，您需要在 ECS 叢集上啟用 Container Insights：

### 透過 AWS 管理控制台啟用

1. 登入 AWS 管理控制台
2. 導航至 Amazon ECS 服務
3. 選擇您的叢集
4. 在「設定」標籤中，選擇「更新叢集設定」
5. 勾選「Container Insights」選項
6. 點擊「更新」保存設定

### 透過 AWS CLI 啟用

執行以下命令：

```bash
aws ecs update-cluster-settings --cluster 您的叢集名稱 --settings name=containerInsights,value=enabled
```

## 識別 CPU 使用率最高的容器

### 方法一：使用 CloudWatch Container Insights 儀表板

1. 登入 AWS 管理控制台
2. 導航至 CloudWatch 服務
3. 在左側導航欄中，選擇「Insights」>「Container Insights」
4. 選擇「效能監控」
5. 從下拉菜單中選擇您的 ECS 叢集
6. 查看「主要貢獻者」部分，該部分顯示資源使用率最高的容器
7. 按 CPU 使用率排序，以識別 CPU 消耗最高的容器

### 方法二：使用 CloudWatch Logs Insights 查詢

1. 導航至 CloudWatch 服務
2. 在左側導航欄中，選擇「Logs」>「Logs Insights」
3. 從下拉菜單中選擇 `/aws/ecs/containerinsights/您的叢集名稱/performance` 日誌組
4. 在查詢編輯器中輸入以下查詢：

```
fields @timestamp, TaskId, ContainerName, CpuUtilized, CpuReserved
| filter Type="Container"
| sort CpuUtilized desc
| limit 10
```

5. 點擊「執行查詢」按鈕
6. 結果將顯示 CPU 使用率最高的前 10 個容器

### 方法三：創建自定義儀表板

您可以創建一個自定義儀表板來持續監控高 CPU 使用率的容器：

1. 導航至 CloudWatch 服務
2. 在左側導航欄中，選擇「儀表板」>「創建儀表板」
3. 輸入儀表板名稱，例如「ECS-CPU-監控」
4. 選擇「添加小部件」>「線圖」
5. 在「指標」標籤中，選擇「查詢」
6. 輸入以下 CloudWatch Metrics 查詢：

```
SELECT MAX(CpuUtilized) 
FROM SCHEMA("ECS/ContainerInsights", ClusterName, ServiceName, TaskId, ContainerName) 
WHERE ClusterName = '您的叢集名稱'
GROUP BY ContainerName
ORDER BY MAX(CpuUtilized) DESC
LIMIT 10
```

7. 點擊「創建小部件」
8. 保存儀表板

## 設置 CPU 使用率警報

### 透過 AWS 管理控制台設置警報

1. 導航至 CloudWatch 服務
2. 在左側導航欄中，選擇「警報」>「所有警報」
3. 點擊「創建警報」
4. 選擇「選擇指標」
5. 導航至「ECS」>「ContainerInsights」
6. 選擇 `CpuUtilized` 指標
7. 選擇您的叢集、服務和容器名稱
8. 點擊「選擇指標」
9. 設置警報條件，例如「當 CpuUtilized 大於 80% 持續 5 分鐘時」
10. 點擊「下一步」
11. 配置通知設置（例如，發送到 SNS 主題）
12. 點擊「下一步」
13. 輸入警報名稱和描述
14. 點擊「創建警報」

### 透過 AWS CLI 設置警報

執行以下命令：

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name 高CPU容器警報 \
  --alarm-description "當容器 CPU 超過 80% 時發出警報" \
  --metric-name CpuUtilized \
  --namespace AWS/ECS \
  --statistic Maximum \
  --period 60 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=ClusterName,Value=您的叢集名稱 Name=ServiceName,Value=您的服務名稱 \
  --evaluation-periods 5 \
  --alarm-actions arn:aws:sns:區域:帳戶ID:您的SNS主題
```

## 自動化監控與回應

您可以設置自動化工作流程來回應高 CPU 使用率事件：

1. 創建一個 Lambda 函數來處理高 CPU 使用率事件
2. 將 CloudWatch 警報配置為觸發此 Lambda 函數
3. Lambda 函數可以執行諸如擴展服務、重啟容器或發送詳細通知等操作

## 常見 CPU 問題及解決方案

| 問題 | 可能原因 | 解決方案 |
|------|---------|---------|
| 持續高 CPU 使用率 | 應用程序設計效率低下 | 優化代碼、增加資源分配 |
| CPU 使用率突然飆升 | 流量突增或內存洩漏 | 實施自動擴展、檢查應用程序日誌 |
| CPU 使用率接近限制 | 資源配置不足 | 增加任務定義中的 CPU 限制 |
| 特定容器持續高 CPU | 可能存在無限循環或計算密集型任務 | 檢查應用程序代碼、優化算法 |

## 最佳實踐

1. **設置適當的 CPU 閾值**：基於歷史使用模式設置警報閾值
2. **定期審查 CPU 指標**：使用 Container Insights 儀表板定期檢查 CPU 趨勢
3. **結合日誌分析**：將 CPU 指標與應用程序日誌相關聯，以識別問題根源
4. **實施自動擴展**：基於 CPU 指標配置 ECS 服務的自動擴展策略
5. **優化容器資源分配**：確保任務定義中的 CPU 分配與實際需求相匹配

## 相關 AWS 文檔資源

- [為 Amazon ECS 設置 Container Insights](https://docs.aws.amazon.com/zh_tw/AmazonCloudWatch/latest/monitoring/deploy-container-insights-ECS-cluster.html)
- [查看 Amazon ECS 的 Container Insights 指標](https://docs.aws.amazon.com/zh_tw/AmazonCloudWatch/latest/monitoring/Container-Insights-view-metrics.html)
- [使用 CloudWatch Logs Insights 查看 Container Insights 數據](https://docs.aws.amazon.com/zh_tw/AmazonCloudWatch/latest/monitoring/Container-Insights-analyze-ECS.html)
- [Container Insights 可用指標](https://docs.aws.amazon.com/zh_tw/AmazonCloudWatch/latest/monitoring/Container-Insights-metrics-ECS.html)

## 結論

透過 Amazon ECS Container Insights，您可以全面監控容器的 CPU 使用情況，識別性能瓶頸，並及時採取行動。本文檔提供的方法和工具可以幫助您有效地識別和管理高 CPU 使用率的容器，確保應用程序的穩定運行和最佳性能。
