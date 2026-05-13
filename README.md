# 🚨 Centralized Incident Management Architecture

This Kestra workflow is designed as a centralized incident management system. ⚙️

It uses a Flow Trigger to monitor other Kestra workflows for failures, captures detailed execution context, and orchestrates automated actions based on severity including:

* 📢 Slack notifications
* 🎫 ServiceNow incident creation
* 📝 Operational logging
* 🚦 Severity-based routing
* 🔁 Retry orchestration

---

# ⚠️ Important Configuration Notes

## 📌 Upstream Flow Requirements

Ensure upstream workflows publish structured error details as outputs.

The `incident-management` workflow expects an `errorDetails` JSON object for comprehensive context capture including:

* ❌ failed task name
* 💬 error message
* 📚 stack trace
* 🔄 retry count
* 📦 input payload

## 🧩 Example Upstream Flow Configuration

```yaml
outputs:
  - id: errorDetails
    type: JSON
    value: |
      {
        "message": "{{ task.outputs.errorDetails.value.error.message | default('') }}",
        "stackTrace": "{{ task.outputs.errorDetails.value.error.stackTrace | default('') }}",
        "errorDetailsId": "{{ task.outputs.errorDetails.value.id | default('Unknown') }}",
        "retryAttempt": "{{ task.outputs.errorDetails.value.attempts.count | default(0) }}",
        "inputPayload": {{ task.outputs.errorDetails.value.inputs | json | default('{}') }}
      }

tasks:
  - id: my_failing_task
    type: some.plugin.FailingTask

    errors:
      - id: errorDetails
        type: io.kestra.plugin.core.debug.Return
        format: "{{ taskrun | json }}"
```

---

# 🔐 Required Kestra Secrets

Configure the following secrets in your Kestra environment.

## 💬 Slack Secrets

```bash
kestra secrets create SLACK_CRITICAL_WEBHOOK "<webhook-url>"

kestra secrets create SLACK_GENERAL_WEBHOOK "<webhook-url>"

kestra secrets create SLACK_INFO_WEBHOOK "<webhook-url>"
```

## 🎫 ServiceNow Secrets

```bash
kestra secrets create SNOW_DOMAIN "<instance>.service-now.com"

kestra secrets create SNOW_USERNAME "<username>"

kestra secrets create SNOW_PASSWORD "<password>"

kestra secrets create SNOW_CLIENT_ID "<client-id>"

kestra secrets create SNOW_CLIENT_SECRET "<client-secret>"
```

---

# 🏷️ Recommended Production Labels

Apply labels to your production workflows to enrich incident context and improve routing.

## 📋 Example

```yaml
labels:
  app_name: payment-service
  environment: production
  business_impact: High
```

These labels are automatically propagated into:

* 📢 Slack alerts
* 🎫 ServiceNow incidents
* 📝 Operational logs
* 🚦 Severity classification

---

# 🚨 Supported Severity Levels

| Severity  | Action                                        |
| --------- | --------------------------------------------- |
| 🔴 High   | Slack Critical Alert + P1 ServiceNow Incident |
| 🟠 Medium | Slack Warning Alert + P2 ServiceNow Incident  |
| 🟢 Low    | Slack Informational Notification              |

---

# 👀 Observability Benefits

This architecture provides:

* 📡 centralized workflow monitoring
* ⚡ real-time operational visibility
* 🏢 enterprise incident automation
* 📏 standardized failure handling
* 🧾 audit-ready execution tracking
* ♻️ reusable orchestration patterns

---

# ✨ Features

## 🤖 Automated Workflow Failure Detection

Detects:

* ❌ FAILED
* ⚠️ WARNING
* 🛑 KILLED executions

Event-driven orchestration using Flow Triggers ⚡

---

## 💬 Real-Time Slack Notifications

Sends alerts to dedicated Slack channels 📢

Severity-based routing:

* 🔴 High
* 🟠 Medium
* 🟢 Low

Includes:

* 🆔 Flow ID
* 🔢 Execution ID
* 🗂️ Namespace
* 🌍 Environment
* ❌ Failed task
* 💬 Error message

---

## 🎫 ServiceNow Incident Creation

Automatically creates:

* 🚨 P1 incidents for High severity
* ⚠️ P2 incidents for Medium severity

Includes:

* 🔗 Correlation IDs
* 🏷️ Application metadata
* 🌍 Environment metadata
* 🔄 Retry count
* 📦 Payload snapshot

---

## 🧠 Dynamic Execution Context Capture

Captures:

* ❌ Failed task name
* 💬 Error message
* 📚 Stack trace
* 📦 Input payload
* 🔄 Retry attempts
* 🆔 Execution metadata

---

## 🛡️ Production-Ready Reliability

Includes:

* 🔁 Exponential retries
* ⏳ Retry backoff
* 📈 Retry max intervals
* ⌛ Timeout handling
* 🔐 Secret management
* 🧩 Failure isolation patterns

---

# 🏗️ Architecture

```text
Kestra Workflow Failure
        │
        ▼
Flow Trigger Listener
        │
        ▼
Context Enrichment
        │
 ┌──────┼──────┐
 ▼      ▼      ▼
Slack  SNOW   Logging
Alert Incident Audit
```

---

# 📁 Repository Structure

```text
.
├── flows/
│   ├── incident-management.yaml
│   └── sample-failing-flow.yaml
│
├── README.md
└── assets/
```

