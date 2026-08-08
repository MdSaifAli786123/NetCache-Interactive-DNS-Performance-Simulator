# NetCache-Interactive-DNS-Performance-Simulator

# NetCache — Interactive DNS Performance Simulator

An interactive **Computer Networks simulation platform** for studying how DNS caching and network conditions affect resolution performance.

NetCache simulates the end-to-end flow of DNS requests through a **client → DNS cache → upstream DNS server** architecture and allows network parameters to be changed dynamically while the simulation is running.

The system models:

* DNS caching
* LRU cache eviction
* TTL-based cache expiration
* Query locality
* Network latency
* Network jitter
* Packet loss
* Retransmissions
* Upstream DNS traffic
* Cache hit/miss behavior
* P95 latency
* Interactive performance visualization

The complete system is designed to run in **Google Colab** without requiring Docker, Mininet, NS-3, or any special networking configuration.

---

## 1. Project Overview

DNS is one of the most frequently used services on the Internet. Every time a client accesses a domain, the corresponding hostname must be resolved to an IP address.

Repeatedly querying upstream DNS infrastructure introduces additional latency and network traffic. DNS caching addresses this by storing previously resolved records temporarily and serving subsequent requests locally until their TTL expires.

NetCache provides a controlled environment to investigate this behavior.

Instead of implementing isolated networking algorithms, the project models a complete DNS request lifecycle:

```text
                    DNS NETWORK SIMULATION

        ┌───────────────┐
        │    Client     │
        │  / User Apps  │
        └───────┬───────┘
                │
                │ DNS Query
                ▼
        ┌───────────────┐
        │  DNS Resolver │
        │    + Cache    │
        └───────┬───────┘
                │
          Cache Lookup
           /         \
         HIT         MISS
          │            │
          │            ▼
          │     ┌───────────────┐
          │     │ Network Link  │
          │     └───────┬───────┘
          │             │
          │       Latency/Jitter
          │       Packet Loss
          │             │
          │             ▼
          │     ┌───────────────┐
          │     │ Upstream DNS  │
          │     │    Server     │
          │     └───────┬───────┘
          │             │
          │       DNS Response
          │             │
          │             ▼
          │       Cache Update
          │             │
          └─────────────┘
                │
                ▼
           Client Response
```

---

# 2. Objectives

The primary objectives of NetCache are to:

1. Simulate the behavior of a DNS caching resolver.
2. Model realistic network conditions.
3. Study the effect of cache size and TTL on DNS performance.
4. Model user query locality.
5. Simulate packet loss and retransmissions.
6. Measure DNS response latency.
7. Quantify upstream DNS traffic reduction.
8. Provide an interactive environment where parameters can be changed dynamically.
9. Visualize the effect of network conditions in real time.

---

# 3. Key Features

### DNS Cache

The simulator implements a custom DNS cache using:

* **LRU eviction**
* **TTL-based expiration**
* Dynamic cache capacity

When the cache reaches its capacity, the least recently used entry is removed.

```text
Cache:

[github.com]
[google.com]
[openai.com]
[python.org]
      ↓
Cache Full
      ↓
Least Recently Used Entry
      ↓
Evicted
```

---

### TTL-Based Expiration

Each cached DNS record has an expiration time.

```text
DNS Response
     │
     ▼
Cache Entry
     │
     ├── Created At
     └── Expires At
             │
             ▼
        TTL Expired?
          /      \
        No        Yes
        │          │
        ▼          ▼
      HIT        MISS
```

This allows the simulator to investigate how shorter TTL values increase upstream DNS traffic.

---

### Query Locality

Real workloads are not uniformly random. Users frequently revisit popular websites.

The simulator therefore provides a **query locality parameter**.

For example:

```text
Locality = 0%

google.com
github.com
amazon.com
reddit.com
youtube.com
...

```

versus:

```text
Locality = 80%

google.com
google.com
google.com
github.com
github.com
google.com
...
```

