# tiny-tailscale

Minimal, stripped-down [Tailscale](https://tailscale.com/) combined binaries — auto-built from upstream stable releases.

This fork adds no source changes to tailscale. It provides a CI pipeline that builds optimized single-binary releases using tailscale's own `cmd/featuretags --min` tool to strip unused features while keeping everything needed for embedded Linux deployments (routers, cellular modems, IoT).

## Downloads

Grab the latest build from [**Releases**](https://github.com/iamromulan/tiny-tailscale/releases).

| Architecture | File | Notes |
|---|---|---|
| amd64 | `tiny-tailscale_<ver>_amd64.tgz` | x86-64 |
| 386 | `tiny-tailscale_<ver>_386.tgz` | x86 32-bit |
| arm64 | `tiny-tailscale_<ver>_arm64.tgz` | AArch64 (Qualcomm modems, RPi 3+, etc.) |
| arm | `tiny-tailscale_<ver>_arm.tgz` | ARMv7 32-bit |
| mips | `tiny-tailscale_<ver>_mips.tgz` | MIPS big-endian softfloat |
| mipsle | `tiny-tailscale_<ver>_mipsle.tgz` | MIPS little-endian softfloat |
| mips64 | `tiny-tailscale_<ver>_mips64.tgz` | MIPS64 big-endian |
| mips64le | `tiny-tailscale_<ver>_mips64le.tgz` | MIPS64 little-endian |
| riscv64 | `tiny-tailscale_<ver>_riscv64.tgz` | RISC-V 64-bit |
| geode | `tiny-tailscale_<ver>_geode.tgz` | x86 softfloat (Geode/i486 SoCs) |

## What's different from official tailscale?

### Single combined binary

Each release is a **single `tailscaled` binary** with the CLI built in (`ts_include_cli`). A `tailscale` symlink is included — when invoked as `tailscale` (or with `TS_BE_CLI=1`), it runs as the CLI client. No need for two separate binaries.

### Stripped for size

Features are stripped using tailscale's built-in `cmd/featuretags` tool with `--min`, keeping only what's needed:

Binaries are built with `-s -w` (symbol and DWARF stripping) and `-trimpath`. Feature selection uses dependency resolution — if a kept feature requires another feature, it's automatically included.

<details>
<summary><b>Included features (24 + auto-resolved dependencies)</b></summary>

| Feature | Description |
|---|---|
| cli | Combined CLI embedded into tailscaled binary |
| ssh | Tailscale SSH access |
| serve | Serve and Funnel support |
| taildrop | File sharing between nodes |
| advertiseexitnode | Run an exit node |
| useexitnode | Use exit nodes |
| advertiseroutes | Advertise routes for other nodes |
| useroutes | Use routes advertised by other nodes |
| doctor | Diagnose issues with Tailscale and its host |
| syspolicy | System policy / MDM configuration |
| clientmetrics | Client metrics support (required by syspolicy) |
| cliconndiag | CLI connection error diagnostics |
| colorable | Colorized terminal output |
| relayserver | DERP relay capability |
| osrouter | Configure OS network stack, IPs, and routing tables |
| health | Health checking support |
| portmapper | NAT-PMP/PCP/UPnP port mapping for NAT traversal |
| iptables | Linux iptables firewall support |
| listenrawdisco | Raw sockets for robust NAT traversal |
| cachenetmap | Cache network map to disk for faster restarts |
| wakeonlan | Wake-on-LAN support (wake LAN devices remotely) |
| gro | Generic Receive Offload for better throughput |
| linkspeed | Set TUN device link speed for OS routing/QoS |
| tundevstats | Poll TUN device traffic statistics |

Dependencies auto-resolved: c2n, dbus, netstack, peerapiclient, peerapiserver

</details>

<details>
<summary><b>Omitted features</b></summary>

| Feature | Description |
|---|---|
| ace | Alternate Connectivity Endpoints |
| acme | ACME TLS certificate management |
| appconnectors | App Connectors support |
| aws | AWS integration |
| bakedroots | Embedded CA x509 root certificates |
| bird | Bird BGP integration |
| captiveportal | Captive portal detection |
| capture | Packet capture |
| clientupdate | Client auto-update support |
| cloud | Cloud environment detection |
| completion | CLI shell completion |
| completion_scripts | Embedded CLI shell completion scripts |
| conn25 | Route traffic through connector devices |
| debug | General debug support |
| debugeventbus | Eventbus debug support |
| debugportmapper | Portmapper debug support |
| desktop_sessions | Desktop sessions support |
| dns | MagicDNS and system DNS configuration |
| drive | Tailscale Drive (file server) |
| hujsonconf | HuJSON config file support |
| identityfederation | Auth key gen via identity federation |
| ipnbus | IPN notification bus |
| kube | Kubernetes integration |
| linuxdnsfight | Linux DNS fight detection |
| logtail | Upload logs to log.tailscale.com |
| netlog | Network flow logging |
| networkmanager | Linux NetworkManager integration |
| oauthkey | OAuth secret-to-authkey resolution |
| outboundproxy | Localhost HTTP/SOCKS5 proxy over Tailscale |
| portlist | Advertise listening service ports |
| posture | Device posture checking |
| qrcodes | QR codes in CLI |
| resolved | Linux systemd-resolved integration |
| sdnotify | systemd notification support |
| synology | Synology NAS integration |
| systray | Linux system tray |
| tailnetlock | Tailnet Lock (network key signing) |
| tap | Layer 2 (ethernet) support |
| tpm | TPM support |
| unixsocketidentity | Differentiate LocalAPI users over unix sockets |
| useproxy | Use system proxies to reach Tailscale servers |
| usermetrics | User-facing metrics |
| webclient | Web client UI |
| webbrowser | Open URLs in user's web browser |

</details>

### Static, zero dependencies

All binaries are statically compiled (`CGO_ENABLED=0`). No libc dependency — runs on musl (OpenWrt), glibc (QTI Linux, standard distros), bionic (Android), or any other Linux variant without modification.

## Auto-build system

A GitHub Actions workflow runs daily, checking upstream `tailscale/tailscale` for new stable release tags (even minor version = stable). When a new stable tag is found:

1. Syncs the tag from upstream
2. Generates optimized build tags via `cmd/featuretags --min --add=<kept features>`
3. Cross-compiles for all 10 architectures
4. Creates a GitHub Release with all tarballs

No manual intervention needed — releases appear automatically within 24 hours of an upstream stable release.

## Install

```sh
# Download and extract
tar xzf tiny-tailscale_1.96.4_arm64.tgz

# Copy to system
cp tiny-tailscale_1.96.4_arm64/tailscaled /usr/sbin/
cp -P tiny-tailscale_1.96.4_arm64/tailscale /usr/bin/

# Start
tailscaled &
tailscale up
```

For OpenWrt/embedded systems, these binaries are designed to be packaged into `.ipk` files with proper init scripts. See [quectel-rgmii-toolkit](https://github.com/iamromulan/quectel-rgmii-toolkit) for an example.

## Upstream

This fork tracks [tailscale/tailscale](https://github.com/tailscale/tailscale). No source code is modified — only this README and the CI workflow are added.

- **Issues with tailscale itself** → [tailscale/tailscale issues](https://github.com/tailscale/tailscale/issues)
- **Issues with the tiny builds** → [tiny-tailscale issues](https://github.com/iamromulan/tiny-tailscale/issues)
- **Tailscale docs** → [tailscale.com/kb](https://tailscale.com/kb/)

## Legal

WireGuard is a registered trademark of Jason A. Donenfeld.
