# netshoot-secure

[![Security Scan](https://img.shields.io/badge/CVEs-0%20Critical%20|%200%20High-brightgreen)](https://github.com/anchore/grype)
[![Go Version](https://img.shields.io/badge/Go-1.25.5-00ADD8)](https://go.dev/)
[![Alpine](https://img.shields.io/badge/Alpine-Edge-0D597F)](https://alpinelinux.org/)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

A security-hardened network troubleshooting container based on [nicolaka/netshoot](https://github.com/nicolaka/netshoot). All critical and high vulnerabilities eliminated through source builds and careful dependency management.

## Security Comparison

| Metric | Original netshoot | netshoot-secure | Reduction |
|--------|:-----------------:|:---------------:|:---------:|
| Critical CVEs | 26 | **0** | 100% |
| High CVEs | 186 | **0** | 100% |
| Medium CVEs | 200+ | 10 | 95% |

## Key Security Improvements

| Change | Why |
|--------|-----|
| Alpine Edge base | Latest security patches, newest package versions |
| Go 1.25.5 source builds | Eliminates Go stdlib vulnerabilities in grpcurl, calicoctl, fortio |
| Pinned release tags | calicoctl built from v3.31.2 for CVE-2023-41378, CVE-2024-33522 fixes |
| urllib3 upgrade | Fixes Python HTTP client vulnerabilities |
| Removed ctop, termshark | Abandoned projects (2022) with unfixable CVEs |

## Included Tools

<details>
<summary><b>50+ Network Diagnostic Tools</b> (click to expand)</summary>

### DNS & Name Resolution
`dig` `nslookup` `drill` `host` `bind-tools`

### HTTP & API Testing
`curl` `httpie` `wget` `grpcurl` `websocat`

### Network Diagnostics
`ping` `mtr` `traceroute` `tcptraceroute` `fping` `arping` `nmap` `nping` `netcat` `socat`

### Traffic Capture & Analysis
`tcpdump` `tshark` `ngrep` `iftop` `iptraf-ng`

### Performance Testing
`iperf` `iperf3` `fortio` `ab`

### Kubernetes & Network Policy
`calicoctl`

### IP, Routing & Firewall
`ip` `ss` `netstat` `route` `arp` `bridge` `iptables` `nftables` `ipset` `conntrack` `ipvsadm`

### TLS/SSL & Mail
`openssl` `swaks`

### Utilities
`ethtool` `scapy` `trippy` `jq` `vim`

</details>

## Quick Start

### Build

```bash
docker build -f Dockerfile.secure -t netshoot-secure:latest .
```

### Run Locally

```bash
docker run -it --rm --cap-add=NET_ADMIN --cap-add=NET_RAW netshoot-secure:latest bash
```

### Deploy to Kubernetes

```bash
# Load into kind cluster
kind load docker-image netshoot-secure:latest --name <cluster>

# Deploy
kubectl apply -f deployment.yaml

# Connect
kubectl exec -it deployment/netshoot -- bash
```

## Kubernetes Deployment

The included `deployment.yaml` provides:

```yaml
hostNetwork: true          # Capture all node traffic
capabilities:
  - NET_ADMIN              # Network configuration
  - NET_RAW                # Raw socket access (tcpdump, nmap)
```

### Common Use Cases

```bash
# Capture DNS traffic on the node
kubectl exec deployment/netshoot -- tshark -i any -f "port 53"

# Test service connectivity
kubectl exec deployment/netshoot -- curl -v http://my-service:8080

# Debug another pod's network namespace
kubectl debug <pod> -it --image=netshoot-secure:latest --target=<container>

# Performance test between pods
kubectl exec deployment/netshoot -- iperf3 -c <target-pod-ip>
```

## Security Scanning

### Recommended: Grype

```bash
grype netshoot-secure:latest
```

### Alternative: Trivy

```bash
trivy image netshoot-secure:latest
```

### Known Scanner Discrepancies

Trivy may report HIGH vulnerabilities in calicoctl (CVE-2023-41378, CVE-2024-33522). These are **false positives** caused by Go module pseudo-versioning in monorepos. The image builds calicoctl from release tag `v3.31.2` which contains all security fixes. Grype correctly identifies 0 HIGH/CRITICAL vulnerabilities.

## Architecture

```
Dockerfile.secure
├── Stage 1: Go Builder (golang:1.25.5-alpine)
│   ├── grpcurl    (built from source)
│   ├── calicoctl  (built from v3.31.2 tag)
│   └── fortio     (built from source)
│
└── Stage 2: Runtime (alpine:edge)
    ├── Network tools (apk packages)
    ├── Python tools (pip upgraded)
    └── Go binaries (copied from builder)
```

## Removed Tools

| Tool | Reason | Alternative |
|------|--------|-------------|
| ctop | Abandoned March 2022, runc CVEs | `docker stats`, k9s |
| termshark | Abandoned 2022, Go stdlib CVEs | `tshark` (included) |

## Contributing

1. Fork the repository
2. Create a feature branch
3. Run security scans before submitting
4. Submit a pull request

## License

Apache License 2.0 - see [LICENSE](LICENSE)

## Credits

Based on [nicolaka/netshoot](https://github.com/nicolaka/netshoot) by Nicolas Leiva.
