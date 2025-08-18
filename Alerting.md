# New Relic Alert Setup

If your application’s URLs (endpoints) like /login, /api/data, /checkout start taking too long to respond, it usually means something is wrong:
- Slow database queries
- Overloaded server / CPU spike
- Network latency
- Code bottlenecks
- Third-party API delays

***Fast response time = good user experience.***

***Slow response time = bad user experience → alert immediately.***

### Query for latency monitoring

We’ll monitor average response time per URL (faceted by request.uri):
```
FROM Transaction
SELECT average(duration)
WHERE appName = 'Hiringdog-Backend'
  AND transactionType = 'Web'
FACET request.uri
```

Optional:
```
FROM Transaction
SELECT percentile(duration, 95)
WHERE appName = 'Hiringdog-Backend'
  AND transactionType = 'Web'
FACET request.uri
```
## Create an Alert Condition

Go to:` Alerts & AI → Alert conditions → New condition`

-- Name your alert condition: `Latency Alert`

-- Policy name: ` Hiringdog-policy `

-- Group incidents into issues: ` ✅ One issue per condition and signal (ensures each URL gets its own alert)`

-- Close open incidents after: ` 1 hour `(recommended, not too long)

### Configure Signal & Thresholds

***Data aggregation:***

Window duration: ` 1 minute ` 

Sliding window aggregation: ` ON `

Streaming method: ` Event timer `

Evaluation delay: `2 minutes (to wait for late events)`

Gap filling: ` None `

Evaluate per facet: ` ✅ ON `

***Static thresholds:***

***Critical:*** `average(duration) > 120s for at least 5 minutes`

***Warning:*** `average(duration) > 60s for at least 3 minutes`

#### Customize Incident Messages

Title template: `[Latency] {{conditionName}} – {{facet}} is {{latestValue}}s (> {{threshold}}s) `

Description template:

```
The endpoint {{facet}} in {{appName}} is experiencing high latency.
Current value: {{latestValue}}s
Threshold: {{threshold}}s
```
Runbook URL: ` https:// `

Enable on save: ✅

### Create Alert Policy & Slack Workflow

***Create a policy***
Go to: ` Alerts & AI → Policies → New policy `

Name:  `Hiringdog`

***Connect Slack***
Go to: ` Alerts & AI → Destinations → New → Slack `

Add your Slack workspace or webhook

Name destination: ` Slack ` 
select your workspace and allow the new relic to connect.
create a channel in slack: ` #alerts-backend `

### Create Workflow

Go to: `Alerts → Workflows → New`
Trigger: Issues from policy = `Hiringdog-Backend`
Destination:  `Slack`, `#alerts-backend`


