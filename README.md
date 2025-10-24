# Instana Observability, selected topics

Topics:
- Events, Alerts, and Smart Alerts
- Automation
- Synthetic Monitoring
- Logs

**Events, Alerts, and Smart Alerts**

Instana defines almost 500 built-in events that can be fired by Instana sensors.</br>
Events represent curated knowledge catpured in Instana.<br/>

Set of events is extensible with custom events.<br/>

Events types: Change, Issue, or Incident.<br/>

*Changes* are created for `online/offline`, or *change detected* transitions.<br/>
*Issues* are created if entity is `unhealthy`. All issues are independent from one another.<br/>
*Incident* is an aggregation of related issues impacting a service. Incidents are linked to the *edge* service from the dependency graph. Incidents prevent alert storms because alerts are not created for related services.<br/>

All events are displayed in the Instana UI *Events* section grouped by type.<br/>

A *change* has a start time and end time.<br/>
An *issue* has a start time, duration, and end time. An issue can be closed manually, or automatically.<br/>
An *incident* has a triggering event, duration, and end time. An incident is closed when all related issues are closed.<br/>

*Change example*
Changes are created for *online*, *offline*, or *change detected* transitions.<br/>
```
Change detected
Time: Ended: Duration: -
Triggering Entity: openshift-marketplace/certified-operators-dwjkg
Description:
The value podIp has changed from "" to "10.128.2.75".
The value phase has changed from "Pending" to "Running".
State: closed by Instana
```

*Issue example*
```
Garbage Collection Activity High (10.03%)
Started:, Ended:, Duration: 2 minutes
State: closed by Instana
Triggering Entity: shipping service 1.0 (java)
Metrics: PS Scavenge Time 5000 ms
Recommended actions: AI Actions, policies
```

```
Killed by out of memory killer
Started:, Ended:, Duration: 3 m, Severity: Critical
State: closed by Instana
Triggering entity: nginx: master process nginx-debug -g daemon off;
Description: This process received an 'out-of-memory-kill' event; at the time it had allocated 100 MB of memory over 25600 pages of 4 KB each.
```

*Incident example*
This incident is triggered by *Smart Alert*
```
Problem on WebSphere - datasources.BPM Business Space data source.averageWaitTime detected
Triggered:, Ended:, Duration: 18h 53m 11s
Severity: Warning
Affected entities: 1
Triggering event: WebSphere - datasources.BPM Business Space data source.averageWaitTime is greater than or equal to 0
Triggering entity: WebSphere @PCCell1/Node1/AppClusterMember1
Metric violations:
Related events: (13)
Recommended actions: policy1, confidence: high
```


*Alerts*

Alerts are created in the *Settings/Alerts* section of Instana UI.<br/>

An alert is defined for one or more events or event types (change, issue, incident).<br/>
An alert is narrowed down by *scope* to an application of dynamic query.<br/>
Alerts can be enhanced with custom payload.<br/>

Alert is published on one or more *Alert Channels*.<br/>

Create one or more *Alert Channels* before creating an alert.<br/>

Alert will fire every time in-scope event is raised by instana sensor.<br/>

Incident alerting is optimized to prevent event storms.<br/>
Alerts are not filred for related services.<br/>


*Smart Alerts*

*Static Alerts* vs *Smart Alerts*
- static thresholds vs baselines
- curated knowledge vs learned behaviour

*Smart Alerts* can be defined for:<br/>
- Infrastructure
- Applications
- WebSites
- MobileApplications
- Logs
- Synthetics
- Service Levels

*Smart Alerts* use machine learning to fire alerts.<br/>

All *Smart Alerts* can be configured to trigger *Incidents*.<br/>

*Forecast alerting* (preview) will fire proactive alerts based on historical data.<br/>

*Infrastructure Smart Alert*

host metric *h.cpu_system_usage* is collected every 1 second.<br/>
alert is fired for every metric.<br/>

```
for (all h: {h: host, h.cpu_count > 4}) fire smart alert if threshold (mean(bucket: {h}, h.cpu_system_usage over 10 mins)) < 2% is false for 30 mins or more.
```

