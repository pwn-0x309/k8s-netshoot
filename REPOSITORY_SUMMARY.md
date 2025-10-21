# Repository Summary: k8s-netshoot

## Overview

**netshoot** is a comprehensive Docker and Kubernetes network troubleshooting container that provides a swiss-army knife of networking tools for diagnosing and resolving network issues in containerized environments.

## Purpose

This project addresses the complexity of Docker and Kubernetes network troubleshooting by providing a single, ready-to-use container image packed with powerful networking diagnostic tools. It eliminates the need to install troubleshooting packages directly on production hosts or application containers.

## Key Features

### 1. Network Namespace Support
- Can attach to container network namespaces for debugging specific containers
- Supports host network namespace for troubleshooting host-level issues
- Can enter Docker network namespaces using `nsenter`
- Kubernetes pod network namespace support via ephemeral containers or sidecars

### 2. Extensive Tool Collection (70+ packages)

**Core Networking Tools:**
- `tcpdump`, `tshark`, `termshark` - Packet capture and analysis
- `iperf`, `iperf3` - Network performance testing
- `nmap`, `nping` - Network scanning and discovery
- `netcat`, `socat` - Network connections and port testing
- `curl`, `httpie` - HTTP client tools
- `grpcurl` - gRPC testing

**DNS & Resolution:**
- `drill`, `bind-tools` - DNS queries
- `net-snmp-tools` - SNMP utilities

**Network Analysis:**
- `mtr`, `traceroute`, `tcptraceroute` - Route tracing
- `iftop`, `iptraf-ng` - Traffic monitoring
- `ethtool` - Ethernet device queries
- `netstat` - Network statistics

**Routing & Firewalls:**
- `iproute2` - Advanced routing (ip command)
- `bridge-utils` - Bridge administration
- `iptables`, `nftables`, `ipset` - Firewall management
- `conntrack-tools` - Connection tracking
- `ipvsadm` - IPVS administration

**Other Utilities:**
- `ctop` - Container metrics monitoring
- `calicoctl` - Calico networking management
- `fortio` - Load testing
- `swaks` - SMTP testing
- `scapy` - Packet manipulation
- `websocat` - WebSocket client
- `strace`, `ltrace` - System/library call tracing

### 3. Multi-Platform Support
- **x86_64/amd64**: Full support
- **ARM64/aarch64**: Full support
- Multi-architecture Docker images using buildx

### 4. User-Friendly Shell Environment
- **ZSH** with Oh-My-Zsh framework
- **Powerlevel10k** theme
- Auto-suggestions plugin
- Custom MOTD (Message of the Day) with ASCII art
- Pre-configured for optimal troubleshooting experience

## Architecture

### Build Process
The Dockerfile uses a **multi-stage build**:

1. **Stage 1 (Fetcher)**: Debian-based stage that downloads pre-compiled binaries
   - ctop (container monitoring)
   - calicoctl (Calico networking)
   - termshark (TUI packet analyzer)
   - grpcurl (gRPC client)
   - fortio (load testing)

2. **Stage 2 (Final Image)**: Alpine Linux 3.18.0 base
   - Installs 70+ networking packages via apk
   - Copies binaries from fetcher stage
   - Configures ZSH environment
   - Sets up proper permissions for OpenShift compatibility

### Repository Structure
```
k8s-netshoot/
├── Dockerfile              # Multi-stage container build definition
├── Makefile               # Build automation (x86, ARM64, multi-arch)
├── README.md              # Comprehensive documentation with examples
├── LICENSE                # GPL v2 license
├── zshrc                  # ZSH configuration
├── motd                   # Custom message of the day
├── build/
│   └── fetch_binaries.sh  # Script to download external binaries
├── configs/
│   ├── netshoot-calico.yaml     # Kubernetes Calico configuration
│   └── netshoot-sidecar.yaml    # Kubernetes sidecar deployment
├── img/                   # Screenshot images for documentation
└── .github/               # GitHub-specific files
```

## Usage Patterns

### Docker Usage
```bash
# Debug container's network namespace
docker run -it --net container:<container_name> nicolaka/netshoot

# Debug host network
docker run -it --net host nicolaka/netshoot
```

