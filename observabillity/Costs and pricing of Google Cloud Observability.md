

Google Cloud Observability services are fully managed, meaning their cost is usage-based rather than infrastructure-based. This ensures you only pay for the data you ingest and the operations you perform.


#### Cloud Logging
**Based on**:

- Total volume of chargeable logs ingested into the platform.
    

**Free allotment/month**:

- Each project receives its first 50 GiB at no cost.
    
- Logs retained for the default retention period.
    

**Examples of usage that incurs cost**:

- Cloud Load Balancing logs.
    
- Custom logs.
    
- Error Reporting data ingested via Logging.
    
- The write operation in the Cloud Logging API.
    
- Logs stored beyond 30 days incur a retention charge for non-required buckets.



#### Cloud Monitoring
**Based on**:

- Volume of chargeable metrics ingested.
    
- Number of chargeable API calls.
    
- Execution of Cloud Monitoring uptime checks.
    
- Metrics ingested by using Google Cloud Managed Service for Prometheus.
    

**Free allotment/month**: 

- All [non-chargeable Google Cloud metrics.(opens in a new tab)](https://cloud.google.com/stackdriver/pricing#metrics-non-chargeable)
    
- First 150 MiB per billing account.
    
- First 1 million Read API calls per billing account.
    
- 1 million uptime check executions per project.
    

**Examples of usage that incurs cost**:

- Cloud Monitoring custom metrics.
    
- External metrics written to Cloud Monitoring through API or client libraries.
    
- The read operation in the Monitoring API (except from the Google Cloud console).


#### Cloud Trace
**Based on**:

- Number of spans ingested and eventually scanned.
    

**Free allotment/month**:

- First 2.5 million spans.
    

**Examples of usage that incurs cost**:

- Spans for App Engine apps outside of the default spans.
    
- Cloud Load Balancing.
    
- Custom apps.


## Network telemetry costs 

The networking logs, including VPC Flow logs, Firewall Rules Logging, and Cloud NAT, will cost you the standard log storage fees. However, if you store them in Cloud Logging, they won't cost you anything extra to generate.

If you export the network telemetry logs to an external service, cost is incurred to generate logs. The cost is in addition to any destination or networking fees.

![](https://storage.googleapis.com/cloud-training/cls-html5-courses/T-STACKD2-B/v1.2/EN/content/assets/Costs2.png)

Network Intelligence Center incurs costs for metrics overlaid on the network topology, Network Analyzer and performance dashboard.

Network Intelligence Center also incurs a cost for running connectivity tests and Firewall insights. Refer to the [documentation(opens in a new tab)](https://cloud.google.com/network-intelligence-center/pricing) for more information on pricing models for Firewall insights.


## Free functions

Many functions of Google Cloud Observability are free, including:

- •
    
    Using Cloud Profiler.
    
- •
    
    Collecting and using the Cloud Audit Logs, Access Transparency logs, BigQuery Data Access logs and anything excluded from logs.
    
- •
    
    Creating and using dashboards.
    
- •
    
    Visualizing Google Cloud and Anthos metrics and log streams.
    
- •
    
    App Engine standard trace spans.
    
- •
    
    Uptime checks.
    
- •
    
    Logs analytics when queries are running in Cloud Logging.