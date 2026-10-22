# HPC Monitoring and Observability

## Overview

This component documents the monitoring and observability stack for a virtualized HPC cluster running Rocky Linux 9.8.

The monitoring design combines infrastructure metrics, centralized log collection, and dashboards to help administrators inspect system health and troubleshoot services.

## Architecture

```text
                  HPC Infrastructure
            +-----------+-------------+
            |           |             |
         Master 1    Master 2     Compute Nodes
            |           |             |
            +-----------+-------------+
                        |
               +--------+--------+
               |                 |
            Metrics             Logs
               |                 |
          Prometheus          Grafana Alloy
               |                 |
               |                Loki
               |                 |
               +--------+--------+
                        |
                     Grafana
                        |
                HPC Dashboards
```

This is a logical architecture. Verify which hosts run each service, which exporters are enabled, and which data sources are connected before treating the diagram as a deployment record.

## Components

| Component | Responsibility |
|---|---|
| Prometheus | Scrapes and stores metrics from configured targets |
| Grafana Alloy | Collects and forwards configured telemetry, including logs |
| Loki | Stores and indexes log metadata for querying |
| Grafana | Visualizes metrics and logs through dashboards |
| Exporters and application endpoints | Expose metrics for Prometheus, where configured |

## Metrics Workflow

1. A monitored service or exporter exposes metrics.
2. Prometheus scrapes configured targets at the defined intervals.
3. Metrics are stored as time-series data.
4. Grafana queries Prometheus through a configured data source.
5. Dashboards visualize available infrastructure and service metrics.

Potential HPC metrics include CPU utilization, memory usage, disk activity, network throughput, and service availability. Only metrics exposed by configured collectors can be displayed.

## Log Workflow

1. System and application logs are generated on monitored hosts.
2. Alloy reads the configured log sources.
3. Alloy forwards log entries to Loki.
4. Loki stores the logs and makes them queryable.
5. Grafana uses its Loki data source to explore and display log entries.

Log collection depends on correct file paths, permissions, Alloy configuration, and network connectivity to Loki.

## Validation

### 1. Check service status

On the host running each service:

```bash
systemctl status prometheus
systemctl status alloy
systemctl status loki
systemctl status grafana-server
```

Service names can vary by installation method. Run only the commands corresponding to services installed on that host.

### 2. Check Prometheus targets

Open the Prometheus web interface and inspect its **Targets** page. Confirm that the expected targets are being scraped successfully.

A configured target is not necessarily a healthy target; check its reported scrape state.

### 3. Check Loki readiness

If Loki's HTTP endpoint is configured on port 3100 and reachable locally:

```bash
curl -sS http://localhost:3100/ready
```

A successful readiness response indicates that Loki is ready to serve requests. It does not prove that logs are arriving.

### 4. Check Alloy

Inspect the service logs:

```bash
journalctl -u alloy -n 100 --no-pager
```

Review Alloy's configuration and logs for file-access failures, rejected entries, and delivery errors.

### 5. Validate Grafana

In Grafana:

1. Open **Connections → Data sources**.
2. Verify the Prometheus data source.
3. Verify the Loki data source.
4. Test each data source.
5. Open the relevant dashboard or Explore view.
6. Confirm that actual metrics and logs are returned.

Menu names may differ slightly between Grafana versions.

These filenames are examples. Use screenshots that exist and accurately demonstrate the described functionality.

## Validation Checklist

- [ ] Prometheus starts successfully.
- [ ] Expected monitoring targets are configured.
- [ ] Prometheus targets return metrics successfully.
- [ ] Alloy starts successfully.
- [ ] Alloy can read the intended log sources.
- [ ] Loki is ready and accepts log entries.
- [ ] Logs from a test source are queryable in Loki.
- [ ] Grafana data sources connect successfully.
- [ ] Dashboard panels display actual data.
- [ ] Relevant Slurm and compute-node services are monitored.

## Security Considerations

- Restrict access to Grafana, Prometheus, Loki, and Alloy management endpoints.
- Avoid exposing internal monitoring ports directly to untrusted networks.
- Do not commit passwords, API tokens, or sensitive configuration.
- Apply appropriate permissions to system logs.
- Redact usernames, internal addresses, and sensitive log content before publishing screenshots.

## Current Status

Document each component separately as configured, operational, validated, or pending.

For example, a running Loki service does not prove that Alloy is successfully forwarding logs. A dashboard opening successfully does not prove that every panel has valid data.

## Future Improvements

- Add Linux node exporters to all relevant hosts.
- Monitor Slurm controller and compute-node health.
- Create alerts for node failures, resource pressure, and service outages.
- Build dashboards for job counts, node states, and resource utilization.
- Document retention, storage capacity, and backup policies.
- Validate the complete telemetry path with a known test metric and log entry.
