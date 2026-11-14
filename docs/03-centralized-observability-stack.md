# 03. Centralized Observability Stack (`ubuntu-Host`)

**Target Node:** `ubuntu-Host` (`192.168.2.176`) — Raspberry Pi 4 running Ubuntu LTS.

## 1. Host Prep & Least-Privilege Operator Isolation
1. **Docker Engine Dependency Stabilization:** Resolved Ubuntu Focal repository deprecations (`docker-model-plugin` missing package) and placed `apt-mark hold` on `containerd` and `docker.io` to prevent broken package upgrades during `apt upgrade`.
2. **Service User & Data Ownership:**
   * Provisioned non-sudo operator `gops` to manage Compose files under `/home/gops/observability`.
   * Created `/opt/grafana-data` owned by UID/GID `472:472` (matching the internal `grafana` container user) so dashboard state and plugins survive directory migrations.
   ```bash
   sudo mkdir -p /opt/grafana-data
   sudo chown -R 472:472 /opt/grafana-data
   ```

## 2. SD-Card-Safe Compose Configuration
* **SQLite WAL Mode:** Enabled `GF_DATABASE_WAL=true` and `GF_DATABASE_BUSY_TIMEOUT=10000` in `observability/docker-compose.yml` to prevent `SQLITE_BUSY` database locks during concurrent alert evaluation and dashboard writes.
* **Bounded Prometheus TSDB:** Set `scrape_interval: 30s` in `observability/prometheus.yml` and capped retention (`--storage.tsdb.retention.time=7d`, `--storage.tsdb.retention.size=1GB`) to protect SD card endurance.
* **Internal Bridge DNS:** Connected Grafana to Prometheus using `http://prometheus:9090` across Docker's embedded DNS (`observability_default`).

## 3. Dashboards & External Data Sources
* **Raspberry Pi Home Lab Overview:** Visualizes node availability (`up`), CPU load, RAM usage, SD card I/O busy time, MicroK8s ready pods by namespace, and Pi-hole block rates.
* **Open-Meteo Weather Panel (Grafana Infinity Plugin):** Configured a JSON API data source (`https://api.open-meteo.com/v1/forecast?latitude=-34.9285&longitude=138.6007&hourly=temperature_2m`) using UQL/JSONPath parsing to correlate ambient temperature with Raspberry Pi thermal metrics.