Higher locality generally produces more cache hits.

---

### Network Latency

The simulated network introduces configurable latency between the resolver and upstream DNS server.

```text
Client
  │
  ▼
Cache
  │
  │  Network Latency
  ▼
DNS Server
```

The parameter can be varied from low-latency to high-latency network conditions.

---

### Network Jitter

Real networks do not always provide identical latency for every request.

The simulator therefore models latency variation around the configured network latency.

Conceptually:

```text
Observed Latency
      =
Base Network Latency
      +
Random Jitter
```

This produces a more realistic latency distribution.

---

### Packet Loss and Retransmission

The simulator can introduce packet loss during upstream DNS communication.

```text
DNS Query
    │
    ▼
Network
    │
    ├── Delivered ──────► DNS Server
    │
    └── Lost
         │
         ▼
    Retransmission
```

This allows the effect of unreliable network conditions on DNS response latency to be studied.

---

# 4. End-to-End Pipeline

The complete simulation pipeline is:

```text
┌───────────────────────────────┐
│ 1. Simulation Configuration   │
│                               │
│ Cache Size                    │
│ TTL                           │
│ Query Locality                │
│ Network Latency               │
│ Jitter                        │
│ Packet Loss                   │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│ 2. Request Generation         │
│                               │
│ Generate DNS domain request   │
│ according to locality model   │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│ 3. Cache Lookup               │
│                               │
│ Check domain                  │
│ Check TTL                     │
└───────────────┬───────────────┘
                │
          ┌─────┴─────┐
          │           │
        HIT          MISS
          │           │
          ▼           ▼
┌──────────────┐  ┌────────────────┐
│ Local Cache  │  │ Upstream DNS   │
│ Response     │  │ Request        │
└──────┬───────┘  └───────┬────────┘
       │                  │
       │           ┌──────┴──────┐
       │           │             │
       │       Delivered        Lost
       │           │             │
       │           │        Retransmission
       │           │             │
       │           └──────┬──────┘
       │                  │
       │                  ▼
       │          ┌───────────────┐
       │          │ Cache Update  │
       │          │ + TTL         │
       │          └───────┬───────┘
       │                  │
       └──────────────────┘
                │
                ▼
┌───────────────────────────────┐
│ 4. Performance Measurement    │
│                               │
│ Latency                      │
│ P95 Latency                  │
│ Cache Hit Ratio              │
│ Upstream Queries             │
│ Packet Loss                  │
│ Retransmissions              │
│ TTL Expirations              │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│ 5. Interactive Dashboard      │
│                               │
│ Live Graphs                   │
│ Metrics                       │
│ Request Logs                  │
│ Network State                 │
└───────────────────────────────┘
```

---

# 5. System Architecture

## High-Level Architecture

```text
                         ┌─────────────────────┐
                         │   User Parameters   │
                         ├─────────────────────┤
                         │ Cache Size          │
                         │ TTL                 │
                         │ Locality            │
                         │ Latency             │
                         │ Jitter              │
                         │ Packet Loss         │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Request Generator   │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │   DNS Resolver      │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │    DNS Cache        │
                         │                     │
                         │ LRU + TTL           │
                         └──────────┬──────────┘
                                    │
                              Cache Miss
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Network Simulator   │
                         │                     │
                         │ Latency             │
                         │ Jitter              │
                         │ Packet Loss         │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │  Upstream DNS       │
                         │     Server          │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Performance Engine  │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Interactive         │
                         │ Dashboard           │
                         └─────────────────────┘
```

---

# 6. Core Components

## 6.1 Request Generator

The request generator creates DNS queries from a predefined domain pool.

Example domain set:

```text
google.com
github.com
youtube.com
amazon.com
wikipedia.org
microsoft.com
apple.com
linkedin.com
reddit.com
stackoverflow.com
cloudflare.com
python.org
openai.com
netflix.com
...
```

