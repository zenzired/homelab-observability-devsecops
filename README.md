# Multi-Node Raspberry Pi Home Lab: MicroK8s, DevSecOps, Automation & Observability

[![Kubernetes](https://img.shields.io/badge/MicroK8s-v1.35.6-326CE5?logo=kubernetes&logoColor=white)](https://microk8s.io/)
[![Prometheus](https://img.shields.io/badge/Prometheus-v2.x-E6522C?logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Grafana-OSS-F46800?logo=grafana&logoColor=white)](https://grafana.com/)
[![Kyverno](https://img.shields.io/badge/DevSecOps-Kyverno%20%2B%20Trivy-00B39F)](https://kyverno.io/)
[![n8n](https://img.shields.io/badge/Automation-n8n-EA4B71?logo=n8n&logoColor=white)](https://n8n.io/)

A four-node Raspberry Pi home lab engineering environment combining a **MicroK8s control plane**, **DevSecOps admission control and vulnerability scanning**, **event-driven workflow automation (`n8n`)**, **network-wide DNS sinkholing (`Pi-hole`)**, and **centralized metrics observability (`Prometheus` + `Grafana`)**.

Workloads are physically isolated across dedicated boards to eliminate single-board microSD I/O contention while maintaining least-privilege Linux operator accounts (`k8sops`, `gops`, `n8nops`) per node.

---

## Architecture Topology

```mermaid
flowchart TB
    subgraph LAN["Home Lab LAN (192.168.2.0/24)"]
        direction TB

        subgraph PI5["microk8sMaster — Raspberry Pi 5 (192.168.2.174)"]
            direction TB
            K8S["MicroK8s v1.35.6<br/>Ubuntu 24.04 LTS (V30 A2 SD)"]
            KYV["Kyverno Admission Control<br/>(disallow-latest-tag)"]
            TRV["Trivy Vulnerability Scanner"]
            KSM["kube-state-metrics<br/>NodePort :30808"]
            NE1["node_exporter :9100"]
            K8S --- KYV
            K8S --- TRV
            K8S --- KSM
        end

        subgraph PI4_OBS["ubuntu-Host — Raspberry Pi 4 (192.168.2.176)"]
            direction TB
            subgraph DOCKER_OBS["Docker Compose (observability_default)"]
                PROM["Prometheus :9090<br/>(30s scrape | 7d / 1GB cap)"]
                GRAF["Grafana OSS :3000<br/>(SQLite WAL enabled)"]
                GRAF -- "http://prometheus:9090<br/>(Embedded Docker DNS)" --> PROM
            end
            VOL[("/opt/grafana-data<br/>UID 472:472")]
            GRAF --- VOL
        end

        subgraph PI4_N8N["aiagent-n8n — Raspberry Pi 4 (192.168.2.167)"]
            N8N["n8n Automation :5678"]
            NE2["node_exporter :9100"]
        end

        subgraph PIZERO["pizero-pihole — Raspberry Pi Zero W (192.168.2.148)"]
            PIH["Pi-hole Core v6.5 / FTL v6.8"]
            PHE["pihole-exporter :9617<br/>(linux_armv6)"]
            NE3["node_exporter :9100"]
        end
    end

    PROM -- "Scrapes :9100 & :30808" --> PI5
    PROM -- "Scrapes :9100" --> PI4_N8N
    PROM -- "Scrapes :9100 & :9617" --> PIZERO
