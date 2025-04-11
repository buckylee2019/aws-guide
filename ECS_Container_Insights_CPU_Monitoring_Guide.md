# Monitoring Amazon ECS Container CPU Usage with Container Insights

## Overview

Amazon ECS Container Insights provides comprehensive monitoring capabilities to help you identify and track container CPU usage. This document will guide you on how to use Container Insights to find containers with the highest CPU usage and set up appropriate monitoring and alerts.

## Enabling Container Insights

Before you start monitoring, you need to enable Container Insights on your ECS cluster:

### Via AWS Management Console

1. Sign in to the AWS Management Console
2. Navigate to the Amazon ECS service
3. Select your cluster
4. In the "Settings" tab, choose "Update Cluster Settings"
5. Check the "Container Insights" option
6. Click "Update" to save the settings

### Via AWS CLI

Run the following command:

```bash
aws ecs update-cluster-settings --cluster your-cluster-name --settings name=containerInsights,value=enabled
```

## Identifying Containers with Highest CPU Usage

### Method 1: Using CloudWatch Container Insights Dashboard

1. Sign in to the AWS Management Console
2. Navigate to the CloudWatch service
3. In the left navigation pane, choose "Insights" > "Container Insights"
4. Select "Performance monitoring"
5. Choose your ECS cluster from the dropdown menu
6. View the "Top contributors" section, which shows containers with the highest resource usage
7. Sort by CPU utilization to identify the containers with the highest CPU consumption

### Method 2: Using CloudWatch Logs Insights Queries

1. Navigate to the CloudWatch service
2. In the left navigation pane, choose "Logs" > "Logs Insights"
3. Select the `/aws/ecs/containerinsights/your-cluster-name/performance` log group from the dropdown menu
4. Enter the following query in the query editor:

```
fields @timestamp, TaskId, ContainerName, CpuUtilized, CpuReserved
| filter Type="Container"
| sort CpuUtilized desc
| limit 10
```

5. Click the "Run query" button
6. The results will show the top 10 containers with the highest CPU utilization

### Method 3: Creating a Custom Dashboard

You can create a custom dashboard to continuously monitor containers with high CPU usage:

1. Navigate to the CloudWatch service
2. In the left navigation pane, choose "Dashboards" > "Create dashboard"
3. Enter a dashboard name, such as "ECS-CPU-Monitoring"
4. Choose "Add widget" > "Line"
5. In the "Metrics" tab, select "Query"
6. Enter the following CloudWatch Metrics query:

```
SELECT MAX(CpuUtilized) 
FROM SCHEMA("ECS/ContainerInsights", ClusterName, ServiceName, TaskId, ContainerName) 
WHERE ClusterName = 'your-cluster-name'
GROUP BY ContainerName
ORDER BY MAX(CpuUtilized) DESC
LIMIT 10
```

7. Click "Create widget"
8. Save the dashboard

## Setting Up CPU Usage Alarms

### Via AWS Management Console

1. Navigate to the CloudWatch service
2. In the left navigation pane, choose "Alarms" > "All alarms"
3. Click "Create alarm"
4. Choose "Select metric"
5. Navigate to "ECS" > "ContainerInsights"
6. Select the `CpuUtilized` metric
7. Choose your cluster, service, and container name
8. Click "Select metric"
9. Set the alarm condition, such as "When CpuUtilized is greater than 80% for 5 minutes"
10. Click "Next"
11. Configure notification settings (e.g., send to an SNS topic)
12. Click "Next"
13. Enter an alarm name and description
14. Click "Create alarm"

### Via AWS CLI

Run the following command:

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name HighCpuContainerAlarm \
  --alarm-description "Alarm when container CPU exceeds 80%" \
  --metric-name CpuUtilized \
  --namespace AWS/ECS \
  --statistic Maximum \
  --period 60 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=ClusterName,Value=your-cluster-name Name=ServiceName,Value=your-service-name \
  --evaluation-periods 5 \
  --alarm-actions arn:aws:sns:region:account-id:your-sns-topic
```

## Automated Monitoring and Response

You can set up automated workflows to respond to high CPU usage events:

1. Create a Lambda function to handle high CPU usage events
2. Configure CloudWatch alarms to trigger this Lambda function
3. The Lambda function can perform actions such as scaling the service, restarting containers, or sending detailed notifications

## Common CPU Issues and Solutions

| Issue | Possible Causes | Solutions |
|-------|----------------|-----------|
| Sustained high CPU usage | Inefficient application design | Optimize code, increase resource allocation |
| CPU usage spikes | Traffic surges or memory leaks | Implement auto-scaling, check application logs |
| CPU usage near limits | Insufficient resource allocation | Increase CPU limits in task definition |
| Specific container with high CPU | Possible infinite loops or compute-intensive tasks | Check application code, optimize algorithms |

## Best Practices

1. **Set appropriate CPU thresholds**: Base alarm thresholds on historical usage patterns
2. **Regularly review CPU metrics**: Use Container Insights dashboards to periodically check CPU trends
3. **Combine with log analysis**: Correlate CPU metrics with application logs to identify root causes
4. **Implement auto-scaling**: Configure auto-scaling strategies for ECS services based on CPU metrics
5. **Optimize container resource allocation**: Ensure CPU allocations in task definitions match actual needs

## Related AWS Documentation Resources

- [Setting up Container Insights for Amazon ECS](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/deploy-container-insights-ECS-cluster.html)
- [Viewing Container Insights Metrics for Amazon ECS](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-view-metrics.html)
- [Using CloudWatch Logs Insights to View Container Insights Data](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-analyze-ECS.html)
- [Available Metrics for Container Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-metrics-ECS.html)

## Conclusion

With Amazon ECS Container Insights, you can comprehensively monitor container CPU usage, identify performance bottlenecks, and take timely action. The methods and tools provided in this document can help you effectively identify and manage containers with high CPU usage, ensuring stable operation and optimal performance of your applications.
