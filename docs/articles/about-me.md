# Profile - About

I'm a network-focused SRE working in Japan.

I have more than 15 years of industry experience, especially with in-depth knowledge of networking including wireless. I'm good at real-time processing like packet analytics, metrics monitoring and others. I'm also a backend engineer using Go and TypeScript.

I maintain several Prometheus exporters and CLI tools within my areas of expertise. I rely on **daily.dev** to record the problem each tool solves, the approach I chose, the actual use cases, how I use it in the AI era and the development environment.

In the AI era, I run these tools for two purposes, **Knowledge Control** and **Drift Detection**.

- **Knowledge Control**: Keeping upstream state as Prometheus series and JSON files enables least privilege, cuts **token cost** and protects **upstream performance**. It also tightens **security** by hiding the upstream credentials from agents.
- **Drift Detection**: Scraping configuration as metrics lets my existing alert rules detect an agent's unexpected change, such as a disabled service. They preserve **reliability** by warning early of degradation or an outage.

As of September 2026, the following repositories are available:

- **[umatare5/twelvedata-exporter](https://github.com/umatare5/twelvedata-exporter):** Prometheus Twelvedata Exporter allows a Prometheus instance to monitor prices of stocks, ETFs and mutual funds. See **[introducing article](https://daily.dev/posts/introducing-twelvedata-exporter-a-prometheus-exporter-for-twelve-data-qgqjetz42)**.

- **[umatare5/controld-exporter](https://github.com/umatare5/controld-exporter):** Prometheus Control D Exporter allows a Prometheus instance to monitor metrics endpoints and subscriptions. See **[introducing article](https://daily.dev/posts/introducing-controld-exporter-a-prometheus-exporter-for-control-d-xj5vymsb5)**.

- **[umatare5/xflow-exporter](https://github.com/umatare5/xflow-exporter):** Prometheus xflow Exporter allows a Prometheus instance to monitor network traffic flows via NetFlow, IPFIX and sFlow. See **[introducing article](https://daily.dev/posts/introducing-xflow-exporter-a-lightweight-flow-analytics-exporter-for-enterprise-networks-5nln0plyj).**

- **[umatare5/cisco-wnc-exporter](https://github.com/umatare5/cisco-wnc-exporter):** Prometheus Cisco WNC Exporter allows a Prometheus instance to monitor Catalyst 9800 wireless network metrics. See **[introducing article](https://daily.dev/posts/introducing-cisco-wnc-exporter-a-lightweight-wireless-telemetry-exporter-for-cisco-catalyst-9800-0l8bhmnpo).**

- **[umatare5/cisco-wnc-cli](https://github.com/umatare5/cisco-wnc-cli):** A CLI for the Cisco Catalyst 9800 WLC (WNC), designed for easy operation and automation via RESTCONF. See **[introducing article](https://daily.dev/posts/introducing-cisco-wnc-cli-a-restconf-based-cli-tool-for-cisco-catalyst-9800-k9avombfh)**.

- **[umatare5/cisco-ios-xe-wireless-go](https://github.com/umatare5/cisco-ios-xe-wireless-go):** The Go SDK for building applications and automation tools for Cisco Catalyst 9800 Wireless Controller. See **[introducing article](https://daily.dev/posts/introducing-cisco-ios-xe-wireless-go-a-go-sdk-for-cisco-catalyst-9800-gb6l92qid).**

## Work Experience

- **Company #4:**
  - Infrastructure Engineer (2023/1-)
  - Software Engineer (2022/2 -)
- **Company #3:**
  - Infrastructure Engineer (2014/7 - 2022/1)
- **Company #2:**
  - Lead Infrastructure Engineer (2013/7 - 2014/4)
  - Infrastructure Engineer (2010/7 - 2013/3)
- **Company #1:**
  - Operator (2008/7 - 2010/6)
  - Intern (2008/4 - 2008/6)

- **Second / Part-Time**
  - Infrastructure and Software Engineer (2021/12 -)

## Technical Skills

### Public Cloud / SaaS (2020-)

- **Compute:** AWS EC2, Google Compute Engine
- **Container:** AWS ECS, AWS Fargate, Google Cloud Run
- **DNS:** AWS Route53, Google Cloud DNS
- **Mail:** AWS SES
- **Load Balancing:** AWS ALB/NLB, Google Cloud Load Balancing
- **Authentication:** GCP Identity-Aware Proxy, GCP Workload Identity, Auth0
- **Log Management:** AWS CloudWatch Logs, Google Cloud Logging
- **Monitoring:** AWS CloudWatch, Google Cloud Monitoring, Datadog
- **Configuration Management:** CloudFormation, Terraform
- **CI/CD:** Github Actions
- **Batch:** AWS Lambda, Google Cloud Functions, Google Apps Script
- **API:** Google Cloud Endpoints
- **BackOffice:** Kintone, ServiceNow ITSM

### Linux (2012-)

- **DNS:** BIND
- **DHCP:** ISC DHCP
- **Web:** Apache, Nginx, Squid
- **Mail:** Postfix, Sendmail, Dovecot
- **FTP:** vsftpd
- **Load Balancing:** Heartbeat, LTM
- **Directory Service:** OpenLDAP
- **Authentication:** FreeRADIUS
- **Realtime Processing:** Fluentd, Norikra
- **Log Management:** Rsyslog, swatch
- **Monitoring:** Nagios, Prometheus
- **Observability:** Cacti, Growthforecast, Kibana, Grafana
- **Configuration Management:** Chef, Ansible
- **NoSQL:** MongoDB
- **Timeseries:** RRDtool, Elasticsearch, InfluxDB, Cortex, Mimir
- **CI/CD:** Jenkins, Drone

### Networking (2012-2020, 2022-)

- **Load Balancing:** F5
- **Routing and Switching:** Cisco
- **Wireless:** Cisco, Meraki, Aruba
- **Security:** Cisco, Juniper Networks

### Storage (2012-2014)

- **SAS:** MSA
- **iSCSI:** Lefthand
- **FC:** NetApp

### Virtualization (2010-2022)

- **Hypervisor:** KVM, vSphere, Hyper-V
- **Security:** vCenter SSO
- **Backup & Recovery:** vCenter DataRecovery
- **Desktop:** Horizon View
- **Application:** Docker Swarm
- **Others:** vCenter Server, vCenter Converter

### Windows (2008-2014)

- **DNS:** Microsoft DNS
- **DHCP:** Microsoft DHCP
- **Web:** IIS
- **Mail:** IIS SMTP
- **FTP:** IIS FTP
- **Load Balancing:** NLB, MSFC
- **Directory Service:** Active Directory
- **Authentication:** ADCS, NPS
- **Security:** WSUS
- **Backup & Recovery:** BackupExec
- **Batch:** JP1/AJS

### Programming (2008-)

- **Powershell:** 1 to 2 (2008-2014)
- **ASP.NET and C#.NET:** 2.0 to 3.5 (2012-2014)
- **Ruby:** 2.3 to 2.7 (2016-2020)
- **TypeScript:** 2.2 to latest (2016-)
- **Go:** 1.15 to latest (2020-)

In the AI era, as of September 2026, I rely on **Anthropic Claude** for all my coding.

### Others (2008-)

- **Documentation:** Lotus Notes, Word, Confluence, Notion, Google Docs
- **Drawing:** Visio, PowerPoint, Cacoo, Google Slide, draw.io, Lucidchart

In the AI era, as of September 2026, I also rely on **Google Gemini** for all my documentation.

## Qualifications

- Amateur Third-Class Radio Operator
- Maritime Ⅱ-Category Special Radio Operator
- On-The-Ground Ⅲ-Category Special Radio Operator