host metric *h.cpu_system_usage* is collected every 1 second.<br/>
host entities are grouped by zone: group(h.zone)<br/>
alert is fired per group.<br/>

```
for (all g: {h: host, h.cpu_count > 4, group_by(h.zone)}) fire smart alert if threshold (mean(bucket: {all h in g}, h.cpu_system_usage over 10 mins)) < 2% is false for 30 mins or more.
```

*Application Smart Alert*

Application Smart Alerts support different scenarios (blueprints):
- slow calls: Receive an alert when calls to selected services and endpoints of this application perspective are slower than usual.
- erroneous calls: Receive an alert when the rate or count of erroneous calls for selected services and endpoints of this Application Perspective is higher than normal.
- http status codes: Receive an alert every time when matching HTTP status codes occur more often than usual.
- low number of calls: Receive an alert when the number of calls is significantly lower than expected compared to the available past data. This might be an indication of a problem upstream of this application or a drop in the user traffic to the application.
- high number of calls: Receive an alert when the number of calls is significantly higher than expected compared to the available past data. This might be an indication of an attack or a bot generating too many requests to the application.

*Application Smart Alerts* can be fired per application, per service, or per endpoint in the application.<br/>

*Examples*

Slow calls, static threshold.<br/>
```
for (a: application, a="baw", bpm.activity_name="hello") fire smart alert if threshold (mean(bucket: {all s in a}, a.latency over 10 mins)) < 10 ms is false for 30 mins or more
```

slow calls, threshold over static daily sesonality.<br/>
By how many much *latency mean over 10 mins* exceeds *latency static daily seasonality* when measured in number of std deviations.<br/>
```
for (a: application, a="baw", bpm.activity_name="hello") fire smart alert if threshold (mean(bucket: {all s in a}, a.latency over 10 min)) >= (latency_static_daily_seasonality by 5 std deviations) is false for 30 mins or more
```

Slow calls, adaptive threshold.<br/>
By how many much *latency mean over 10 mins* exceeds *latency adaptive threshold* when measured in number of std deviations.<br/>
```
for (a: application, a="baw", bpm.activity_name="hello") fire smart alert if threshold (mean(bucket: {all s in a}, a.latency over 10 min)) >= (latency_adaptive_threshold by 5 std deviations) is false for 30 mins or more
```


*Logs Smart Alert*

*Logs Smart Alert* is configured for the *Log Count* metric.<br/>

Examples:<br/>

Log count static threshold.<br/>
```
for (l: log, logs.message=~"hello") fire smart alert if threshold count(bucket: {l}, l.logs) > 100 is false for 30 mins or more
```

Log count static threshold, fire per group:<br/>
```
for (all g: {l: log, logs.message=~"hello", group_by(l.zone)}) fire smart alert if threshold (bucket: {all l in g}, l.log_count) > 100 is false for 30 mins or more
```

*Synthetics Smart Alert*

*Synthetic Test* runs a script from defined *Location*.

Example:<br/>

*Synthetics Smart Alert* is defined for the number of times *Syntetic Test* fails per location.<br/>

```
for (t: SynthaticTest, t.location="east-1") fire smart alert if threshold count(bucket: {t}, t.fail)) > 5
```

*Service Levels Smart Alert*

*Service Level Objective* measures application performance metric against a goal and error budget.<br/>

Example:<br/>

*Service Level Smart Alert* is written against error budget consumption percentage.<br/>

```
for (slo: serviceLevelObjective) fire smart alert if threshold (bucket: {slo}, slo.error_budget_consumed) > 10% is false for 30 mins or more
```


**Automation**

Enable Automation Framework<br/>
```
...
featureFlags:
  - name:  feature.automation.enabled
    enabled: true
...
```

Enable action sensors to match action types<br/>

Action Script sensor:<br/>
```
com.instana.plugin.action.script:
  enabled: true # by default is false

  # spaces not allowed in directory name
  scriptExecutionHome: /home/simon/instana-agent/script-execution-home

  # no 'root' or 'Administrator'
  runAs: user

  # required for Windows
  runAsUserPassword: password
    configuration_from:
      type: vault
      secret_key:
        path: <secret_path>
        key: <secret_key>

  chrootEnabled: false
```

