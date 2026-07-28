# prometheus — monitoring lab

A small self-contained Prometheus lab: one Prometheus server scraping a Linux host through three exporters, with Grafana dashboards on top. Kept as a snapshot of a hands-on monitoring exercise — for the production-grade setup I run now (kube-prometheus-stack, ArgoCD, SLO burn-rate alerting), see [k8s-monitoring-stack](https://github.com/Inblade/k8s-monitoring-stack).

## Stack

| Component | Port | What it covers |
|---|---|---|
| Prometheus | 9090 | Scraping and storage, 15s interval |
| node_exporter | 9100 | Host metrics: CPU, memory, disk, network |
| cAdvisor | 8080 | Per-container resource usage |
| mysqld_exporter | 9104 | MySQL health and performance |

Configuration lives in [`prometheus.yaml`](prometheus.yaml): four static scrape jobs (Prometheus itself + the three exporters above) against a single lab VM. The IP in the config pointed to a disposable lab instance that no longer exists.

## Screenshots

The `Screenshot *.png` files show the resulting Grafana dashboards: node overview, container metrics, and MySQL panels built on these scrape jobs.

## Running it yourself

```bash
# on the target host
docker run -d --net=host prom/node-exporter
docker run -d --net=host gcr.io/cadvisor/cadvisor
docker run -d --net=host -e DATA_SOURCE_NAME="user:pass@(localhost:3306)/" prom/mysqld-exporter

# point prometheus.yaml targets at the host, then
docker run -d -p 9090:9090 -v $PWD/prometheus.yaml:/etc/prometheus/prometheus.yml prom/prometheus
```

Then add Prometheus as a Grafana data source and import dashboards 1860 (Node Exporter Full), 14282 (cAdvisor), and 7362 (MySQL Overview).
