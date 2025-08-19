# Monitoring Success & Error Rates in Django Using New Relic

### New Relic
New Relic tracks and presents technical metrics via APM (Application Performance Monitoring): Latency  Traffic  Errors
- APM → Transactions: See slow endpoints and response times.
- APM → Errors: Track 4xx, 5xx, and exceptions.
- APM → Databases: See slow queries.
- Service maps: Visualize connections to DB, cache, other services.

### APM (Application Performance Monitoring)
***Tracks:***  Application code execution, Transactions, Database calls, Response times, Errors, Throughput, Traces  
***Best for:*** Finding performance bottlenecks in your backend code.
**Data Source:** Instrumented agent in your app (`newrelic.ini`, `newrelic-admin`).

### Transactions
A transaction is a timed event that starts when your app begins processing a request (HTTP, background job, message queue, etc.) and ends when it’s finished.  
It represents one logical unit of work in the application with end-to-end visibility.
It measures:
- ****Duration**** → How long it took
- ****Activity**** → What happened inside (DB calls, external API calls, etc.)
- ****Outcome**** → Success or failure

Transactions are the foundation of APM because they tell you:
- ****Performance:**** How fast each endpoint or background job runs.
- ****Errors:**** Which transactions fail most often and why.
- ****Throughput:**** How many requests your app handles over time.
- ****Bottlenecks:**** Which parts of the code or dependencies slow things down.

#### Types of Transactions
***Web transactions:***  
- HTTP request/response cycles  
- Tracked automatically by Python agent  
- Example: `/api/login/`, `/api/signup/`

***Background transactions:***  
- Non-web processes  
- Must be manually instrumented  
- Example: `Celery task`, `background cron job`, `queue worker`

### Logs  
- Collect raw application logs (Django logs, HTTP access logs, warnings, tracebacks) into New Relic  
- Troubleshoot issues and correlate logs with APM traces
****Source:**** Log forwarder or log integration in APM

### Configure Application Identification
****Make data easy to find and understand in New Relic****
   - Set a unique `app_name` in `newrelic.ini`  
   - Use consistent naming across environments (e.g., `MyApp-Prod`, `MyApp-Staging`)
   - Set correct environment labels (tags/labels) for dashboard filtering

### Enable Relevant Features
Capture the right data to calculate success rate:
- Enable distributed tracing  
  ```ini
  distributed_tracing.enabled = true
  ```
- Enable error collection
```ini
 error_collector.enabled = true
```
- Enable transaction tracing
```ini
transaction_tracer.enabled = true
```

#### Django Application Structure
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
- Application

##### Install Required Dependencies
In application python environment install new relic
```bash
pip install newrelic
```
###### Configure newrelic.ini
This file should be located at: /Hiringdog-Backend/newrelic.ini
```newrelic.ini
[newrelic]
license_key = <new_relic_licensekey>
app_name = <application_name>
monitor_mode = true
# Logging configuration
log_file = stdout
log_level = info

ssl = true
high_security = false

transaction_tracer.enabled = true
transaction_tracer.transaction_threshold = apdex_f
transaction_tracer.record_sql = obfuscated
transaction_tracer.stack_trace_threshold = 0.5
transaction_tracer.explain_enabled = true
transaction_tracer.explain_threshold = 0.5

# Error collector
error_collector.enabled = true
error_collector.record_errors = true
error_collector.ignore_status_codes =


# Browser monitoring
browser_monitoring.auto_instrument = true

# Thread profiler
thread_profiler.enabled = true

# Distributed tracing
distributed_tracing.enabled = true
span_events.enabled = true

# Enable logging
application_logging.enabled = true
application_logging.forwarding.enabled = true
application_logging.metrics.enabled = true
application_logging.local_decorating.enabled = true
# infinite_tracing.trace_observer.host = YOUR_TRACE_OBSERVER_HOST

# Custom insights events
custom_insights_events.enabled = true
attributes.include = httpResponseCode


# Environment-specific settings
[newrelic:development]
monitor_mode = true

[newrelic:test]
monitor_mode = false

[newrelic:staging]
app_name = <application_name> (Staging)
monitor_mode = true

[newrelic:production]
monitor_mode = true
```
##### Launch Django with New Relic
In base.py/setting.py config the new relic
```
# New Relic Configuration
NEW_RELIC_CONFIG_FILE = os.path.join(BASE_DIR, 'newrelic.ini')
NEW_RELIC_LICENSE_KEY = os.getenv('NEW_RELIC_LICENSE_KEY')
NEW_RELIC_APP_NAME = os.getenv('NEW_RELIC_APP_NAME')

# Force New Relic initialization with correct settings
try:
    import newrelic.agent
    
    # Set environment variables
    os.environ['NEW_RELIC_CONFIG_FILE'] = NEW_RELIC_CONFIG_FILE
    os.environ['NEW_RELIC_LICENSE_KEY'] = NEW_RELIC_LICENSE_KEY
    os.environ['NEW_RELIC_APP_NAME'] = 'Hiringdog-Backend'
    
    # Initialize New Relic
    newrelic.agent.initialize(NEW_RELIC_CONFIG_FILE)
    
    
    print("✓ New Relic initialized successfully")
    print(f"✓ Application name: Hiringdog-Backend")
    
except ImportError:
    print("⚠ New Relic not installed - monitoring disabled")
except Exception as e:
    print(f"⚠ New Relic initialization failed: {e}")
```

restart/run the application after configuration
```
python manage.py runserver
```

###### Query Data in New Relic
Go to: ` New Relic → Query your data → Logs → NRQL `

Use these queries:


#### Clean Up
To stop monitoring temporarily:
```
unset NEW_RELIC_CONFIG_FILE
```
Or stop forwarding logs by removing or disabling the LOGGING configuration in settings.py.

#### Validate Data in New Relic
Check APM dashboard for application. Review key metrics such as `Throughput` `Error rate` `Response time`. Based on these, create custom dashboards as needed to highlight the most relevant insights.