Additional requirements:
- agent is not a Windows service
- powershell 7.4 or later
- set runAsUserPassword
- runAs user no permissions to agent install folder


*Matching event to an action*
- event similarity: match an event against policy events. set policy action score based on event similarity.
- nlp: match event against actions in the action catalog. calc action score.
- success rate: an action is given a score based on sucess rate for the action in the past occurences of the event.
- policy: event matches policy trigger. policy is listed in 'recommeded actions' with high score.
- turbonomic: if the event entity has turbonomic actions linked to it, these actions are recommended based on the nlp score.

Action scores are classified into low/medium/high.<br/>
Recommended actions include policies for the event, and actions with *medium* and *high* scores.<br/>

For the incident, lookup recommended actions from the triggering event, or probable root cause.<br/>

*intellegent remidiation with watsonx*
Use gen ai to create new actions for the event.<br/>
Event details is llm prompt.<br/>

There are many types of actions...<br/>

We'll use example of a script action.<br/>
Script action parameters can be static or dynamic.<br/>

The values of static parameters are assigned at the time of parameter definition.<br/>

The values of dynamic parameters are based on tags.<br/>
Eg: We can assign host.fqdn tag to script parameter to represent a host name.<br/>

As an example, we will create 'Must Gather' script action for Business Automation Workflow.<br/>

*Policy: Linking actions to smart alerts*
Policies link Smart Alerts to actions.<br/>

There are 2 types of policies: user initiated and automatic.<br/>

User initiated policies are run manually by the user.<br/>

For automatic policies, action scope is determined from the scope of a firing smart alert.<br/>
Automatic action scope can be restricted to the subset of the event scope by dynamic focus query.<br/>

When event occurance matches conditions in policy trigger, the policy is listed in *recommended actions*<br/>

*Recommended actions*
Policies in *recommended actions* are listed first, followed by other recommendations based on the events similarity score.<br/>


**Synthetic Monitoring**

*POP* is a runtime to host synthetic tests. *POP* (Point of Presense) represents remote user location.<br/>

*POP* runtime can host several types of synthetic tests:<br/>
- API Simple test: rest API
- API Script test: multiple API's on NodeJS runtime
- Browser Simple test: simple HTTP request from the browser.
- Browser Script test: NodeJS browser scripts, or Selenium IDE scripts.
- SSL Certificate test: check if certificate is about to expire.
- DNS test: DNS query.

*PoP* runs tests assigned to it's location. For each test run *PoP* sends test results to Instana backend.<br/>

*POP* is deployed as a helm chart on a kubernetes cluster.<br/>

There are many options to configure, secure, and tune Synthetic Monitoring helm chart deployment.<br/>
Instana UI can help with the initial configuration values.<br/>

```
helm install synthetic-pop \
  --repo "https://agents.instana.io/helm" \
  --namespace <namespace> \
  --create-namespace \
  --set downloadKey="<download key>" \
  --set controller.location="<yourLocationName>;<yourLocationDisplayName>;<yourLocationCountry>;<yourLocationCity>;0;0;<yourLocationDescription>" \
  --set controller.clusterName="<yourClusterName>" \
  --set controller.instanaKey="<instana key>" \
  --set controller.instanaSyntheticEndpoint="https://szesto.io/synthetics" \
  --set redis.tls.enabled=false \
  --set redis.password="<yourPassword>" \
  synthetic-pop
```

Create synthetic test and add it to the location.<br/>

Write a test script in Instana UI, or download external script file.<br/>

Schedule test for execution.<br/>

Configure *Smart Alerts* on Synthetic Test execution results.<br/>
For *Smart Alert* configure test, scope, how many times a test should fail per location before alerting, alert channel, and properties.<br/>

*Logs in Context*

Logs:
- captured by the tracers, container sensors, level >= warn
- ingested with open telemetry
- tracing sdk

logs are correlated with traces and metrics (show)
traces and calls are enhanced with log messages (show)

*Opentelemetry logs*

Opentelemetry logs require open telemetry contrib collector.

Logs smart alerts


*logs*
```
using Instana, logging
https://www.ibm.com/docs/en/instana-observability/1.0.306?topic=instana-logging
```

