# Azure Monitor

Azure monitor is a solution that gathers information on how your application is running.

It could be split up to these three components:
- *Data source* : is the application and workload itself.
- *Data Platform:* Stores the monitoring data.
- *Consumption*: Makes use of that data to find insight.

![](Images/Pasted%20image%2020260103131332.png)
## Data Platform 
*Metrics*: are numerical values that describe an aspect of a system at a particular point in time.  Azure Monitor Metrics is a time-series database.

*Logs*: Logs can contain different types of data, be structured or free-form text, and they contain a timestamp.

*Traces*: Distributed tracing allows you to see the path of a request as it travels through different services and components. Azure Monitor gets distributed trace data from instrumented applications.

*Changes*: Logs  changes such as deployments that may have caused issues in your system.
## Azure Monitor Application Insights
Integrating with Open Telemetry (OTel) provides a vendor-neutral approach to collecting and analysing telemetry data, enabling comprehensive observability of your applications.

This provides the benefit to export your logs as is to services that [supports](https://opentelemetry.io/ecosystem/vendors/)  OTel. OTel is a mature project on the Cloud Native Computing foundation.

There are two endpoint that ingest data from you application and some firewall changes need to be made to allow this.

| Purpose      | Hostname                                                 | Type     | Ports |
| ------------ | -------------------------------------------------------- | -------- | ----- |
| Telemetry    | `{region}.in.applicationinsights.azure.com`              | Regional | 443   |
| Live Metrics | `{region}.livediagnostics.monitor.azure.com`  <br>  <br> | Regional | 443   |

![](Images/Pasted%20image%2020260103133814.png)