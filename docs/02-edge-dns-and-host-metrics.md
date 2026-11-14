# 02. Edge DNS & Host Metrics (`pizero-pihole`)

**Target Node:** `pizero-pihole` (`192.168.2.148`) — Raspberry Pi Zero W (`armv6l`) running Pi-hole Core v6.5 / Web v6.6.1 / FTL v6.8.

## 1. Host Telemetry (`node_exporter` v1.8.2)
Installed `node_exporter` as a `systemd` service listening on `0.0.0.0:9100`:
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-armv6.tar.gz
tar xvfz node_exporter-1.8.2.linux-armv6.tar.gz
sudo mv node_exporter-1.8.2.linux-armv6/node_exporter /usr/local/bin/
sudo useradd -rs /bin/false node_exporter
```

Created `/etc/systemd/system/node_exporter.service`:
```ini
[Unit]
Description=Prometheus Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

Enabled and verified:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
curl -s http://localhost:9100/metrics | head -n 10
```

## 2. Pi-hole DNS Telemetry (`pihole-exporter` — `linux_armv6`)
Because the Raspberry Pi Zero W uses an `armv6l` processor, ARM64 and ARMv7 binaries fail with `Exec format error`. Deployed the `linux_armv6` release binary:
```bash
wget https://github.com/eko/pihole-exporter/releases/latest/download/pihole_exporter-linux_armv6 -O /usr/local/bin/pihole_exporter
sudo chmod +x /usr/local/bin/pihole_exporter
```
Configured `pihole-exporter` to scrape the local Pi-hole instance and expose Prometheus metrics on `http://192.168.2.148:9617/metrics`, tracking total DNS queries, blocked queries, block percentage, and upstream DNS latency.
