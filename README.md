# allThingsHonk

**Simulating Distributed DDoS in Fediverse: ActivityPub Delivery Failures on Honk Servers**

This repo contains scripts and experiments demonstrating how server outages in decentralized ActivityPub (Fediverse) networks create DDoS-like effects through retry overloads. Tracks "tries" (delivery attempts) during simulated failures.

[Overview](#references)

## Overview

ActivityPub's push delivery retries during outages (power failure, updates, DDoS) cause network-wide lag and resource exhaustion. Even blocked users trigger media fetches via MRF on every view. This study uses lightweight Honk servers to quantify "tries" and undelivered recipients (rcpt) across TWKN-scale simulations.

Why Honk? Minimalist (SQLite, no proxies/likes), easy to scale (5000+ ports via Nginx wildcard), source-modifiable for aggressive retry testing vs. Pleroma/Mastodon bloat.

## Key Findings

| Scenario    | Avg Tries/Host | Undelivered Rcpt % | Notes              |
|-------------|----------------|-------------------|--------------------|
| Normal      | 1-2            | <5%               | Baseline           |
| 50% Down    | 4-6            | 30%               | Post-outage spike  |
| Aggressive  | 10+            | 70%+              | DDoS simulation    |


Reveals ripple effects: one downed server backlogs entire network.

## Scripts

- honk-manage: Start/stop instances, random_sleep (1s-1h) simulates real outages, purge-all cleans up.

- honk-aggressively: Bark random lorem posts, subscribe-all for network stress.

- honk-servers: Controls honkdistribution% of servers posting at honktime intervals.

- peeping-tom: SQLite logger for runs/tries/rcpt from FIFO pipe.

- honk-manager: Orchestrates all—sets up/down%, runs peeping + honking in loop.

## Setup

1. Multi-server Nginx (bikalpa{5000-6000}.ashwink.com.np):
```
server {
    server_name ~^bikalpa(\d+)\.ashwink\.com\.np$;
    proxy_set_header Host $http_host;
    proxy_pass http://localhost:$1;
}
```
2. Wildcard SSL for *.ashwink.com.np. Map hosts in /etc/hosts or local DNS.

3. Honk mods: Edit retry intervals (5s, 25s, 1m, 90s, 2m+).

4. Dependencies: Bash, SQLite3, Go (for Honk), Nginx.

## Data Collected

- runs: Timestamped sync points + total tries/rcpt-left.

- tries: Undelivered attempts per host (high = overload).

- rcpt: Per-recipient delivery fails.

## ​Running Experiments
```
# Syntax: ./honk-manager <total_servers> <up_percent> <down_percent> <honk_percent> [honktime]
./honk-manager 100 70 30 20 30

# View data
sqlite3 peeping.db "SELECT * FROM runs ORDER BY timestamp DESC LIMIT 10;"
```

## References

[Honk​](https://humungus.tedunangst.com/r/honk)

[ActivityPub.rocks​](https://activitypub.rocks/)

[Pleroma](https://pleroma.social/)

