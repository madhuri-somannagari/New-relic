# New Relic Dashboard
The custom Dashboard Creation of Monitoring the application

### Log-Based Metrics
#### Request Success & Failure (Aggregated)

This query calculates the total requests along with success and failure percentages:
```NRQL
FROM Log 
SELECT 
  count(*) AS 'Total Requests',
  percentage(count(*), WHERE message LIKE '%"is_success": true%') AS 'Success',
  percentage(count(*), WHERE message LIKE '%"is_success": false%') AS 'Failure'
WHERE message LIKE '%HTTP Request%'
  AND entity.name = 'Hiringdog-Backend'
SINCE 1 week ago
```
Insight: Provides an overall success vs. failure rate for HTTP requests in the past week.

#### Request Success & Failure (Time Series)

This query shows request outcomes over time for trend analysis:
```NRQL
FROM Log 
SELECT 
  count(*) AS 'Total Requests',
  filter(count(*), WHERE message LIKE '%"is_success": true%') AS 'Success',
  filter(count(*), WHERE message LIKE '%"is_success": false%') AS 'Failure'
WHERE message LIKE '%HTTP Request%'
  AND entity.name = 'Hiringdog-Backend'
SINCE 1 week ago TIMESERIES
```
Insight: Displays success/failure trends over time, useful to detect sudden spikes in errors.

### Transaction-Level Metrics
#### Transactions by Status Code

This query tracks HTTP status code categories (2xx, 4xx, 5xx) per URI:

```NRQL
FROM Transaction
SELECT
  count(*) AS 'Total Transactions',
  percentage(count(*), WHERE response.status LIKE '2%') AS '2xx Success %',
  percentage(count(*), WHERE response.status LIKE '4%') AS '4xx Client Error %',
  percentage(count(*), WHERE response.status LIKE '5%') AS '5xx Server Error %'
FACET request.uri
WHERE appName = 'Hiringdog-Backend' 
SINCE 1 week ago
```
Insight: Breaks down success vs. error percentages per endpoint, helping identify problem APIs.

#### Service Health Dashboard

This query evaluates response times, errors, and system health with thresholds and alert indicators:
```NRQL
WITH
  1 as warnDurationThresh,
  4 as critDurationThresh,
  3 as warnErrPctThresh,
  10 as critErrPctThresh
SELECT
  average(duration)*1000 AS 'Response Time (ms)',
  if(
    average(duration) >= 0 and average(duration) < warnDurationThresh 
    and percentage(count(*), WHERE error is True) < warnErrPctThresh, '🟢',
      if(
        (average(duration) >= warnDurationThresh and average(duration) < critDurationThresh) 
        OR (percentage(count(*), WHERE error is True) >= warnErrPctThresh 
        and percentage(count(*), WHERE error is True) < critErrPctThresh), '🟡',
        '🔴' )
  ) AS 'Health',
  max(duration)*1000 as 'Max Response Time (ms)',
  count(*) as 'Calls',
  rate(count(*),1 minute) as 'Calls / min',
  percentage(count(*), WHERE error is true) as '% Errors',
  filter(count(*), WHERE error is true) as 'Total Errors'
FROM Transaction
WHERE
  transactionType = 'Web'
  AND request.uri NOT RLIKE r'/.*.(ico|css)'
FACET appName, request.uri AS 'Business Transaction'
SINCE 1 week ago
```
Insight:

🟢 = Healthy (low response time, low error %)

🟡 = Warning (slightly high latency/errors)

🔴 = Critical (high latency or high error rate)

This acts as a traffic-light health indicator for backend transactions.
