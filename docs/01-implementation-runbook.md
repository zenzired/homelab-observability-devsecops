# 01. Implementation Runbook

## Phase 1: Kubernetes Control Plane & DevSecOps (`microk8sMaster` — `192.168.2.174`)
1. **OS & Storage Provisioning:** Flashed Ubuntu 24.04 LTS onto a V30 A2-rated microSD card on the Raspberry Pi 5 to provide sufficient random 4K write IOPS for `kine` / `dqlite`.
2. **MicroK8s Installation & Core Addons:**
   ```bash
   sudo snap install microk8s --classic --channel=1.35/stable
   sudo usermod -a -G microk8s k8sops
   microk8s enable dns hostpath-storage metrics-server rbac
   ```
3. **Namespace Segmentation:**
   ```bash
   kubectl create namespace cka-lab
   kubectl create namespace devsecops
   kubectl create namespace monitoring
   ```
4. **DevSecOps Admission Control & Image Scanning:**
   * Deployed **Kyverno** and applied a `ClusterPolicy` (`disallow-latest-tag`) rejecting pods referencing unpinned `:latest` tags in `cka-lab`.
   * Deployed **Trivy** to scan container workloads for `HIGH` and `CRITICAL` vulnerabilities.
5. **Telemetry Exporters:**
   * Verified `node_exporter` on `:9100`.
   * Exposed `kube-state-metrics` via NodePort `:30808` for external scraping by Prometheus.

## Phase 2: Workflow Automation Node (`aiagent-n8n` — `192.168.2.167`)
1. Provisioned least-privilege operator account `n8nops`.
2. Deployed `n8n` via Docker Compose on port `:5678` for event-driven automation and future alert/CVE webhook ingestion.
3. Installed `prometheus-node-exporter` listening on `:9100`.

## Phase 3: Edge DNS & Host Metrics (`pizero-pihole` — `192.168.2.148`)
* See [02. Edge DNS & Host Metrics](02-edge-dns-and-host-metrics.md) for the exact `systemd` unit configurations and `linux_armv6` binary installation.

## Phase 4: Centralized Observability Hub (`ubuntu-Host` — `192.168.2.176`)
* See [03. Centralized Observability Stack](03-centralized-observability-stack.md) for Docker engine package holds, UID `472:472` volume permissions, and SQLite WAL tuning.