---

# ✅ Prerequisites

Before using this project, ensure you have:

* ⚙️ Kestra installed
* 💬 A Slack workspace
* 🎫 A ServiceNow instance
* 🔐 Kestra secrets configured

---

# 🚀 Installing Kestra

## 🐳 Docker Compose

```yaml
version: '3'

services:
  kestra:
    image: kestra/kestra:latest

    ports:
      - "8080:8080"

    command: server standalone
```

## ▶️ Start Kestra

```bash
docker compose up -d
```

## 🌐 Open

```text
http://localhost:8080
```

---

# 💬 Slack Webhook Configuration

## 1️⃣ Step 1 — Create a Slack App

Open:

`Slack API Apps Page`

Click:

* ➕ Create New App
* 🛠️ From Scratch

---

## 2️⃣ Step 2 — Enable Incoming Webhooks

Inside your Slack app:

Navigate to:

`Incoming Webhooks`

Enable:

✅ Activate Incoming Webhooks

---

## 3️⃣ Step 3 — Create Webhooks

Click:

`Add New Webhook to Workspace`

Select channels such as:

* 🚨 #critical-alerts
* 🧑‍💻 #dev-alerts
* 📢 #flow-notifications

Copy generated webhook URLs.

## 📌 Example

```text
https://hooks.slack.com/services/XXXX/YYYY/ZZZZ
```

---

# 🔐 Configure Slack Secrets in Kestra

Add secrets using Kestra CLI:

```bash
kestra secrets create SLACK_CRITICAL_WEBHOOK "https://hooks.slack.com/services/XXXX/YYYY/ZZZZ"

kestra secrets create SLACK_GENERAL_WEBHOOK "https://hooks.slack.com/services/AAAA/BBBB/CCCC"

kestra secrets create SLACK_INFO_WEBHOOK "https://hooks.slack.com/services/DDDD/EEEE/FFFF"
```

---

# 🎫 ServiceNow Configuration

## 📌 Required Credentials

You need:

* 🌐 ServiceNow domain
* 👤 Username
* 🔑 Password
* 🆔 Caller ID

---

## 🔐 Configure ServiceNow Secrets

```bash
kestra secrets create SNOW_DOMAIN "your-instance.service-now.com"

kestra secrets create SNOW_USERNAME "admin"

kestra secrets create SNOW_PASSWORD "password"

kestra secrets create SNOW_CALLER_ID "system"
```

---

# 📥 Importing the Workflow

## 🖥️ Option 1 — Kestra UI

Open Kestra UI

Navigate to:

`Flows`

Click:

* ➕ Create Flow
* 📋 Paste YAML
* 💾 Save

---

## 💻 Option 2 — CLI

```bash
kestra flow create flows/incident-management.yaml
```

---

# 💥 Sample Failing Workflow

The repository includes a sample workflow intentionally designed to fail.

It demonstrates:

* ⚡ Automatic trigger detection
* 📦 Error context propagation
* 💬 Slack alerting
* 🎫 ServiceNow incident creation

---

# 📦 Error Context Structure

## 📌 Example propagated payload

```json
{
  "message": "Unknown Error",
  "severity": "HIGH",
  "environment": "dev",
  "applicationName": "elephant-service",
  "executionId": "12345"
}
```

---

# 🚦 Severity Routing

| Severity  | Action                         |
| --------- | ------------------------------ |
| 🔴 High   | Slack + P1 ServiceNow Incident |
| 🟠 Medium | Slack + P2 ServiceNow Incident |
| 🟢 Low    | Slack Notification Only        |

---

# 🔁 Retry Strategy

All critical integrations include:

```yaml
retry:
  type: exponential
  interval: PT15S
  maxInterval: PT5M
  maxAttempt: 5
  warningOnRetry: true
```

---

# 🚀 Recommended Production Enhancements

Add:

* 🤖 AI-powered remediation suggestions
* 📊 Historical incident correlation
* 📟 PagerDuty integration
* 💬 Microsoft Teams integration
* 📡 OpenTelemetry tracing
* ☁️ S3/GCS archival
* 🛠️ Auto-remediation workflows

---

# 🔒 Security Best Practices

## ❌ Never Hardcode

* 🔑 passwords
* 🌐 webhook URLs
* 🎟️ tokens
* 🧠 API keys

✅ Always use Kestra Secrets.

---

# 🛠️ Troubleshooting

## 💬 Slack Notifications Not Working

Check:

* 🌐 Webhook URL validity
* 🔐 Slack app permissions
* 📢 Channel access

---

## 🎫 ServiceNow Incident Not Created

Verify:

* 👤 ServiceNow credentials
* 🗂️ Table permissions
* 🌐 Firewall/network rules

---

## ⚡ Trigger Not Firing

Ensure:

* 📂 Failed flow namespace matches trigger preconditions
* ❌ Execution state is FAILED/WARNING/KILLED

---

# 🤝 Contributing

Contributions are welcome. 🎉

Please open:

* 🐞 Issues
* 💡 Feature Requests
* 🔀 Pull Requests

---

# 📄 License

MIT License 📜

---

# 🙌 Acknowledgements

Built using:

* ⚙️ Kestra
* 💬 Slack
* 🎫 ServiceNow

---

# 👨‍💻 Author

Created by **Prajwal Prasanna** 🚀

⭐ If you found this project useful, consider starring the repository.

