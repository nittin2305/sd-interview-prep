# Alerting and Monitoring

> **References:** [Google SRE Book - Monitoring](https://sre.google/sre-book/monitoring-distributed-systems/) | [PagerDuty Best Practices](https://www.pagerduty.com/resources/learn/alerting-best-practices/) | [AWS CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)

---

## SLO-Based Alerting

Alert on **symptoms** (user impact), not **causes** (individual metrics).

```
❌ Bad alert: CPU > 80% on order-service-i-1234
   (Cause-based, may not impact users, too noisy)

✅ Good alert: Order success rate < 99% over last 5 minutes
   (Symptom-based, directly measures user impact)

✅ Good alert: p99 order processing latency > 2 seconds
   (Symptom-based, user-visible degradation)
```

---

## Four Golden Signals (Google SRE)

| Signal | What to measure | Alert on |
|--------|----------------|---------|
| **Latency** | Response time (p50, p99) | p99 > SLO target |
| **Traffic** | RPS, QPS | Sudden drop (indicates total failure) |
| **Errors** | Error rate (4xx, 5xx) | Error rate > 0.1% sustained |
| **Saturation** | CPU, memory, queue depth | Resource > 80% |

---

## USE Method (for Resources)

For each system resource (CPU, memory, disk, network):
- **U**tilization: How busy is the resource? (%)
- **S**aturation: How much extra work is queued? (queue depth)
- **E**rrors: Error count

---

## Alert Severity Levels

| Severity | Response Time | Example | Escalation |
|---------|--------------|---------|------------|
| **P1 Critical** | Immediate (24/7) | 100% error rate, data loss | Wake on-call now |
| **P2 High** | < 30 min | Error rate > 5%, latency 5× SLO | Page on-call |
| **P3 Medium** | Business hours | Error rate > 1%, approaching capacity | Slack notification |
| **P4 Low** | Next sprint | Non-critical degradation | Ticket |

---

## AWS CloudWatch Alarms

```java
// Using AWS CDK to define alarms
public class MonitoringStack extends Stack {

    public MonitoringStack(Construct scope, String id) {
        super(scope, id);
        
        String serviceName = "order-service";
        Topic alertTopic = new Topic(this, "AlertTopic");
        
        // P2: High error rate
        Metric errorRate = new Metric(MetricProps.builder()
            .namespace("OrderService")
            .metricName("ErrorRate")
            .statistic("Average")
            .period(Duration.minutes(5))
            .build());
        
        new Alarm(this, "HighErrorRate", AlarmProps.builder()
            .alarmName(serviceName + "-HighErrorRate")
            .alarmDescription("Order service error rate > 1% for 5 minutes")
            .metric(errorRate)
            .threshold(1.0)           // 1%
            .comparisonOperator(ComparisonOperator.GREATER_THAN_THRESHOLD)
            .evaluationPeriods(3)     // 3 consecutive 5-min periods
            .datapointsToAlarm(2)     // 2 out of 3 must breach
            .treatMissingData(TreatMissingData.NOT_BREACHING)
            .build())
            .addAlarmAction(new SnsAction(alertTopic));
        
        // P1: Complete outage
        Metric successRate = new Metric(MetricProps.builder()
            .namespace("OrderService")
            .metricName("SuccessRate")
            .statistic("Average")
            .period(Duration.minutes(1))
            .build());
        
        new Alarm(this, "ServiceOutage", AlarmProps.builder()
            .alarmName(serviceName + "-ServiceOutage")
            .alarmDescription("Order service completely unavailable")
            .metric(successRate)
            .threshold(95.0)          // < 95% success rate
            .comparisonOperator(ComparisonOperator.LESS_THAN_THRESHOLD)
            .evaluationPeriods(2)
            .datapointsToAlarm(2)
            .build())
            .addAlarmAction(new SnsAction(alertTopic));
        
        // P3: Latency degradation
        Metric p99Latency = new Metric(MetricProps.builder()
            .namespace("OrderService")
            .metricName("ProcessingDuration")
            .statistic("p99")
            .period(Duration.minutes(5))
            .build());
        
        new Alarm(this, "HighLatency", AlarmProps.builder()
            .alarmName(serviceName + "-HighLatency")
            .alarmDescription("p99 order processing > 3 seconds")
            .metric(p99Latency)
            .threshold(3000)          // 3 seconds in ms
            .comparisonOperator(ComparisonOperator.GREATER_THAN_THRESHOLD)
            .evaluationPeriods(3)
            .datapointsToAlarm(2)
            .build())
            .addAlarmAction(new SnsAction(alertTopic));
    }
}
```

---

## Runbook Template

```markdown
# Runbook: OrderService-HighErrorRate

## Alert
- Alert name: order-service-HighErrorRate
- Severity: P2
- Threshold: Error rate > 1% for 10 minutes

## Impact
- Users cannot place orders
- Revenue impact: ~$50K/minute at peak

## Diagnosis Steps
1. Check CloudWatch dashboard: https://...
2. Check error logs in CloudWatch Logs Insights:
   ```
   fields @timestamp, @message
   | filter level = "ERROR" and service = "order-service"
   | stats count() by errorCode
   | sort count() desc
   ```
3. Check downstream dependencies:
   - Payment service health: https://...
   - DynamoDB metrics: https://...
4. Check recent deployments: https://...
5. Check X-Ray trace samples for errors: https://...

## Remediation
- If payment service errors: enable circuit breaker (set PAYMENT_CB_ENABLED=true)
- If DynamoDB throttling: increase capacity or enable on-demand
- If OOM errors: increase ECS task memory (see Terraform vars)
- If deployment-related: rollback via CodeDeploy: `aws deploy stop-deployment --deployment-id d-XXXXXX`

## Escalation
- 30 minutes no resolution → escalate to P1
- Contact: on-call → senior engineer → engineering manager
```

---

## Composite Alarms

```java
// Alert only when BOTH error rate AND traffic indicate real failure
// (prevents false alarms when traffic is near zero)

Alarm errorAlarm = ...; // Error rate > 1%
Alarm trafficAlarm = ...; // RPS > 10 (real traffic)

new CompositeAlarm(this, "RealFailure", CompositeAlarmProps.builder()
    .alarmDescription("Real failure: high errors with real traffic")
    .alarmRule(AlarmRule.allOf(errorAlarm, trafficAlarm))
    .build());
```

---

## Interview Q&A

**Q1: What is alert fatigue and how do you combat it?**
> Alert fatigue: too many alerts cause engineers to ignore or silence them, missing real incidents. Combat: (1) Only alert on user-visible symptoms (error rate, latency) not causes (CPU). (2) Set thresholds that require sustained breach (3 consecutive periods). (3) Use composite alarms to reduce false positives. (4) Regular alert reviews — delete alerts with < 5% action rate. (5) Separate paging (P1/P2) from Slack notifications (P3/P4).

**Q2: How do you set the right alert threshold?**
> Look at historical data: what does "normal" look like? Set threshold at 3-5 standard deviations from normal, or based on SLO: if SLO is 99.9% success rate, alert at 99% (10% of error budget spent in 5 min). Use CloudWatch anomaly detection which automatically learns baseline and alerts on deviations. Test by simulating failures and verifying alerts trigger appropriately.

**Q3: What is the difference between monitoring and observability?**
> Monitoring: predefined dashboards and alerts for known failure modes. "Is my service up?" Observability: the ability to understand system behavior from outputs alone, including novel failure modes you didn't anticipate. "Why is my service slow in this specific edge case?" Monitoring is a subset of observability. You achieve observability through: high-cardinality logs, distributed tracing, rich metrics.
