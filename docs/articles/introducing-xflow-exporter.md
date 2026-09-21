# Introducing xflow-exporter: A Lightweight Flow Analytics Exporter for Enterprise Networks

This article introduces **[umatare5/xflow-exporter](https://github.com/umatare5/xflow-exporter)**, a Prometheus exporter designed to turn flow datagrams into metrics without an analytics backend.

## Background

A decade ago, analyzing network traffic meant assembling Fluentd, Norikra, Elasticsearch and Kibana into one system. Prometheus and Kafka were both new then, so the choices were few.

Several outstanding open-source projects answer this today, among them **[Akvorado](https://github.com/akvorado/akvorado)** and **[pmacct](https://github.com/pmacct/pmacct)**.

- **Akvorado is a phenomenal modern tool that gives deep network visibility**. However, strict enterprise production environments often put its AGPLv3 license through a compliance review first.
- **pmacct is a highly mature and meticulously crafted project**. Its modular design carries everything from a small deployment to a service provider network. However, that reach asks for several processes and a backend like Kafka behind them. Each is one more thing to run and be paged for.

Either way the precondition is a license review to clear or a stack of processes to run. Enterprise networks and small-to-medium data centers with neither still had no flow analytics light enough to deploy on their own. So I started this project to solve these challenges with something rougher and simpler.

## The xflow-exporter Approach

Both projects are the work of people who have been at this far longer than I have. Where the analysis needs more than a trend, they are what I would reach for. I offer xflow-exporter as one alternative for the trend itself.

xflow-exporter is a **Prometheus exporter that receives flow datagrams and publishes aggregates**. One MIT-licensed binary decodes NetFlow, IPFIX and sFlow. The same binary collects, enriches and aggregates them.

PromQL and Alertmanager already do the querying and the alerting, so the exporter keeps its own responsibilities small. It publishes aggregates like traffic per TCP flag or DSCP value, and the query does the rest.

Compared with a dedicated analytics backend, the design accepts three constraints:

- **Static Aggregation**: Each collector aggregates on one fixed set of dimensions, so the flexible multi-dimensional analysis a Kafka-backed platform supports is out of reach. That rules out large-scale traffic analysis.
- **Precision Loss**: `--aggregation.top-k` publishes 1000 entries per table by default and withholds the tail entirely. A sum over a table ranks its busiest entries, so the grain is too coarse for security forensics.
- **Scalability Limits**: Every map keyed by wire data takes a bound, 65536 devices and 256 observation domains on each. [Bounded State](https://github.com/umatare5/xflow-exporter/blob/main/docs/architecture.md#bounded-state) lists the rest with the constant behind each one.

## Actual Use Cases

The dashboard below is the one from [examples/](https://github.com/umatare5/xflow-exporter/blob/main/examples/grafana_xflow-exporter-dashboard.json), imported as it ships.

![xflow-exporter Grafana dashboard](https://media.daily.dev/image/upload/s--hTfScCi7--/f_auto/v1790021969/ugc/content_7643e3ff-299b-4d2f-a615-14fdf7904903?_a=BAMAMicg0)

Ephemeral ports drive the cardinality in flow data, so a reply leg folds onto the service's port rather than taking an entry under a client's number. Host pairs come next, and `--aggregation.top-k` caps each table at its busiest entries. Even so, I split the TSDB into a short-term role and a long-term one.

- **Short-Term Metrics (Monitoring)**: Keep every published series here at full resolution, where the dashboards and the alert rules read them.
- **Long-Term Metrics (Capacity Planning)**: Aggregate with recording rules on the short-term TSDB and forward only the summarized series. [`examples/prometheus_record_rules.yml`](https://github.com/umatare5/xflow-exporter/blob/main/examples/prometheus_record_rules.yml) carries the ones I run.

Filter explicitly on both `remote_write` and `remote_read`. Prometheus 3.14 takes these two forms:

For `remote_write`, use `write_relabel_configs` to drop what long-term storage does not need:

```yaml
remote_write:
  - url: "https://your-remote-storage-endpoint/api/v1/write"
    write_relabel_configs:
      # Drop the per-pair tables and forward the recording rules instead
      - source_labels: [__name__]
        regex: "xflow_(host_pair|service|destination)_(bytes|packets|flows)_total"
        action: drop
```

For `remote_read`, use `required_matchers` to select the label your recording rules attach, so only queries for precomputed series reach remote storage:

```yaml
remote_read:
  - url: "http://your-remote-storage-provider:8080/api/v1/read"
    required_matchers:
      precompute: "true"
```

## Usage in the AI Era

I run this exporter for **Knowledge Control**, because an agent acting on a network needs to know how traffic is trending. Devices export at their own sampler rate whether or not anyone is asking, so no query an agent writes runs back to one.

- **Knowledge Control**: The per-pair series carry real cardinality, so a short-retention TSDB drops them early. An agent reads `xflow_host_pair_bytes_total` there and holds no device credential, which cuts **token cost** and tightens **security**.
- **Drift Detection**: A flow is an observation rather than a configuration, so no intended state exists to compare a reading against. The exporters that read configuration are where that concern lives.

## Development Environment

If you want to test the exporter yourself, here is the lab environment topology I use. I hope this serves as a helpful reference for setting up your own test environment.

```text
                    Internet
                        |
                [Juniper SRX300] Firewall
                        |
                 [Cisco C891FJ] L3 Switch
                        |
        +---------------+------------------+
        |               |                  |
    [HP 2530]    [Cisco C9800-CL]   [Cisco C2960CX] L2 Switch
    L2 Switch          WLC               |   |
                                       [AP] [AP]
```
