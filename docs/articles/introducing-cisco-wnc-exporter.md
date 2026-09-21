# Introducing cisco-wnc-exporter: A Lightweight Wireless Telemetry Exporter for Cisco Catalyst 9800

This article introduces **[umatare5/cisco-wnc-exporter](https://github.com/umatare5/cisco-wnc-exporter)**, a Prometheus exporter designed to monitor Cisco Catalyst 9800 controllers over RESTCONF rather than streaming telemetry.

## Background

A decade ago, monitoring AireOS wireless LAN controllers (WLCs) relied heavily on SNMP. The protocol's own constraints kept it from exporting granular client metrics efficiently.

As a workaround, Cisco offered appliances like WCS and Prime Infrastructure (PI). They scraped the controllers over SSH and web logins, then mapped and reported on what they gathered.

Released around 2018, the Cisco Catalyst 9800 Series left the AireOS architecture behind and adopted Cisco's standard IOS-XE software. That shift opened options the SNMP era never had.

IOS-XE brings Model-Driven Telemetry (MDT) and gRPC Streaming Telemetry with it. Cisco's own answers to wireless observability are **[Cisco Catalyst Center](https://www.cisco.com/site/us/en/index.html)** and **[Meraki Cloud Monitoring for Wireless](https://documentation.meraki.com/Wireless/Cloud-Managed_Hybrid_Operating_Mode_for_Catalyst_Wireless_LAN_Controllers)**.

- **Cisco Catalyst Center covers the entire network, a unified platform with automation and assurance**. For a scope limited to wireless LAN observability, its deployment and operational requirements are more than the job needs.
- **Meraki Cloud Monitoring for Wireless brings the appliance into the Meraki dashboard**. It has no architectural support for C9800-CL, the virtual edition this exporter runs against.

Either way the precondition is Catalyst Center or a hardware appliance. Small-to-medium enterprise networks with neither still had no wireless observability tool they could deploy on their own. So I started this project to solve these challenges at a cost those networks can carry.

## The cisco-wnc-exporter Approach

Where the job spans the wired network as well as wireless, Catalyst Center is the platform I would still choose. I offer cisco-wnc-exporter as one alternative for wireless alone.

cisco-wnc-exporter is a **Prometheus exporter built only for the Cisco Catalyst 9800 wireless LAN controller**. Released under the MIT license, one binary reads AP, client, WLAN and controller metrics.

The exporter carries one collector per entity, and each reads only the YANG endpoints its own metrics need. Nothing walks `/restconf/data` itself, which is what keeps the load on the controller down.

Compared with push-based streaming telemetry like MDT, the design accepts three constraints:

- **Polling Model**: The exporter reads at an interval. A client roam that starts and ends between two refreshes never appears at all.
- **HTTPS Listening**: The WLC has to listen on an HTTPS port for RESTCONF. Put that port behind a separate management interface or an ACL, because RESTCONF reaches whatever the account behind it can reach.
- **Controller Load**: A scrape never reaches the controller, so `--wnc.cache-ttl` and not `scrape_interval` sets how often the exporter reads the management plane. At the 55-second default, no dashboard refresh can lower that floor.

## Actual Use Cases

The dashboard below is the user dashboard from [examples/](https://github.com/umatare5/cisco-wnc-exporter/blob/main/examples/grafana_cisco-wnc-exporter-user-dashboard.json), imported as it ships.

![content_dda891d2-3f0b-44b1-b81d-3ff436645319.webp](https://media.daily.dev/image/upload/s--5492rJL3--/f_auto/v1790019281/ugc/content_385af417-8982-4339-94c4-c6ebbfdcd857?_a=BAMAMicg0)

Client MAC addresses drive the cardinality here, and every other client series joins back to `wnc_client_info` through its `mac` label. So I split the TSDB into a short-term role and a long-term one.

- **Short-Term Metrics (Monitoring)**: Keep every per-client series here at full resolution, where the dashboards and the alert rules read them.
- **Long-Term Metrics (Capacity Planning)**: Aggregate with recording rules on the short-term TSDB and forward only the summarized series. The long-term store never sees the per-client cardinality that made the aggregation necessary.

Filter explicitly on both `remote_write` and `remote_read`. Prometheus 3.14 takes these two forms:

For `remote_write`, use `write_relabel_configs` to drop what long-term storage does not need:

```yaml
remote_write:
  - url: "https://your-remote-storage-endpoint/api/v1/write"
    write_relabel_configs:
      # Drop the per-client error counters and forward a recording rule instead
      - source_labels: [__name__]
        regex: "wnc_client_(mic_mismatch|mic_missing|rts_retries)_total"
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

I run this exporter for **Knowledge Control** and **Drift Detection**, because an agent triaging Wi-Fi needs RF state it cannot see and I need to know what it changed. Both read from the TSDB and never from the controller. Only the refresh cadence reaches the management plane, which protects **upstream performance** whatever either of us asks.

- **Knowledge Control**: `wnc_client_info` mints a series per address change, so a short-retention TSDB drops them early. An agent reads them there and never on the controller, which cuts **token cost** and tightens **security**.
- **Drift Detection**: The WLAN collector reports the configuration each WLAN resolves to, so `wnc_wlan_enabled` falling to 0 is an alert rather than a user report. Catching an agent's change there preserves **reliability**.

## Development Environment

If you want to test the exporter yourself, here is the lab environment topology I use. I hope this serves as a helpful reference for setting up your own test environment.

```text
                    Internet
                       |
               [Juniper SRX300] Firewall
                       |
                [Cisco C891FJ] L3 Switch
                       |
       +---------------+---------------+
       |                               |
 [Ubuntu (qemu)]                 [Cisco C2960CX] L2 Switch
       |                               |
 [C9800-CL WLC]           +------------+------------+
   - 17.12.x              |            |            |
   - 17.15.x         [AP1815I]    [AP2802I]    [CW9166I]
   - 17.18.x
   - 26.1.x
```
