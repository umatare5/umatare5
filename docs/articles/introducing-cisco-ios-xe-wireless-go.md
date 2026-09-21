# Introducing cisco-ios-xe-wireless-go: A Go SDK for Cisco Catalyst 9800

This article introduces **[umatare5/cisco-ios-xe-wireless-go](https://github.com/umatare5/cisco-ios-xe-wireless-go)**, an open-source Go SDK designed to operate Cisco Catalyst 9800 Series controllers.

## Background

A decade ago, monitoring AireOS wireless LAN controllers (WLCs) relied heavily on SSH. Like most operators, I used custom CLI tools to log in and parse text outputs.

Released around 2018, the Cisco Catalyst 9800 Series adopted standard IOS-XE software and with it programmable interfaces like RESTCONF. However, the C9800 RESTCONF implementation deviates significantly from standard REST conventions. The underlying YANG data structures and the complexity of 802.11 itself are the cause.

When operators build custom automation tools for this infrastructure, they encounter several implementation challenges:

- **Protocol Semantics**: IOS-XE RESTCONF enforces a strict separation between configuration data (`cfg`), operational data (`oper`) and remote procedure calls (`rpc`).
- **Performance Risks**: Every tree defined in the YANG model exposes an endpoint. Accidentally querying resource-intensive endpoints (e.g., `/restconf/data`) imposes severe loads on the device's management plane.
- **YANG Discrepancies**: The [official Cisco YANG models](https://github.com/YangModels/yang) serve as blueprints. Operators encounter unimplemented endpoints, missing arguments and type mismatches against real hardware.
- **Dynamic Response Structures**: Some endpoints stay hidden until the matching feature is enabled, and response fields vary with the parameters sent. The model tracks the device's configuration state, so clients must handle payloads flexibly.
- **PUT/POST Equivalence**: RPC endpoints (`rpc`) used for actions like restarting APs or controllers often yield identical results whether developers invoke them via PUT or POST, deviating from standard REST conventions.

I started this project to solve these challenges once, rather than troubleshooting them case by case.

## The cisco-ios-xe-wireless-go Approach

This SDK centralizes the RESTCONF code that downstream tools share. Rather than following the YANG definitions alone, it exposes the operations verified on real C9800 hardware.

Interacting with YANG structures and RESTCONF semantics demands intricate logic, and the SDK absorbs it. It handles automatic namespace switching, precise URI construction and endpoint-specific HTTP method selection. Dependent applications carry none of it.

## Actual Use Cases

Two tools run on this SDK in production. The exporter reads AP, client, WLAN and controller state on every scrape. The CLI queries several WLCs in parallel and prints the result as JSON.

- **[umatare5/cisco-wnc-exporter](https://github.com/umatare5/cisco-wnc-exporter)**: Prometheus Cisco WNC Exporter allows a Prometheus instance to monitor Catalyst 9800 wireless network metrics.
- **[umatare5/cisco-wnc-cli](https://github.com/umatare5/cisco-wnc-cli)**: A CLI for the Cisco Catalyst 9800 WLC (WNC), designed for easy operation and automation via RESTCONF.

## Usage in the AI Era

I maintain this SDK for **Knowledge Control**, which here means bounding what an agent may change rather than what it may read. AI agents refactor whatever sits in their context window, and logic that lives in the main application repository is in range of every feature change.

- **Knowledge Control**: A separate read-only dependency holds that logic out of range. The interactions verified against real hardware stay immutable, which preserves **reliability** for every tool built on it.
- **Drift Detection**: A library has no running configuration to drift against. The exporter built on it turns controller configuration into metrics, which is where the concern lives.

## Development Environment

If you want to build on this SDK yourself, here is the lab environment topology I use. I hope this serves as a helpful reference for setting up your own test environment.

```text
                    Internet
                       |
               [Juniper SRX300] Firewall
                       |
                [Cisco C891FJ] L3 Switch
                       |
       +---------------+---------------+
       |                               |
 [Ubuntu (qemu)]                [Cisco C2960CX] L2 Switch
       |                               |
 [C9800-CL WLC]           +------------+------------+
   - 17.12.x              |            |            |
   - 17.15.x         [AP1815I]    [AP2802I]    [CW9166I]
   - 17.18.x
   - 26.1.x
```
