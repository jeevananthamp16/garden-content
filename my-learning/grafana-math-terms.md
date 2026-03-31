---
title: "Grafana - Math Terms"
visibility: "public"
date: "2026-03-30"
tags: []
---

In Grafana, mathematical operations can be applied using the Query Language of your specific database or through Grafana's Transformations and Expressions features. 

1. Standard Deviation (StdDev)
Standard deviation measures how much values vary from the average. A high value suggests high volatility, while a low value indicates stability.
Example: Monitoring the response time of a web server. If the average is 200ms but the standard deviation is 500ms, your users are likely experiencing inconsistent performance.
Grafana Implementation:
Prometheus (PromQL): Use stddev_over_time(metric[range]) to see variation over a period or stddev(metric) for variation across multiple instances.
InfluxDB (Flux): Use the stddev() function on a stream of data.
Transformation: Go to the Transform tab, select Reduce, and choose StdDev as the calculation to show a single value in a Stat or Gauge panel. 

2. Rate
Rate calculates how fast a value is increasing or decreasing over a specific time interval. It is primarily used for "Counter" metrics that only go up (like total requests).
Example: Calculating "Requests per second" from a counter that tracks the "Total requests since the server started".
Grafana Implementation:
Prometheus: Use rate(metric[5m]). This calculates the per-second average rate of increase over the last 5 minutes.
InfluxDB (Flux): Use aggregateWindow(every: 1m, fn: rate) or the derivative() function to find the rate of change between points.
SQL (PostgreSQL/MySQL): Typically implemented using window functions like LAG() to find the difference between the current and previous value divided by the time difference. 

3. Quantile (Percentile)
A quantile (e.g., 0.95 or 95th percentile) shows the value below which a certain percentage of data falls. It is better than an "Average" for understanding user experience because it ignores minor outliers but captures the "worst-case" scenarios. 
Example: A 95th percentile latency of 300ms means 95% of your users are getting a response in less than 300ms, while only 5% are seeing slower times.
Grafana Implementation:
Prometheus: Use histogram_quantile(0.95, sum by (le) (rate(metric_bucket[5m]))) for histogram data. Use quantile_over_time(0.95, metric[1h]) to find the percentile of a single metric over the last hour.
InfluxDB (Flux): Use quantile(q: 0.95) to return the 95th percentile value from your data set.
Transformation: Use the Sort by or Filter by Value transformations if you need to manually identify top-tier data points within a table. 

4. Basic Arithmetic (Expressions)
If you need to perform simple math like adding two queries together or calculating a percentage (e.g., (Success / Total) * 100).
Grafana Implementation:
Expressions: Below your queries, click + Add expression. You can use syntax like $A / $B where 
 and 
 are your query labels.
Binary Operators: Data sources like Prometheus support direct operators in the query field: node_memory_Active_bytes / node_memory_MemTotal_bytes * 100. 

Beyond the basics, there are several "advanced" mathematical concepts used in Grafana to make messy raw data readable and actionable.

1. Derivatives (Velocity)
While a Rate usually looks at how much a counter (like total bytes) increases per second, a Derivative measures the "steepness" of the change between any two data points.
Concept: If Rate is your speed, Derivative is your acceleration. It tells you how fast a value is changing right now compared to a moment ago.
Example: Monitoring disk space. A derivative helps you see if the disk is filling up faster today than it was yesterday.
Grafana Implementation: In InfluxDB, use derivative(1m). In Prometheus, idelta() or irate() are often used to see the change between the last two samples.

2. Histograms (Distribution)
Instead of one average number, a Histogram counts how many events fall into specific "buckets."
Concept: Imagine sorting people by height into boxes: 5ft–5.5ft, 5.5ft–6ft, etc. It shows you the "shape" of your data.
Example: Web latency. You might see a "bimodal" distribution where most users are fast (200ms), but a specific group is very slow (2s) due to a bug.
Grafana Implementation: Use the Heatmap panel. It takes histogram data and turns it into a "weather map" of your performance.

