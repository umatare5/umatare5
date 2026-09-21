# Introducing controld-exporter: A Prometheus Exporter for Control D

This article introduces **[umatare5/controld-exporter](https://github.com/umatare5/controld-exporter)**, a REST API client Prometheus exporter designed to observe Control D.

## Background

Google Public DNS arrived in 2009, making public resolvers the default. That traffic crossed the internet as plaintext, leaving it open to interception and forgery.

Cloud-managed DNS services answered with DoH, DoT, and DoQ. Encrypted transport closed that exposure. Malware protection followed, turning resolvers into filtering points rather than just transport mechanisms.

Several outstanding encryped malware protection resolvers available today, among them **[NextDNS](https://nextdns.io/)** and **[Cloudflare DNS](https://www.cloudflare.com/learning/dns/dns-over-tls/)**:

- **NextDNS is a well-known name in the encrypted malware protection DNS domain**. However, being based in Okinawa, every hop leaves the island over submarine cables. The resolution latency I measured was higher than acceptable.
- **Cloudflare DNS 1.1.1.2 requires the least setup of any encrypted resolver I have used**. However, it is fully managed by design, and its filtering rules are not customizable.

Compared with those two, [Control D](https://controld.com) answered both at a price I was willing to carry. It held the latency down and left the filtering rules mine to set, so it is the resolver I have relied on ever since.

Control D puts custom malware protection behind a paid plan and its bill follows the scale of use. Its GUI is also where an operator changes something without meaning to. So I started this project to solve these challenges, and to keep both the bill and the configuration in Prometheus.

## The controld-exporter Approach

controld-exporter is a **Prometheus exporter operating as a REST API client**. One MIT-licensed binary reads health, configuration and billing from the Control D API.

Of those three, the `controld_profile_*` series count what a profile has configured rather than what it matched. They move when someone edits a profile and stay flat under any amount of traffic, which is what makes a diff on them mean something.

## Actual Use Cases

The dashboard below is the one from [examples/](https://github.com/umatare5/controld-exporter/blob/main/examples/control-d-exporter-dashboard.json), imported as it ships.

![Control D exporter dashboard](https://media.daily.dev/image/upload/s--sCVCxD-p--/f_auto/v1790029230/ugc/content_16f6e27a-6ef7-49c7-a826-5fef0217340d?_a=BAMAMicg0)

The alerting rules ship beside it. This one watches the points of presence I actually resolve against, and a node reporting `0` has stopped answering while the account itself stays healthy:

```yaml
- alert: NetworkServiceDown
  expr: controld_network_health_code{country_name="JP"} == 0
```

Billing gets the same treatment. `controld_billing_subscription_amount_total` carries the amount per currency, so a threshold just above my plan catches a change before the statement does:

```yaml
- alert: BillingAmountHigh
  expr: controld_billing_subscription_amount_total{currency="USD"} > 2
```

## Usage in the AI Era

I run this exporter for **Knowledge Control** and **Drift Detection**, because an agent working on my DNS must not hold the key to it. The exporter keeps that key and the agent reads Prometheus. Only the scrape reaches the Control D API, whatever the agent goes on to ask.

- **Knowledge Control**: A SaaS platform authenticates with an API key, so giving an agent the knowledge usually means giving it the key. It reads the series and never `CTRLD_API_KEY`, which cuts **token cost** and tightens **security**.
- **Drift Detection**: The `controld_profile_*` series move only when a profile changes, so a filter an agent disabled shows up as a count falling. Catching it there rather than in a resolution failure preserves **reliability**.

## Development Environment

If you want to test the exporter yourself, a Control D trial account and an API key are all it takes to start. Control D is a subscription service, so the token has to come from a paid account once the trial ends. The [Control D documentation](https://docs.controld.com/docs/) carries the plans and their limits.
