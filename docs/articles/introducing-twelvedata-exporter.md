# Introducing twelvedata-exporter: A Prometheus Exporter for Twelve Data

This article introduces **[umatare5/twelvedata-exporter](https://github.com/umatare5/twelvedata-exporter)**, a REST API client Prometheus exporter designed to fetch financial metrics via Twelve Data.

## Background

I came to see stock price trends as infrastructure metrics through years of metrics analysis experience. They are continuous numeric time series, so a Prometheus exporter can collect them and PromQL can analyze them.

Open-source projects answered this before mine, among them **[tcolgate/yquotes-exporter](https://github.com/tcolgate/yquotes_exporter)** and **[marcopaganini/quotes-exporter](https://github.com/marcopaganini/quotes-exporter)**.

- **yquotes-exporter was an early project** that showed how well Yahoo Finance data carries into Prometheus.
- **quotes-exporter was a subsequent implementation**. I relied on it heavily and took its architecture as the direct reference for this project.

Both were pioneering work, and both read from the Yahoo Finance API. Yahoo then tightened its rate limits and moved endpoints, which left the two tools hard to run steadily.

I went looking for another platform and found [Twelve Data](https://twelvedata.com). It needs an API key and its free plan restricts what I can reach, but the service model covered what I needed for US equities and indices.

So I started this project to automate my trading at no infrastructure cost. Each trading decision takes seconds and follows indicators rather than my reaction to price movements.

## The twelvedata-exporter Approach

twelvedata-exporter is a **Prometheus exporter operating as a REST API client**. One MIT-licensed binary fetches quotes and publishes them as metrics.

Once prices are metrics, stock analysis and alerting belong to the Prometheus ecosystem. Most common technical analysis reproduces in PromQL. RSI, MACD and moving averages all come out of the query language alone.

**Acknowledgments**: This exporter takes its shape from quotes-exporter and yquotes-exporter. I am deeply grateful to Marco and Tristan, who built and maintained those projects themselves. Mine would not exist without theirs.

## Actual Use Cases

I run the free plan and scrape 8 markets and 4 individual stocks at 15-minute intervals.

A Grafana dashboard showing only the indicators and never the raw price is what keeps my trading systematic. It sits on an iPad in the hallway I pass every morning, and the color alone says whether a trade is due. On a day that needs none, the whole reading costs about 5 seconds.

One symbol is one `/quote` call to Twelve Data, so the daily count over the 6.5-hour trading window comes to this:

```text
Total Credits = 12 (Symbols) × 26 (Scrapes) = 312
```

A 15-minute interval is not the fastest the plan allows. I chose the slowest one that still meets my requirement, so Twelve Data serves nothing I do not use.

The free plan sets three boundaries:

- **Credit Cost**: One symbol costs one credit per scrape. The symbol count is what moves the daily total, not the metric count.
- **Daily Quota**: The free plan allows 800 credits a day. Each symbol I add costs another 26, so 312 leaves room to grow.
- **Market Coverage**: The free plan reaches US markets and US individual stocks alone.

The trading window grows from 6.5 hours to 24 when brokers open 24-hour trading on December 6, 2026. Widening the interval, cutting symbols or paying for a plan is the choice that follows.

## Usage in the AI Era

I run this exporter for **Knowledge Control**. An agent reading the quotes never holds the key to the account.

- **Knowledge Control**: A SaaS platform authenticates with an API key, so giving an agent the data usually means giving it the key. It never sees `TWELVEDATA_API_KEY`, which tightens **security**.
- **Drift Detection**: A quote is an observation rather than a configuration, so no intended state exists to compare a reading against. This exporter does not serve that purpose.

## Development Environment

If you want to test the exporter yourself, a free Twelve Data account and an API key are all it takes. Run the binary with that token and the metrics start arriving. The [Twelve Data documentation](https://twelvedata.com/docs) carries the plan limits.
