

# Error Reporting

<div align="center">


	  <img src="Pasted image 20260807004606.png"> 
</div>




Error Reporting looks through all the logs that your application and infrastructure has reported. It then counts, analyzes, and aggregates the exceptions to report them on your preferred notification channel such as email, mobile app, slack, or through web hooks.


Error Reporting can only analyze log entries that:

- Are stored in Cloud Logging buckets in the global region.
    
- Have the same source and destination Google Cloud projects.
    
	- Have the customer-managed encryption keys (CMEK)  disabled that is you need to make the google cloud responsible for managing keys


><b> If you route logs to a different Cloud project, regionalized buckets, or enable CMEK, then Error Reporting doesn't capture and analyze those logs.</b>



## Features in Error Reporting



### Understand errors

It helps understand errors. Review at a glance the top or new errors for your application in a clear dashboard. Reviewing a log stream to find important errors can slow you down when you’re troubleshooting. Error Reporting brings you the processed data directly to help you understand and fix the root causes faster.

### Intelligent aggregation

Real production problems can be hidden in mountains of data. Error Reporting helps you see the problems through the noise by constantly analyzing your exceptions. Problems are intelligently aggregated into meaningful groups tailored to your programming language and framework.

###  Instant error notification

You do not wait for your users to report problems. Error Reporting is always watching your service and instantly alerts you when a new application error cannot be grouped with existing ones. Directly navigate from a notification to the details of the new error. Error Reporting is available on desktop and in the Google Cloud app for iOS and Android.

### Broad language and product support

