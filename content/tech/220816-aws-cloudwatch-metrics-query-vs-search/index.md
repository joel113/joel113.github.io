+++
author = "Johannes Ehm"
title = "AWS CloudWatch Search and Query API"
date = "2022-08-16"
description = "A brief description of the AWS CloudWatch Search and Query API"
tags = [
    "aws",
    "cloudwatch",
		"metrics"
]
+++

AWS offers with CloudWatch a powerful service to observe a AWS infrastructure and applications which are running on the AWS infrastructure. Collecting and querying metrics is a major part of observing infrastructure and applications. When I was working with querying AWS CloudWatch metrics I did found out that AWS CloudWatch acutally provides to ways to query the metrics and that is important to know that two ways are available:

- CloudWatch Metrics Insights with a SQL kind query language
- Cloudwatch search expressions

## CloudWatch query expressions from Metrics Insights

CloudWatch Metrics Insight offers a rich sql kind query language to build queries on top of time series data.

https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/query_with_cloudwatch-metrics-insights.html

It is important to know, that CloudWatch Metric Insights offers strong restrictions on the amount of data which can be processed. The most important limit is the ability to query only for the last three hours of data and that each get metric operation can process only a single query.

https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch-metrics-insights-limits.html

If the features of CloudWatch Metrics Insights queries are not needed, the CloudWatch search expressions should be used which offer a more simple search expression language without limits.

## Clouddatch search expressions

CloudWatch search expressions offer a simple query language to retrieve time series data from CloudWatch. The syntax requires providing a namespace, dimensions, search terms, aggregation functions and the period.

https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/using-search-expressions.html

Opposite to CloudWatch Metrics Insights it is not possible e.g. to group or order time series data. Limits to the data which can be queried using search expressions are imposed from the data which is available at CloudWatch in generall. Data points of time series data with a frequency of less than 60 seconds are only available for 3 hours. After 15 days data is only available with a period of 300 seconds. After 455 days data is only available with a period of 3600 seconds.

## CloudWatch Get Metric Data

The GetMetricData API of CloudWatch is able to process the query language as well the search expressions.

https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_GetMetricData.html

However, be aware that there are also the search expressions when the restrictions of CloudWatch Metrics Insights are too tight or when the search expressions of CloudWatch Metrics is not expressive enough.
