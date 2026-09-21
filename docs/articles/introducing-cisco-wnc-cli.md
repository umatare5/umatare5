# Introducing cisco-wnc-cli: A RESTCONF-based CLI Tool for Cisco Catalyst 9800

This article introduces **[umatare5/cisco-wnc-cli](https://github.com/umatare5/cisco-wnc-cli) (wnc)**, a RESTCONF-based CLI designed to operate Cisco Catalyst 9800 Series controllers.

## Background

A decade ago, monitoring AireOS wireless LAN controllers (WLCs) relied heavily on SSH. Like most operators, I used custom CLI tools to log in and parse text outputs.

Released around 2018, the Cisco Catalyst 9800 Series adopted standard IOS-XE software and with it programmable interfaces like RESTCONF. The same shift made both layers strictly defined and highly complex, the underlying YANG entities and the standard CLI above them.

Cisco offers Catalyst Center as the official answer, a unified platform covering the entire network with automation and assurance. For a scope limited to wireless LAN operation, its deployment and operational requirements are more than the job needs.

Operators who skip that platform and manage the controllers directly meet the complexity head on:

- **Complex CLI**: The IOS-XE CLI follows RFCs and the underlying YANG entities strictly. One operational question takes several discrete commands, and the operator parses every output.
- **Complex GUI**: 802.11 is complex, so the GUI that covers it is comprehensive. That coverage buries the handful of views daily operation actually needs.
- **Multi-WLC Operations**: Load balancing often spreads access points across several WLCs. Operators reconcile that split by hand on every query.
- **SSH Overhead**: Opening a stateful SSH session for every operation is tedious, and it resists scripting.

I started this project to solve these challenges for wireless LAN operation alone, with an interface light enough to script.

## The cisco-wnc-cli Approach

**cisco-wnc-cli (wnc) is a command-line tool operating as a RESTCONF client**. Released under the MIT license, it reads Catalyst 9800 controllers and acts on them over that one protocol.

Three design decisions answer the obstacles above:

- **YANG Model Abstraction**: It holds the YANG structure internally and prints only what daily operation reads. Neither the CLI hierarchy nor the GUI menu tree is in the way.
- **Stateless Communication**: Each invocation is one RESTCONF exchange, with no session to open or tear down. A script calls it the way it calls any other command.
- **Multi-WLC Operations**: One command queries several WLCs concurrently, and every row carries the controller it came from. Where access points sit no longer changes the command.

Two costs come with those decisions. The CLI holds no state, so a JSON dump answers with the controller as it was when the dump ran. It reads one controller sequentially, so a joined view costs the sum of its requests and no management plane sees a burst.

## Actual Use Cases

The `show` commands read a controller and print one table per subject, from access points through clients to WLANs and tags.

The four below are the ones I run most often:

### Retrieve an overview from two controllers

```bash
wnc show overview --controller wnc1.example.internal --controller wnc2.example.internal --pretty
```

```text
┌───────────┬───────────────────┬──────┬─────────────┬──────┬─── ...
│  AP Name  │      AP MAC       │ Slot │    Mode     │ Band │ Ad ...
├───────────┼───────────────────┼──────┼─────────────┼──────┼─── ...
│ TEST-AP01 │ 00:00:5e:00:53:10 │ 0    │ FlexConnect │ 2.4  │ ✅ ...
│ TEST-AP01 │ 00:00:5e:00:53:10 │ 1    │ FlexConnect │ 5    │ ✅ ...
│ TEST-AP02 │ 00:00:5e:00:53:20 │ 0    │ FlexConnect │ 2.4  │ ✅ ...
│ TEST-AP02 │ 00:00:5e:00:53:20 │ 1    │ FlexConnect │ 5    │ ✅ ...
│ TEST-AP03 │ 00:00:5e:00:53:30 │ 0    │ FlexConnect │ 2.4  │ ✅ ...
│ TEST-AP03 │ 00:00:5e:00:53:30 │ 1    │ FlexConnect │ 5    │ ✅ ...
└───────────┴───────────────────┴──────┴─────────────┴──────┴─── ...
```

![Screenshot 2026-09-21 at 22.34.33.png](https://media.daily.dev/image/upload/s--_XWXTOaD--/f_auto/v1789997774/ugc/content_ab300fe1-1c66-4c9c-8447-4bc027f9fd51?_a=BAMAMicg0)

### Retrieve a list of access points

```bash
wnc show ap --controller wnc2.example.internal
```

```text
AP Name    Model             Serial       Ethernet MAC       Radio M...
TEST-AP01  AIR-AP1815I-Q-K9  FGL0000XXX1  00:00:5e:00:53:10  00:00:5...
TEST-AP03  CW9166I-Q         FGL0000XXX3  00:00:5e:00:53:30  00:00:5...
```

![Screenshot 2026-09-21 at 22.33.18.png](https://media.daily.dev/image/upload/s--iouuxJ5l--/f_auto/v1789997987/ugc/content_ae05115f-c702-45af-b332-9f088a5be3b5?_a=BAMAMicg0)

### Retrieve client information

```bash
wnc show client --controller wnc2.example.internal
```

```text
MAC                IPv4          Device                        SSID ...
00:00:5e:00:53:01  192.168.0.64  Unknown Device                labo-...
00:00:5e:00:53:02  192.168.0.90  Unknown Device                labo-...
00:00:5e:00:53:03  192.168.0.69  TP-LINK TECHNOLOGIES CO.,LTD. labo-...
00:00:5e:00:53:04  192.168.0.77  TP-LINK TECHNOLOGIES CO.,LTD. labo-...
00:00:5e:00:53:05  192.168.0.80  Unknown Device                labo-...
```

![Screenshot 2026-09-21 at 22.40.09.png](https://media.daily.dev/image/upload/s--iOedPYwG--/f_auto/v1789998060/ugc/content_9ef19df6-ca55-4eb5-91c6-1e07fbb39fa1?_a=BAMAMicg0)

### Retrieve WLAN configurations

```bash
wnc show wlan --controller wnc2.example.internal
```

```text
ID  Profile      SSID         Status   Security            Bands  Br...
5   labo-p736b2  labo-p736b2  Enabled  WPA2 PSK            2.4    En...
6   labo-p736b5  labo-p736b5  Enabled  WPA2 PSK            5      En...
7   labo-t6c73d  labo-t6c73d  Enabled  WPA3 802.1X-SHA256  5/6    En...
```

![Screenshot 2026-09-21 at 22.33.56.png](https://media.daily.dev/image/upload/s--_6w0Pe7q--/f_auto/v1789997922/ugc/content_285a56f4-62f6-4bfc-989f-a2980174f568?_a=BAMAMicg0)

Each answers one operational question in one invocation, with no CLI hierarchy to walk and no GUI menu to find.

## Usage in the AI Era

I run this CLI for **Knowledge Control** and **Drift Detection**, because an agent that asks those same questions never reaches the controller. One way I do that is a scheduled `wnc show wlan --format json` snapshot the agent reads instead. The schedule bounds what reaches the management plane, whatever the agent goes on to ask.

- **Knowledge Control**: `--format json` hands an agent a flat typed array where the controller offers raw YANG, which cuts **token cost**. The agent reads the snapshot and never holds `WNC_ACCESS_TOKEN`, which tightens **security**.
- **Drift Detection**: Successive snapshots differ only where something changed, so an agent's unexpected edit reads as a diff. Finding a disabled WLAN there rather than in a user report preserves **reliability**.

```bash
# Retrieve WLAN configurations as a minimized JSON array
❯ wnc show wlan --format json --insecure
warning: TLS certificate verification is disabled
[{"wlan_id":5,"profile":"labo-p736b2","ssid":"labo-p736b2","status":"Enabled","security":"WPA2 PSK","bands":"2.4","broadcast":"Enabled","p2p_block":"Disabled","policy_status":"Active","switching":"Local","interface":"LAB-INTERNAL","session_timeout_seconds":43200,"dhcp_required":true,"policy_profile":"labo-wlan-profile","tags":"labo-wlan-flex","controller":"wnc2.example.internal"},{"wlan_id":6,"profile":"labo-p736b5","ssid":"labo-p736b5","status":"Enabled","security":"WPA2 PSK","bands":"5","broadcast":"Enabled","p2p_block":"Disabled","policy_status":"Active","switching":"Local","interface":"LAB-INTERNAL","session_timeout_seconds":43200,"dhcp_required":true,"policy_profile":"labo-wlan-profile","tags":"labo-wlan-flex","controller":"wnc2.example.internal"}]
```

## Development Environment

If you want to test the tool yourself, here is the lab environment topology I use. I hope this serves as a helpful reference for setting up your own test environment.

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