Support is available for many popular languages, including [Go(opens in a new tab)](https://docs.cloud.google.com/error-reporting/docs/setup/go), [Java(opens in a new tab)](https://docs.cloud.google.com/error-reporting/docs/setup/go), [Node.js(opens in a new tab)](https://docs.cloud.google.com/error-reporting/docs/setup/nodejs), [PHP(opens in a new tab)](https://docs.cloud.google.com/error-reporting/docs/setup/php), [Python(opens in a new tab)](https://docs.cloud.google.com/error-reporting/docs/setup/python), [Ruby(opens in a new tab)](https://cloud.google.com/error-reporting/docs/setup/ruby), and [.NET(opens in a new tab)](https://cloud.google.com/error-reporting/docs/setup/dotnet). Use our client libraries, REST API, or send errors with Cloud Logging.

Error Reporting can aggregate and display errors for a wide range of compute environments (supported Cloud products), including:

- App Engine (Standard and Flexible environments)
    
- Cloud Run and Cloud Run functions
    
- Google Kubernetes Engine (GKE)
    
- Compute Engine
    
- Apps Script
    
- Amazon EC2








# Setup Process

## Step 1:

To report errors, the code needs the Error Reporting Writer Identity and Access Management (IAM) role. Enable the Error Reporting API, and install the client library by using npm.

```shell 
npm install --save @google-cloud/error-reporting_
```



## Step 2:

![[Pasted image 20260807005119.png]]


The easiest way to manually log errors to Error Reporting in Node.js is to import the Error Reporting library.

- You then instantiate a client to start reporting errors to Error Reporting. Optionally, you can also customize the behavior of the Error Reporting library for Node.js. These can be configured by passing objects to options.
    
- Use error message builder to customize all fields.
    
- Call the report method to manually report an error.
    

You can also integrate the Error Reporting library for Node.js to web frameworks like Express.js.




## Step 3:


Configure the environment

Error Reporting Library for Node.js can be configured on many Google Cloud environments. Let’s explore the process for all.

- For App Engine flexible environment and standard environment, Cloud Run, Cloud Run functions, and Apps Script, Error Reporting is automatically enabled.
    
- For Google Kubernetes Engine, add cloud-platform access scope during cluster creation.
    
- For Compute Engine, ensure the service account used has the Error Reporting Writer role.
    
- Outside Google Cloud, provide the Google Cloud project ID and service account credentials to the Error Reporting library for Node.js.




# Cloud Trace


<div align="center">
<img src="Pasted image 20260807010506.png" >
</div>

Cloud Trace is a distributed tracing system that collects latency data from your applications and displays it in the Google Cloud console. You can track how requests propagate through your application and receive detailed near-real time performance insights.

## 

Core capabilities

Trace automatically analyzes all of your application traces to generate in-depth latency reports to surface performance degradations.


#### Identifying bottlenecks:
Cloud Trace help inspect detailed latency information for a single request or view aggregate latency for your entire application. Using the various tools and filters provided, you can quickly find where bottlenecks are occurring and more quickly identify their root cause.

Trace is based off the tools used at Google to keep our services running at extreme scale.


#### Issue detection
Trace continuously gathers and analyzes trace data from your project to automatically identify recent changes to the performance of your application.

These latency distributions, available through the Analysis Reports feature, can be compared over time or versions. If Trace detects a significant shift in the latency profile of your app, you’re automatically alerted


## Terminology

To use Cloud Trace effectively, you must understand the relationship between traces and spans, and what tracing client is. A Trace consists of a tracing client, which collects spans and sends them to Cloud Trace. You can then use the Google Cloud console to view and analyze the data collected by the agent.

- **Trace**: Describes the time that it takes an application to complete a single operation. A trace is a collection of spans.
    
- **Span**: Describes how long it takes to perform a complete suboperation. For example, a trace might describe how long it takes to process an incoming request from a user and return a response. A span might describe how long a particular RPC call requires.
    
- **Tracing Client**: Collects spans and sends them to Cloud Trace.
![[Pasted image 20260807011630.png]]- If an OpenCensus library is available for your programming language, you can simplify the process of creating and sending trace data by using OpenCensus. In addition to being simpler to use, OpenCensus implements batching that might improve performance.
    
- •
    
    If an OpenCensus library doesn't exist, instrument your code by importing the Trace SDK library and using the Cloud Trace API. The Cloud Trace API collects trace data and sends it to your Google Cloud project.
    


### Sending trace data to Cloud Trace

There are two ways to send trace data to Cloud Trace.

_Click each button to learn more about the ways to send trace data to Cloud Trace._

### Automatic tracing

The first option is with automatic tracing. Some configurations support it and it includes:

- App Engine standard environment with Java 8, Python 2, and PHP 5 applications.
    
- HTTP requests and latency data from Cloud Run functions and Cloud Run.
    

![](https://storage.googleapis.com/cloud-training/cls-html5-courses/T-STACKD2-B/v1.2/EN/content/assets/Automatic.jpg)

### Instrumenting the application

The second option is by instrumenting the application. You can do this by using Google client libraries or OpenTelemetry, which is the recommended option.

![](https://storage.googleapis.com/cloud-training/cls-html5-courses/T-STACKD2-B/v1.2/EN/content/assets/Instrumenting.png)

> [!WARNING]
Required IAM permissions
Trace will need to offload tracing metrics to Google Cloud.  As far as the required IAM permissions are concerned, for external systems, or Compute Engine and GKE environments that don't run under the default service account, ensure that they run under a service account with at least the Cloud Trace Agent role.
App Engine, Cloud Run, Cloud Run functions, Google Kubernetes Engine, and Compute Engine have default access. However, Compute Engine and GKE get that access through the default Compute Engine service account.


![[Pasted image 20260807012024.png]]


# Cloud Profiler

![[Pasted image 20260807023658.png]]

Cloud Profiler is a statistical, low-overhead profiler that continuously gathers CPU usage and memory-allocation information from your production applications. It attributes that information to the source code that generated it, which helps you identify the parts of your application that are consuming the most resources. The insights provided illuminate the performance of your application characteristics.



## It provides low-impact profiling

Although it's possible to measure code performance in development environments, the results don’t often show what happens in production. Many production profiling techniques slow down code execution or can only inspect a small subset of a codebase. A comprehensive application performance view is provided without slowing it down. 

### Practical application profile creation

Poorly performing code increases latency and costs for web applications and services every day, without anyone knowing or doing anything about it. Cloud Profiler changes this situation by continually analyzing the performance of CPU or memory-intensive functions that run in an application.

Cloud Profiler presents the call hierarchy and resource consumption of the corresponding function in an interactive flame graph. This graph helps developers figure out which paths are consuming the most resources and what are the different ways your code is called.

### Broad language and product support

Cloud Profiler enables developers to analyze applications that run anywhere, including Google Cloud (App Engine, Compute Engine, GKE) and other on-premises or cloud platforms that support Java, Go, Node.js, and Python. 



## CPU metrics


- CPU time is the time that the CPU spends executing a block of code. The time it was waiting or processing instructions for something else is not included.
    
- Wall time is the time that it takes to run a block of code, including all wait time, including that for locks and thread synchronization. The wall time for a block of code can never be less than the CPU time.

## Heap metrics
- Heap is the amount of memory allocated in the heap of the program when the profile is collected.
    
- Allocated heap is the total amount of memory that was allocated in the heap of the program. Allocated heap includes memory that has been freed and is no longer in use.
## Thread metrics

- Contention provides information about threads stuck waiting for other threads.
    
- Threads contains thread counts.

|**Profile type**|**Go**|Java|Node.js|Python|
|---|---|---|---|---|
|CPU time|Yes|Yes||Yes|
|Heap|Yes|Yes|Yes||
|Allocated heap|Yes||||
|Contention|Yes||||
|Threads|Yes||||
|Wall time||Yes|Yes|Yes|

### Environment

Profiler instruments applications that run in most Google and non-Google compute technologies. Note that Windows guest OS is not supported.

| **Environment**                 | **Go** | Java | Node.js | Python |
| ------------------------------- | ------ | ---- | ------- | ------ |
| Compute Engine                  | Yes    | Yes  | Yes     | Yes    |
| GKE                             | Yes    | Yes  | Yes     | Yes    |
| App Engine flexible environment | Yes    | Yes  | Yes     | Yes    |
| App Engine standard environment | Yes    | Yes  | Yes     | Yes    |
| Dataproc                        |        | Yes  |         |        |
| Dataflow                        |        | Yes  |         | Yes    |
| Outside of Google Cloud         | Yes    | Yes  | Yes     | Yes    |

## Setting up the Profiler agent

Like with other Google application performance management products, the exact setup steps vary by language, so refer the [documentation(opens in a new tab)](https://cloud.google.com/profiler/docs/profiling-python) to find more information. Here, we are sticking with our Python application, which will run on App Engine.

Before you start, ensure that the **Profiler API** is enabled in your project.

1. Start by importing the _googlecloudprofiler_ package. 
    
2. Install the C/C++ compiler and development tools, pip, and Profiler package.
    
3. Then, early as possible in your code, start the profiler.  
    In this example, we are setting the logging level (verbose) to 3, or debug level. That setting will log all messages. The default would be 0 or error only.
    
```shell

gcloud services enable cloudprofiler.googleapis.com

sudo apt-get install -y build-essential
sudo apt-get install -y python3-pip

pip3 install google-cloud-profiler
```

```python
import googlecloudprofiler
def main():
def main():
   try:
      googlecloudprofiler.start(verbose=3)
   except (ValueError, NotImplementedError) as exc:
      print(exc)  # Handle errors here
```

## Analyzing the Profiler interface

The Profiler interface uses a Flame Graph to help you visualize resource consumption across your entire application.
![[Pasted image 20260807024229.png]]

### The Flame Graph

Cloud Profiler displays profiling data by using [Flame Graphs(opens in a new tab)](https://www.brendangregg.com/flamegraphs.html). Unlike trees and standard graphs, flame graphs use screen space efficiently by representing a large amount of information in a compact and readable format.

### Understand the structure

![Architecture diagram of a flame graph.](https://storage.googleapis.com/cloud-training/cls-html5-courses/T-STACKD2-B/v1.2/EN/content/assets/Flame%20graph%201.png)

In this example we have a basic application with a _main_ method, which calls _foo1_, which in turn calls _bar_. Then _main_ calls _foo2_, which also calls _bar_.

As you move through the graphic left to right, you can see how the information is collapsed. First, by removing arrows, then by creating frames, and finally by removing spaces and left-aligning.

In the bottom view, you see the result as it appears in the Profiler.


### Time

![Diagram showing CPU time usage.](https://storage.googleapis.com/cloud-training/cls-html5-courses/T-STACKD2-B/v1.2/EN/content/assets/flame%20graph%20time.jpg)

If you check CPU time, then you can review that the _main_ method takes a total of 9 seconds.

Beneath the _main_ bar, you can review how that CPU time was spent: some in _main_ itself, but most in the calls to _foo1_ and _foo2_. And most of the _foo_ time was spent in _bar_.

So, if we could make _bar_ faster, we could really save some time in _main_.


### Point to a frame

![](https://storage.googleapis.com/cloud-training/cls-html5-courses/T-STACKD2-B/v1.2/EN/content/assets/Flame%20graph%203%20hd.jpg)

When you hold the pointer over a frame, a tooltip opens and displays additional information, which includes:

- The function name
    
- The source file location
    
- And some metric consumption information

## Selecting a frame

![](https://storage.googleapis.com/cloud-training/cls-html5-courses/T-STACKD2-B/v1.2/EN/content/assets/Flame%20graph%204%20hd.jpg)

If you click a frame, the graph is redrawn, which makes the call stack of the selected method more visible.