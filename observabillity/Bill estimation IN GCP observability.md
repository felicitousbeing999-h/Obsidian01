
When you try to estimate prices in Google Cloud, the page of choice should always be the Google Cloud Pricing Calculator. The Pricing Calculator is accurate, but it's only as accurate as the data that you provide it.

![Google Cloud pricing calculator.](https://storage.googleapis.com/cloud-training/cls-html5-courses/T-STACKD2-B/v1.2/EN/content/assets/Calculator%20hd.jpg)

If your operation services are already running on Google Cloud, start by pulling the prices of what you're spending on them. You can look for the requisite data in several places.

For more documentation on the pricing calculator, refer to the [documentation(opens in a new tab)](https://cloud.google.com/products/calculator?hl=en).



You can also use the Cost Estimation API. It provides customer-specific estimates that include your discounts. For example, those negotiated as part of a contract and those based on committed usage. These cost estimates can help you make more informed business decisions.

Review a quick tutorial on how to use the pricing calculator.



## View and monitor metrics


## View monitoring usage by metrics scope

A metrics scope is used in Cloud Monitoring to monitor the resources you care about. The resources could be in a Google Cloud project, an AWS account, or multiple Google Cloud projects and AWS accounts

To view this usage, navigate to **Monitoring**, **Settings** and select the **Summary** tab.

The Metrics Ingested table displays a summary of your metrics ingestion data by resource. This data includes the previous month total usage, the current month to-date usage, and projected usage for the current month.

To get your project-level usage in detail, in the Metrics Ingested table, click **View Bill**., which takes you to the Cloud Billing Reports page.

![](https://storage.googleapis.com/cloud-training/cls-html5-courses/T-STACKD2-B/v1.2/EN/content/assets/Metrics%20scope%20hd.jpg)

## View metrics ingestion in Metrics Diagnostics

After you know the projects where you’re spending on Cloud Monitoring, we want it to be easy to understand which metrics are driving these observability costs. We also want to provide insights on how to reduce spend on unused and noisy metrics.

To get started, go to **Monitoring**, **Metrics Diagnostics.** This page provides many tools to understand metric ingestion and Monitoring API usage. One tool is a “Metrics” table where you can sort and filter metrics by:

- volume or samples ingested,
    
- metric cardinality,
    
- metric name and domain,
    
- metric labels,
    
- project,
    
- error rate, and more.
    

It is recommend sorting metrics by **Metric Data Ingested** in descending order to identify exactly which metrics are primarily driving ingestion volume. Whether intended or not, our customers often find that only a few metrics or metric types drive most consumption. These are the ones that are ripe for cost reduction and optimization.

![](https://storage.googleapis.com/cloud-training/cls-html5-courses/T-STACKD2-B/v1.2/EN/content/assets/Metrics%20diagnosis%20hd.jpg)

## View Logs-Based Metrics usage

To check your logs-based metrics usage, navigate to **Logging**, **Logs-Based Metrics**.

This view provides two key snapshots:

- **Previous Month Usage** represents the sum of bytes ingested in the logs-based metric in the previous calendar month.
    
- **Usage (MTD)** represents the sum of bytes ingested in the logs-based metric in the current calendar month.
    

Clicking any of the column names lets you sort data in ascending or descending order. For example, if you want to review which metrics ingest the most data, sorting data is helpful.

![](https://storage.googleapis.com/cloud-training/cls-html5-courses/T-STACKD2-B/v1.2/EN/content/assets/Logs%20based%20metrics%20h.jpg)