The **query locality** parameter controls how frequently the previous domain is requested again.

---

## 6.2 DNS Cache

The cache maintains DNS records using an ordered data structure.

Each cache entry contains:

```text
Domain
Created Time
Expiration Time
```

The cache performs:

```text
Lookup
   ↓
Exists?
   ↓
TTL Valid?
   ↓
HIT
```

or:

```text
Not Found / Expired
        ↓
      MISS
        ↓
Upstream DNS Query
```

---

## 6.3 LRU Eviction

When the cache reaches its configured capacity:

```text
Cache Capacity = 3

[A]
[B]
[C]

New Entry → D

Least Recently Used:
A

Result:

[B]
[C]
[D]
```

This allows cache capacity to be experimentally varied.

---

## 6.4 Network Model

The network layer models three important characteristics:

### Latency

Controls the base delay of upstream DNS communication.

### Jitter

Introduces variation around the base latency.

### Packet Loss

Randomly drops upstream requests and triggers retransmission behavior.

Together they provide a controlled network environment for experimentation.

---

# 7. Interactive Dashboard

The simulator provides adjustable controls for:

| Parameter       | Purpose                                 |
| --------------- | --------------------------------------- |
| Cache Size      | Maximum number of cached records        |
| TTL             | Lifetime of cached DNS entries          |
| Query Locality  | Probability of repeated domain requests |
| Network Latency | Base upstream network delay             |
| Jitter          | Latency variation                       |
| Packet Loss     | Probability of packet loss              |
| Requests / Step | Number of requests generated per update |
| Update Interval | Simulation refresh rate                 |

Parameters can be modified while the simulation is running.

For example:

```text
Cache Size
20 ───────────────► 5
```

causes the simulator to immediately use the smaller cache for subsequent requests.

Similarly:

```text
Packet Loss
2% ───────────────► 20%
```

changes the network behavior for subsequent requests.

---

# 8. Performance Metrics

The simulator continuously calculates several networking metrics.

## Cache Hit Ratio

Measures how many requests are served directly from the cache.

```text
Cache Hit Ratio =
Cache Hits / Total Requests
```

A higher value indicates more effective caching.

---

## Average Latency

Measures the mean response latency over the observed requests.

```text
Average Latency =
Σ Request Latency / Number of Requests
```

---

## P95 Latency

The 95th percentile latency indicates the response time below which approximately 95% of observed requests fall.

This is useful because average latency alone can hide high-latency requests.

---

## Upstream DNS Queries

Counts the number of requests that had to leave the local cache and reach upstream DNS infrastructure.

A higher cache hit ratio should generally reduce this value.

---

## Packet Loss Events

Counts simulated upstream packet-loss events.

---

## Retransmissions

Counts requests requiring retransmission following packet loss.

---

## TTL Expirations

Tracks how frequently cached records expire and require another upstream resolution.

---

# 9. Experiments

The simulator supports controlled experiments by changing one parameter while keeping other parameters fixed.

## Experiment 1 — Cache Size

Objective:

> Study the effect of cache capacity on DNS caching efficiency.

Example:

```text
Cache Size:
2 → 5 → 10 → 20 → 50 → 100
```

Measure:

* Cache hit ratio
* Average latency
* Upstream DNS queries

Expected relationship:

```text
Larger Cache
      ↓
Fewer Evictions
      ↓
Higher Hit Ratio
      ↓
Fewer Upstream Queries
```

---

## Experiment 2 — TTL

Objective:

> Study the effect of DNS record lifetime on cache effectiveness.

Example:

```text
TTL:
1 s → 5 s → 10 s → 30 s → 60 s
```

Shorter TTL:

```text
Short TTL
   ↓
More Expiration
   ↓
More Cache Misses
   ↓
More Upstream Queries
```

---

## Experiment 3 — Query Locality

Objective:

> Study the relationship between workload locality and cache efficiency.