3. Moving Averages (Smoothing)
Real-world data is often "noisy" with lots of tiny spikes that don't matter. A Moving Average averages the last 
 data points to create a smooth line.
Concept: It "blurs" the data so you can see the long-term trend rather than the instant noise.
Example: CPU usage that jumps between 10% and 90% every second. A 5-minute moving average shows you the actual load on the system.
Grafana Implementation: In Prometheus, use avg_over_time(metric[5m]). In Grafana Transformations, use the Binary Operation or Reduce to calculate means over windows.

4. Top-K and Bottom-K (Filtering)
When you have 1,000 servers, you don't want to see 1,000 lines on a graph. You only want the "worst" ones.
Concept: Automatically filtering the results to show only the highest or lowest values.
Example: "Show me the top 5 containers consuming the most RAM."
Grafana Implementation: In Prometheus, use topk(5, metric). In SQL, use ORDER BY value DESC LIMIT 5.

5. Offset (Time Shifting)
This compares your current data to data from the past (e.g., "now" vs. "this time last week").
Concept: Calculating the difference between two points in time to spot anomalies.
Example: "Is my traffic today normal for a Monday morning?"
Grafana Implementation: In Prometheus, use metric offset 1w. You can then use a Grafana Expression to subtract the old value from the new one: $A - $B.

To round out your Grafana toolkit, here are more advanced data concepts that help you handle "noisy" metrics, find hidden patterns, and compare data across different timeframes.

1. Interquartile Range (IQR) & Outliers
While Standard Deviation tells you how "spread out" data is, IQR helps you find the "middle 50%" and identify outliers (data points that are abnormally high or low).
Concept: It ignores the extreme top and bottom 25% of your data to focus on the "normal" range.
Example: If 99 servers are at 10% CPU but 1 server is at 100%, the average will look high (10.9%). IQR helps you ignore that one "weird" server to see the healthy baseline.
Grafana Implementation: Use Prometheus quantile_over_time (calculating 0.75 minus 0.25) or use the Box Plot visualization in Grafana to see these ranges visually.

2. Correlation
Correlation measures how much two different metrics move together. If Metric A goes up, does Metric B go up too?
Concept: It helps you find the "Root Cause." If "Error Rate" and "Database Connections" spike at the exact same time, they are likely correlated.
Example: Proving that high CPU usage is being caused by an increase in user logins.
Grafana Implementation: Use the Trend or Scatter Plot panels. You can plot two queries against each other (X-axis vs Y-axis) to see if they form a straight line (strong correlation).

3. Cumulative Sum (Totalizing)
This adds each new data point to the sum of all previous points. It turns a "per second" metric into a "total so far" metric.
Concept: Instead of seeing "how much rain fell this minute," you see "how much total rain has fallen today."
Example: Tracking total data transferred (bandwidth) over a 24-hour billing cycle.
Grafana Implementation:
SQL: Use SUM(value) OVER (ORDER BY time).
Prometheus: Use increase(metric[range]) or sum_over_time.
Transformation: Use the Cumulative Sum transformation to turn a bar chart of daily sales into a rising line of "Total Sales."

4. Resampling & Bucketing (Downsampling)
When you have 1 million data points for a 1-hour graph, Grafana can't draw them all. Resampling groups those points into larger "buckets" (e.g., 1-minute chunks).
Concept: It simplifies high-resolution data so it loads faster and looks cleaner.
Example: Taking 1-second temperature readings and averaging them into 5-minute blocks to see the daily trend without the "jitter."
Grafana Implementation:
Prometheus: Handled automatically by the step parameter in the query.
InfluxDB: Use aggregateWindow(every: 5m, fn: mean).
Transformation: Use Group By to collapse multiple rows into one based on time.

5. Boolean Logic (Thresholding)
This converts a mathematical value into a "1" (True) or "0" (False).
Concept: It simplifies complex data into a binary "Is it broken or not?" status.
Example: If CPU > 90, return 1. Otherwise, return 0.
Grafana Implementation:
Expressions: Use a Math expression like $A > 90.
Value Mappings: In the panel settings, map the value 1 to the text "CRITICAL" and the color Red.





