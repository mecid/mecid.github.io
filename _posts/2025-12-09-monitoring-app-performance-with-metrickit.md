---
title: Monitoring app performance with MetricKit
layout: post
---

Xcode Organizer provides access to essential performance metrics such as crashes, energy impact, hangs, launch time, memory consumption, and app terminations. However, it lacks sufficient information to resolve certain issues, particularly app terminations. To address this, Apple introduced the MetricKit framework, enabling us to collect comprehensive performance diagnostics and build a detailed performance dashboard.

To monitor app performance, we need to gather performance data and export it for analysis. The most straightforward way to achieve this is by using an analytics tool. Let’s begin building our app performance monitoring dashboard.

======================================================

Next, we have to import MetricKit and set up a subscription to receive data.

======================================================

As demonstrated in the example above, we utilize the shared instance of the MXMetricManager type to add a subscriber. Our AppDelegate type conforms to the MXMetricManagerSubscriber protocol, which includes two optional functions that enable us to receive metrics and diagnostics.

The MXMetricPayload type contains a collection of properties that adhere to the MXMetric abstract class. For instance, it includes applicationLaunchMetrics and applicationExitMetrics properties that provide comprehensive details. In our instance, I log a few intriguing background terminations. This knowledge enables me to comprehend the reasons behind the system’s termination of the app.

The MXDiagnosticPayload type contains a collection of properties that extend the abstract MXDiagnostic class. For example, it includes cpuExceptionDiagnostics and crashDiagnostics. We use the logCrash function to extract valuable details and log them.

Both MXMetricPayload and MXDiagnosticPayload offer us JSON and dictionary representations of the properties, enabling us to effortlessly upload this information to our custom-made API endpoint and process it on the backend.

Keep in mind that the MXMetricManager may not always receive payloads. The system can aggregate the data and deliver it on a daily schedule. In rare cases, it may deliver more frequently, but you should not rely on any specific schedule. Both payload types provide the timeStampBegin and timeStampEnd properties, which allow us to determine the range of time covered by each payload.

MetricKit fills a critical gap left by Xcode Organizer by giving us deep, system-level insight into how an app behaves in real-world conditions. By subscribing to MXMetricManager and processing MXMetricPayload and MXDiagnosticPayload, we gain visibility into app launches, terminations, crashes, and resource usage that would otherwise be difficult—or impossible—to fully understand.