Example:

```text
Locality:
0% → 25% → 50% → 75% → 90%
```

Higher locality generally produces more repeated queries and therefore higher cache utilization.

---

## Experiment 4 — Network Latency

Objective:

> Study the effect of upstream network delay on DNS response performance.

Example:

```text
Latency:
10 ms → 30 ms → 50 ms → 100 ms → 200 ms
```

Cache hits remain largely independent of upstream latency because they are served locally, while cache misses become increasingly expensive.

---

## Experiment 5 — Packet Loss

Objective:

> Analyze DNS performance under unreliable network conditions.

Example:

```text
Packet Loss:
0% → 1% → 5% → 10% → 20%
```

Increasing packet loss can lead to:

```text
Packet Loss
     ↓
Retransmissions
     ↓
Higher Response Latency
```

---

# 10. Example Scenario

Consider the initial configuration:

```text
Cache Size       = 20
TTL              = 10 seconds
Query Locality   = 70%
Network Latency  = 30 ms
Jitter           = 5 ms
Packet Loss      = 2%
```

Start the simulation and observe the baseline behavior.

Then dynamically change:

```text
Cache Size:       20 → 5
```

The smaller cache causes more frequent evictions.

Next:

```text
TTL:              10 → 2 seconds
```

Cached records expire more frequently.

Finally:

```text
Packet Loss:       2% → 20%
```

More upstream requests experience simulated packet loss and retransmission.

The dashboard allows the resulting changes in latency, cache hit ratio, upstream traffic, and retransmissions to be observed immediately.

---

# 11. Technology Stack

### Programming

* Python

### Simulation

* Custom Python simulation engine
* `collections.OrderedDict`
* `deque`
* Randomized network modeling

### Data Processing

* NumPy
* Pandas

### Visualization

* Matplotlib

### Interactive Interface

* IPyWidgets

### Execution Environment

* Google Colab
* Jupyter Notebook compatible

---

# 12. Why Google Colab?

The simulator is intentionally designed to be lightweight.

It does not require:

* Docker
* Kubernetes
* Mininet
* NS-3
* Linux network namespaces
* Multiple physical machines
* Dedicated network interfaces
* GPU

The entire simulation runs as a controlled Python-based network model.

This makes it suitable for experimentation without modifying the local machine's networking configuration.

---

# 13. Project Structure

A recommended GitHub repository structure is:

```text
NetCache/
│
├── README.md
│
├── notebooks/
│   └── NetCache_DNS_Simulator.ipynb
│
├── results/
│   ├── cache_size_results.csv
│   ├── ttl_results.csv
│   ├── locality_results.csv
│   └── packet_loss_results.csv
│
├── figures/
│   ├── cache_performance.png
│   ├── latency_analysis.png
│   ├── upstream_traffic.png
│   └── network_events.png
│
├── requirements.txt
│
└── LICENSE
```

---

# 14. Running the Project

## Option 1 — Google Colab

Open the notebook in Google Colab and execute the cells sequentially.

```text
1. Initialize environment
        ↓
2. Create DNS cache
        ↓
3. Initialize simulator
        ↓
4. Configure parameters
        ↓
5. Launch dashboard
        ↓
6. Start simulation
        ↓
7. Change parameters dynamically
        ↓
8. Observe network behavior
        ↓
9. Run controlled experiments
```

## Option 2 — Local Jupyter

Install dependencies:

```bash
pip install numpy pandas matplotlib ipywidgets
```

Then launch:

```bash
jupyter notebook
```

and open the simulator notebook.

---

# 15. Example Results

The exact results depend on the simulation seed and configuration.

The project should report measured values such as:

```text
Total Requests
Cache Hit Ratio
Average Latency
P95 Latency
Upstream DNS Queries
Packet Loss Events
Retransmissions
TTL Expirations
```

For example, a final experiment table can be structured as:

