# Alertmanager & PagerDuty Integration Report

## 0. Prerequisites

.env file with (for PagerDuty integration)
- PAGERDUTY_SERVICE_KEY
- PAGERDUTY_URL
- GRAFANA_ADMIN_PASSWORD (or default to admin/admin)

## 1. Overview
end-to-end setup 
- enables **app (whoknows)** to trigger **PagerDuty** incidents 
    - conditions: 
        - HTTP500 errors are detected.  (for testing of Business value metric)
    - latency: within ~14s from alert to incident


| Containers        | Role |
|------------------|------|
| **whoknows_flask** | Produces application metrics and exposes `/metrics` for scraping |
| **Prometheus**   | Scrapes metrics, evaluates alert rules |
| **Alertmanager-1** | Receives alerts from Prometheus and dispatches notifications to PagerDuty |
| **Grafana**      | Visualises real-time metrics and alert history |


## toc:
- [Overview](#overview)
- [Prometheus ➜ Alertmanager wiring](#prometheus-alertmanager-wiring)
- [Alertmanager ➜ PagerDuty integration](#alertmanager-pagerduty-integration)
- [Business-value metric: `whoknows_http_responses_total`](#business-value-metric-whoknows_http_responses_total)
- [End-to-end validation](#end-to-end-validation)
- [Troubleshooting tips](#troubleshooting-tips)
- [Next steps](#next-steps)

---

## 2. Prometheus ➜ Alertmanager wiring
1. Prometheus loads `prometheus.rules.yml`, which contains the custom alert definitions.
2. When a rule fires, Prometheus POSTs the alert payload to both Alertmanager instances (`9093` and `9094`).
3. Alertmanager groups, deduplicates, and forwards notifications to the configured receiver (`oncall`).

### Key rule definitions
| Alert | Expression | Purpose |
|-------|------------|---------|
| **AppDown** | `up{job="whoknows-app"} == 0` | Detects container outage |
| **HTTP500** | `rate(whoknows_http_responses_total{code="500"}[1m]) > 0` | Detects runtime errors (see §4) |

---

## 3. Alertmanager ➜ PagerDuty integration
`src/alertmanager.yml` defines the `pagerduty_configs` block referencing environment variables supplied via `.env`:

```
receivers:
  - name: oncall
    pagerduty_configs:
      - service_key: ${PAGERDUTY_SERVICE_KEY}
        url: ${PAGERDUTY_URL}
        send_resolved: true
```

* **PAGERDUTY_SERVICE_KEY** – Integration key for the *whoknows* service ([link](https://caesari.pagerduty.com/service-directory/PZB6AB0))
* **PAGERDUTY_URL** – PagerDuty REST endpoint

With these variables injected, Alertmanager raises a PagerDuty incident whenever an alert is in the *firing* state. Example incident triggered during test: <https://caesari.pagerduty.com/incidents/Q25G0EDMZT0KJQ>

---

## 4. Business-value metric: `whoknows_http_responses_total`
The application instrumented the following Prometheus counter in `src/backend/app.py`:

```
RESPONSE_COUNTER = Counter(
    "whoknows_http_responses_total",
    "Count of HTTP responses labeled by status code.",
    ["code"]
)
```

Every HTTP response increments `RESPONSE_COUNTER{code="<status>"}`. This unlocks powerful alerting:

* **Immediate visibility** into 5xx spikes, which correlate strongly with customer-visible failures.
* **Reduced MTTD**: Aggregated error rates trip the **HTTP500** alert in under a minute.
* **Actionable context**: Status-code labels help engineering pinpoint regressions to specific endpoints.

Business impact: catching 500-level errors in real-time prevents revenue loss and drives SLA adherence.

---

## 5. End-to-end validation
1. Run `make docker-run` to start the stack.
2. Execute `make unhappy` – this issues a request designed to return a **500**.
3. Observe:
   * Prometheus rule **HTTP500** transitions to *firing*.
   * Alertmanager sends a **trigger** event to PagerDuty.
   * Incident appears on the *whoknows* service dashboard.
4. Restore healthy behaviour and verify the incident auto-resolves once errors cease.

---

## 6. Troubleshooting tips
| Symptom | Likely Cause | Resolution |
|---------|--------------|-------------|
| No metrics in Prometheus | App not scraped | Verify `/metrics` is reachable from Prometheus network |
| Alert doesn't fire | Rule misconfigured | Check expression syntax & scrape interval |
| PagerDuty not notified | Env vars missing | Confirm `PAGERDUTY_SERVICE_KEY` & `PAGERDUTY_URL` in `.env` |

---

## 7. Next steps
* Add dashboards in Grafana to visualise `whoknows_http_responses_total` by status code.
* Introduce **deployment-safety** alerts (e.g., error budget burn rate).
* Automate secret rotation for PagerDuty integration keys.
