# Security Policy

## Vulnerability Status

This image is actively maintained for security. All Go binaries are built from source with the latest Go compiler to ensure stdlib vulnerabilities are addressed.

| Component | Version | Source |
|-----------|---------|--------|
| Base Image | Alpine Edge | Official Alpine |
| Go Compiler | 1.25.5 | golang:1.25.5-alpine |
| grpcurl | latest | Built from source |
| calicoctl | v3.31.2 | Built from release tag |
| fortio | latest | Built from source |

## Scanning Results

Scanned with [Grype](https://github.com/anchore/grype):

- **Critical**: 0
- **High**: 0
- **Medium**: 10
- **Low**: 4

## Reporting a Vulnerability

If you discover a security vulnerability in this project:

1. **Do not** open a public issue
2. Email the maintainer directly or use GitHub's private vulnerability reporting
3. Include steps to reproduce and potential impact
4. Allow reasonable time for a fix before public disclosure

## Security Practices

### Build Process

- Multi-stage Docker builds to minimize attack surface
- Go binaries compiled with `CGO_ENABLED=0` for static linking
- Binary stripping with `-ldflags="-s -w"` to reduce size
- No unnecessary packages in final image

### Dependency Management

- Alpine packages updated on each build (`apk upgrade`)
- Go dependencies pulled fresh during build
- Python packages upgraded via pip
- Release tags used for critical security fixes

### What's NOT Included

Tools removed due to security concerns:

| Tool | Removed Because |
|------|-----------------|
| ctop | Abandoned since 2022, vulnerable runc dependencies |
| termshark | Abandoned since 2022, outdated Go stdlib |

## Recommended Usage

For production environments:

1. Rebuild the image regularly to get latest patches
2. Use specific image digests instead of `:latest` tag
3. Apply Pod Security Standards in Kubernetes
4. Limit capabilities to only what's needed
5. Use network policies to restrict pod communication