### Kubernetes Usage
```bash
# Ephemeral debug container (recommended)
kubectl debug <pod-name> -it --image=nicolaka/netshoot

# Standalone troubleshooting pod
kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot

# Host network debugging
kubectl run tmp-shell --rm -i --tty --overrides='{"spec": {"hostNetwork": true}}' --image nicolaka/netshoot

# Sidecar container (persistent debugging)
kubectl apply -f configs/netshoot-sidecar.yaml
```

### Docker Compose Integration
Compatible with Docker Compose for service-level network debugging with network_mode: service:<service_name>

## Common Troubleshooting Scenarios

The tool addresses various network issues:
- **Latency**: `ping`, `mtr`, `iperf`
- **Routing**: `ip route`, `traceroute`, `netstat`
- **DNS Resolution**: `drill`, `nslookup`, `dig`
- **Firewall Issues**: `iptables`, `nmap`, `netcat`
- **Incomplete ARPs**: `ip neigh`, `arp`
- **Traffic Analysis**: `tcpdump`, `termshark`, `iftop`
- **Service Connectivity**: `curl`, `netcat`, `grpcurl`

## Technology Stack

- **Base OS**: Alpine Linux 3.18.0
- **Shell**: ZSH with Oh-My-Zsh
- **Package Manager**: APK (Alpine Package Keeper)
- **Container Runtime**: Docker (compatible with any OCI runtime)
- **Orchestration**: Docker Swarm and Kubernetes compatible
- **Build Tool**: Docker Buildx for multi-architecture builds

## Build Configuration

**Makefile targets:**
- `build-x86`: Build for amd64 platform
- `build-arm64`: Build for ARM64 platform
- `build-all`: Multi-architecture build (both platforms)
- `push`: Push image to Docker Hub
- `all`: Build all architectures and push

**Image Details:**
- Docker Hub: `nicolaka/netshoot`
- Default version: 0.1
- Latest Alpine: 3.18.0

## Special Features

### OpenShift Compatibility
- Proper group permissions for non-root execution
- Compatible with OpenShift's security context constraints

### Network Namespace Exploration
- Can enter overlay/bridge network namespaces using `nsenter`
- Requires privileged mode and `/var/run/docker/netns` mount
- Enables inspection of network bridges, routing tables, ARP caches at network level

### Packet Capture Capabilities
- Supports `tcpdump` with full packet capture
- `termshark` for TUI-based Wireshark-like experience
- `tshark` for command-line packet analysis
- Requires `NET_ADMIN` and `NET_RAW` capabilities

## Dependencies

**Runtime Requirements:**
- Docker or compatible OCI runtime
- For Kubernetes: kubectl with appropriate cluster access
- For privileged operations: CAP_NET_ADMIN, CAP_NET_RAW capabilities

**Build Requirements:**
- Docker with buildx support (for multi-arch builds)
- Internet connectivity for package downloads

## License

GNU General Public License v2.0 (GPL-2.0)

## Contributing

The project welcomes contributions for additional networking tools and use cases. Requirements:
- Justify the tool's usefulness and uniqueness
- Update Dockerfile to include the package
- Leverage multi-stage builds for source-compiled tools
- Update README with usage examples
- Ensure multi-platform support where applicable

## Maintenance Status

- **Current Alpine Version**: 3.18.0
- **Active Development**: Yes
- **Community Support**: GitHub issues and pull requests
- **Documentation**: Comprehensive README with real-world examples

## Image Size Optimization

Despite the extensive toolset, the image maintains reasonable size through:
- Alpine Linux minimal base (~5MB base)
- Multi-stage builds to exclude build dependencies
- Efficient package management with `--no-cache`
- Strategic use of edge repositories for latest tools

## Security Considerations

- Container can run in privileged mode for advanced debugging
- Supports running as non-root user (OpenShift compatible)
- Network packet capture requires explicit capability grants
- Should be used as ephemeral troubleshooting tool, not production workload

---

**Repository**: https://github.com/nicolaka/netshoot (assumed)
**Maintainer**: nicolaka
**Last Alpine Update**: 3.18.0
**Primary Use Case**: Network troubleshooting in Docker and Kubernetes environments
