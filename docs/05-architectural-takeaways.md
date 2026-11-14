# 05. Key Architectural Takeaways

1. **Engineer Around microSD Storage Bottlenecks First:**
   Single-board Kubernetes and observability stacks fail on random 4K write latency long before exhausting CPU. Combining **V30 A2 media**, **SQLite WAL mode (`GF_DATABASE_WAL=true`)**, **`30s` scrape intervals**, and **strict TSDB retention caps (`7d` / `1GB`)** eliminates datastore lockups and premature SD wear.
2. **Use Container-Network DNS Over Host IPs for Co-Located Services:**
   Connecting Grafana to Prometheus via `http://prometheus:9090` over Docker's embedded DNS (`127.0.0.11`) avoids hairpin NAT/firewall rules and prevents breakage if host DHCP leases change.
3. **Enforce Least-Privilege Operator Boundaries:**
   Separating host-level `sudo` administration (`ubuntu`) from workload service operators (`k8sops`, `gops`, `n8nops`)—paired with explicit UID-mapped host volumes (`/opt/grafana-data` -> `472:472`)—minimizes blast radius and keeps persistent data portable.
4. **Match Binaries Strictly to Hardware ISA:**
   Mixing Raspberry Pi generations (`Pi 5` ARM64, `Pi 4` ARM64, `Pi Zero W` ARMv6) requires explicit binary verification (`uname -m`) before deploying standalone Go exporters.
5. **Troubleshoot Connectivity Bottom-Up Across Layers:**
   Stepping methodically from container port bindings (`ss -tulpn`) -> loopback HTTP (`curl -I localhost:3000`) -> host firewall (`ufw`/`iptables`) -> cross-subnet TCP sockets (`Test-NetConnection`) -> container logs (`docker logs`) separates network blocks from storage locks in minutes.
