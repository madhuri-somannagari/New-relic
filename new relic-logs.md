# Monitoring Success, Error & Warning Rates in Django Using New Relic

This guide walks through how to integrate **New Relic** with a **Django** application to monitor key health metrics:
- ✅ **Success Rate** – Track how many HTTP requests succeed (2xx)
- ❌ **Error Rate** – Monitor failed requests (4xx and 5xx)

It covers:
- Setting up New Relic APM  
- Log forwarding  
- Writing NRQL queries to extract meaningful insights from your application logs
---

## 📘 What is New Relic?

New Relic tracks and presents technical metrics via **APM (Application Performance Monitoring)**:
- Latency  
- Traffic  
- Errors   
---

## 🏗️ Django Application Structure

```
/Hiringdog-Backend
/core
/dashboard
/hiringdogbackend
...
/manage.py
/newrelic.ini
```

#### Requirements
- A New Relic account  
- A New Relic **license key**
- Application logs
---

##### Install Required Dependencies

```bash
pip install newrelic
```

###### Configure newrelic.ini
This file should be located at: /Hiringdog-Backend/newrelic.ini
```ini
[log]
level = info

[app]
app_name = Hiringdog-Backend
monitor_mode = true
license_key = YOUR_LICENSE_KEY
```
##### Launch Django with New Relic

``` python manage.py runserver
```

Enable Log Forwarding
New Relic does not automatically capture logs from Django unless you forward them.

``` python
import logging
from newrelic_telemetry_sdk import Harvester, Log

API_KEY = "YOUR_INSERT_API_KEY"

harvester = Harvester(api_key=API_KEY)

class NewRelicLogHandler(logging.Handler):
    def emit(self, record):
        log = Log(message=self.format(record), level=record.levelname.lower())
        harvester.record(log)

LOGGING = {
    'version': 1,
    'handlers': {
        'newrelic': {
            'class': '__main__.NewRelicLogHandler',
        },
    },
    'root': {
        'handlers': ['newrelic'],
        'level': 'INFO',
    },
}
```
⚠️ Ensure this code runs early in app startup.

###### Query Data in New Relic
Go to: ` New Relic → Query your data → Logs → NRQL `

Use these queries:

✅ Success Rate
```
SELECT percentage(count(*), WHERE status_code >= 200 AND status_code < 300) AS 'Success %' 
FROM Log 
WHERE appName = 'Hiringdog-Backend' 
SINCE 1 hour ago TIMESERIES
```
❌ Server Error Rate (5xx)
```
SELECT percentage(count(*), WHERE status_code >= 500) AS 'Error %'
FROM Log 
WHERE appName = 'Hiringdog-Backend'
SINCE 1 hour ago TIMESERIES
```
⚠️ Client Error / Warning Rate (4xx+)
```
SELECT percentage(count(*), WHERE status_code >= 400) AS 'Error %'
FROM Log 
WHERE appName = 'Hiringdog-Backend'
SINCE 1 hour ago TIMESERIES
```

#### 🧹 Clean Up
To stop monitoring temporarily:
``
unset NEW_RELIC_CONFIG_FILE
```
Or stop forwarding logs by removing or disabling the LOGGING configuration in settings.py.

