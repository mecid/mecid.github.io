---
title: Monitoring app performance with MetricKit
layout: post
---

Xcode Organizer provides access to essential performance metrics such as crashes, energy impact, hangs, launch time, memory consumption, and app terminations. However, it lacks sufficient information to resolve certain issues, particularly app terminations. To address this, Apple introduced the MetricKit framework, enabling us to collect comprehensive performance diagnostics and build a detailed performance dashboard.

To monitor app performance, we need to gather performance data and export it for analysis. The most straightforward way to achieve this is by using an analytics tool. Let’s begin building our app performance monitoring dashboard.

```swift
protocol Analytics {
    func logEvent(_ name: String, value: String)
    func logCrash(_ crash: MXCrashDiagnostic)
}
```

Next, we have to import MetricKit and set up a subscription to receive data.

```swift
final class AppDelegate: NSObject, UIApplicationDelegate, MXMetricManagerSubscriber {
    private var analytics: Analytics?

    func applicationDidFinishLaunching(_ application: UIApplication) {
        MXMetricManager.shared.add(self)
    }

    nonisolated func didReceive(_ payloads: [MXMetricPayload]) {
        for payload in payloads {
            if let exitMetrics = payload.applicationExitMetrics?.backgroundExitData {
                analytics?.logEvent(
                    "performance_abnormal_exit",
                    value: exitMetrics.cumulativeAbnormalExitCount.formatted()
                )
                
                analytics?.logEvent(
                    "performance_cpu_exit",
                    value: exitMetrics.cumulativeCPUResourceLimitExitCount.formatted()
                )
                    
                analytics?.logEvent(
                    "performance_memory_exit",
                    value: exitMetrics.cumulativeMemoryPressureExitCount.formatted()
                )
                
                analytics?.logEvent(
                    "performance_oom_exit",
                    value: exitMetrics.cumulativeMemoryResourceLimitExitCount.formatted()
                )
            }
        }
    }

    nonisolated func didReceive(_ payloads: [MXDiagnosticPayload]) {
        for payload in payloads {
            if let crashes = payload.crashDiagnostics {
                for crash in crashes {
                    analytics?.logCrash(crash)
                }
            }
        }
    }
}
```

As demonstrated in the example above, we utilize the shared instance of the MXMetricManager type to add a subscriber. Our AppDelegate type conforms to the MXMetricManagerSubscriber protocol, which includes two optional functions that enable us to receive metrics and diagnostics.

The MXMetricPayload type contains a collection of properties that adhere to the MXMetric abstract class. For instance, it includes applicationLaunchMetrics and applicationExitMetrics properties that provide comprehensive details. In our instance, I log a few intriguing background terminations. This knowledge enables me to comprehend the reasons behind the system’s termination of the app.

The MXDiagnosticPayload type contains a collection of properties that extend the abstract MXDiagnostic class. For example, it includes cpuExceptionDiagnostics and crashDiagnostics. We use the logCrash function to extract valuable details and log them.

Both MXMetricPayload and MXDiagnosticPayload offer us JSON and dictionary representations of the properties, enabling us to effortlessly upload this information to our custom-made API endpoint and process it on the backend.

Keep in mind that the MXMetricManager may not always receive payloads. The system can aggregate the data and deliver it on a daily schedule. In rare cases, it may deliver more frequently, but you should not rely on any specific schedule. Both payload types provide the timeStampBegin and timeStampEnd properties, which allow us to determine the range of time covered by each payload.

MetricKit fills a critical gap left by Xcode Organizer by giving us deep, system-level insight into how an app behaves in real-world conditions. By subscribing to MXMetricManager and processing MXMetricPayload and MXDiagnosticPayload, we gain visibility into app launches, terminations, crashes, and resource usage that would otherwise be difficult—or impossible—to fully understand.