| Configuration | Cache Hit Ratio | Avg. Latency | P95 Latency | Upstream Queries |
| ------------- | --------------: | -----------: | ----------: | ---------------: |
| Small Cache   |        Measured |     Measured |    Measured |         Measured |
| Medium Cache  |        Measured |     Measured |    Measured |         Measured |
| Large Cache   |        Measured |     Measured |    Measured |         Measured |

**Results should be populated from actual simulator runs rather than hard-coded values.**

---

# 16. Engineering Insights

The simulator can be used to demonstrate several important networking relationships.

### Cache capacity vs traffic

Increasing cache capacity can reduce eviction frequency and therefore reduce upstream DNS traffic when the workload has sufficient locality.

### TTL vs freshness

A smaller TTL causes cached records to expire sooner, increasing upstream resolution frequency.

### Locality vs caching

A workload with strong temporal locality benefits more from caching than a workload where every request targets a different domain.

### Network latency vs caching

Caching becomes particularly valuable when upstream DNS communication has high latency because cache hits avoid that upstream path.

### Packet loss vs reliability

Higher packet loss increases the probability of retransmission and can increase observed response latency.

---

# 17. Limitations

NetCache is a **controlled network simulation**, not a production DNS resolver.

The following aspects are abstracted:

* Actual DNS packet serialization
* Full DNS protocol message structure
* Recursive DNS resolution hierarchy
* Authoritative DNS server implementation
* TCP fallback for DNS
* DNSSEC
* Real routing behavior
* Physical network propagation
* Congestion-control algorithms

The purpose is to isolate and study **DNS caching and network-performance behavior** in a reproducible environment.

---

# 18. Future Extensions

Possible extensions include:

* Multiple DNS resolvers
* Recursive DNS hierarchy
* Authoritative server simulation
* DNS record types such as A, AAAA, CNAME and MX
* DNS cache poisoning scenarios
* DNS-over-HTTPS (DoH) simulation
* DNS-over-TLS (DoT) simulation
* Resolver load balancing
* Multi-client workloads
* Request burst modeling
* Adaptive cache sizing
* Cache replacement policy comparison
* Real DNS query validation
* Exportable experiment reports

These are intentionally outside the current core implementation to keep the project focused.

---

# 19. Learning Outcomes

This project provides practical exposure to:

* DNS resolution
* Client-server communication
* Caching
* TTL
* LRU eviction
* Network latency
* Jitter
* Packet loss
* Retransmission
* Performance benchmarking
* Workload modeling
* Network simulation
* Interactive visualization

It also demonstrates how a networking system can be evaluated experimentally rather than only implemented theoretically.

---

# 20. Resume Summary

**NetCache — Interactive DNS Performance Simulator**

> Built an interactive DNS caching simulator with LRU/TTL management and configurable network conditions across 5,000+ requests; benchmarked cache hit ratio, P95 latency, upstream DNS traffic, and retransmissions across varying cache and network configurations.

---

# 21. Key Takeaway

NetCache transforms DNS caching from a textbook concept into an **interactive network-performance experiment**.

The central relationship is:

```text
             Workload
                │
                ▼
          DNS Request
                │
                ▼
         ┌─────────────┐
         │ DNS Cache   │
         └──────┬──────┘
                │
          ┌─────┴─────┐
          │           │
         HIT         MISS
          │           │
          │           ▼
          │      Network
          │      Conditions
          │           │
          │     ┌─────┼─────┐
          │     ▼     ▼     ▼
          │ Latency Jitter Loss
          │           │
          │           ▼
          │      DNS Server
          │           │
          │           ▼
          │       Cache Update
          │           │
          └───────────┘
                │
                ▼
         Performance Metrics
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
    Hit Ratio Latency Traffic
```

**The project is therefore centered on one clear engineering question:**

> **How do DNS caching policies and network conditions affect resolver performance?**

That single question ties the entire architecture, simulation, dashboard, and experimental evaluation together.