*mutliline log messages*
*stack traces*
*custom dashboards*

*opentelemetry semantics conventions for logs*

*Opentelemetry collector*
[instana logging use](https://www.ibm.com/docs/en/instana-observability/1.0.306?topic=logs-collecting-opentelemetry)

```
#!/bin/bash

download="https://github.com/open-telemetry/opentelemetry-collector-releases/releases/download"

# min version: v0.110.0
version="0.136.0"

os="linux"|"windows"
arch="amd64"

coltar="otelcol-contrib_${version}_${os}_${arch}.tar.gz"

wget ${download}/v${version}/${coltar}
```

```
https://github.com/open-telemetry/opentelemetry-collector-releases/releases/download/v0.136.0/otelcol-contrib_0.136.0_windows_amd64.tar.gz
```

```
https://github.com/open-telemetry/opentelemetry-collector-releases/releases/download/v0.136.0/otelcol-contrib_0.136.0_linux_amd64.tar.gz
```

*collector configuration*
```
receivers:
  ## [REQUIRED] The filelog receiver will collect logs written to file by a process
  filelog:
    ## [REQUIRED] Path (or regex) to the log files that must be read.
    include: [ "/path/to/log/files/to/read" ]

    ## [OPTIONAL] Path (or regex) to the log files that must be ignored.
    exclude: [ "/path/to/log/files/to/ignore" ]

    ## [REQUIRED] Whether to include the file path in the logs
    include_file_path: true

    ## [OPTIONAL] Whether to include the file name in the logs
    include_file_name: true

    ## [OPTIONAL] Preserve the leading white spaces so that the example 'recombine' operator works as expected.
    preserve_leading_whitespaces: true

    operators:
      ## [OPTIONAL] Example recombine operator config to handle multi-line log messages for stack-traces. Requires `include_file_path: true` above.
      - type: recombine
        combine_field: body
        is_first_entry: body matches "^[^\\s]"
        source_identifier: attributes["log.file.path"]


processors:
  ## [OPTIONAL] This is an example log severity parser that sets the **severity_text** field in the log payload, each runs in-order such that the highest matching severity is set.
  ## Note: If the OpenTelemetry Collector does not set log severity, then the severity is set by Instana when analyzing the log message.
  transform/set_log_severity:
    log_statements:
    - context: log
      statements:
      - set(severity_text, "Info") where IsMatch(body.string, ".*INFO.*")
      - set(severity_text, "Warn") where IsMatch(body.string, ".*WARN.*")
      - set(severity_text, "Error") where IsMatch(body.string, ".*ERROR.*")
      - set(severity_text, "Fatal") where IsMatch(body.string, ".*FATAL.*")

  ## Logs must be sent in batches for performance reasons.
  ## Note: No additional `batch` processor configuration is provided since configuration depends on the user scenario.
  batch: {}

exporters:
  ## [REQUIRED] The Instana Agent supports GRPC payloads
  otlp/instanaAgent:
    ## The GRPC port will be 4317 (unless port-forwarding is used to change this).
    ## Note: Be sure to set Instana Agent's OTLP endpoint HOST:PORT combination.
    endpoint: "INSTANA_AGENT_HOST:INSTANA_AGENT_GRPC_PORT"
    ## TLS encryption is disabled in this example.
    tls:
      insecure: true

service:
  pipelines:
    ## Sample logs pipeline using the above configurations.
    logs:
      receivers: [filelog]
      processors: [transform/set_log_severity, batch]
      exporters: [otlp/instanaAgent]

```


*start otel collector*
```
/otelcol-contrib --config=./otel-config.yaml
```

*Automation*
enable action sensors
link actions to events | or smart alerts. create policy to configure this association.
run an action from event instance
link event (and action) to smart alert


enable actions feature

```
featureFlags:
  - name:  feature.automation.enabled
    enabled: true
```


```
com.instana.plugin.action.http:
  enabled: true # by default is false
  maxConcurrentActions: 10
  defaultTimeout: 10
```


*Synthetic monitoring*

enable feature flag

```
featureFlags:
    - name: feature.synthetics.enabled
      enabled: true
```

