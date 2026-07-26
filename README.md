# tiny-tailscale

Minimal, stripped-down [Tailscale](https://tailscale.com/) combined binaries — auto-built from upstream stable releases.

This repo contains no Tailscale source code — only a CI pipeline and this README. A GitHub Actions workflow automatically builds optimized single-binary releases from upstream stable tags using tailscale's own `cmd/featuretags --min` tool to strip unused features while keeping everything needed for embedded Linux deployments (routers, cellular modems, IoT).

## Downloads

Grab the latest build from [**Releases**](https://github.com/Joetooley28/tiny-tailscale/releases).

| Architecture | File | Notes |
|---|---|---|
| arm64 | `tiny-tailscale_<ver>_arm64.tgz` | ARMv8-A 64-bit (including RM551E-GL) |
| arm | `tiny-tailscale_<ver>_arm.tgz` | ARMv7 32-bit (including RM520N/CFW-3212) |

## What's different from official tailscale?

### Single combined binary

The archive is **ready to use**: it includes the `tailscaled` daemon and the `tailscale` command. They share one underlying executable, but users do not need to rename files, create links, or set `TS_BE_CLI`.

### Stripped for size

Features are stripped using tailscale's built-in `cmd/featuretags` tool with `--min`, keeping only what's needed:

Binaries are built with `-s -w` (symbol and DWARF stripping) and `-trimpath`. Feature selection uses dependency resolution — if a kept feature requires another feature, it's automatically included.

<details>
<summary><b>Included features (26 + auto-resolved dependencies)</b></summary>

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
| unixsocketidentity | Unix socket identity for LocalAPI authentication |
| ipnbus | IPN notification bus; required for interactive browser login URLs |

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
3. Cross-compiles ARMv7 (`arm`) and ARMv8-A 64-bit (`arm64`)
4. Creates one GitHub Release containing both architecture tarballs

No manual intervention needed — releases appear automatically within 24 hours of an upstream stable release.

## Install

Replace `<ver>` with the exact version from the release page and `<arch>` with `arm` for ARMv7/`armv7l` devices such as RM520N/CFW-3212, or `arm64` for ARMv8-A/`aarch64` devices such as RM551E-GL. The version must match in the archive filename and extracted directory; for release `v1.98.9`, use `1.98.9` throughout. Download and install only the archive matching the device.

```sh
# Download and extract
tar xzf tiny-tailscale_<ver>_<arch>.tgz

# Copy to system
cp tiny-tailscale_<ver>_<arch>/tailscaled /usr/sbin/
cp -P tiny-tailscale_<ver>_<arch>/tailscale /usr/bin/

# Start
tailscaled &
tailscale up
```

For a quick start and browser login flow:

```sh
tailscaled >/tmp/tailscaled.log 2>&1 & sleep 1; tailscale up
```

For OpenWrt/embedded systems, these binaries are designed to be packaged into `.ipk` files with proper init scripts. See [quectel-rgmii-toolkit](https://github.com/iamromulan/quectel-rgmii-toolkit) for an example.

### This fork

This fork keeps the build focused on ARM modem targets and adds the `ipnbus` feature that the upstream tiny build omits. `ipnbus` is needed for interactive `tailscale up` to return the browser authorization URL. Use the `arm` artifact for ARMv7 devices such as RM520N/CFW-3212, and the `arm64` artifact for ARMv8-A devices such as RM551E-GL.

## Upstream

This repo is a fork of [tailscale/tailscale](https://github.com/tailscale/tailscale) used purely as a CI and release host. No source code is stored here — the workflow fetches upstream tags directly at build time. This repo contains only the CI workflow, this README, and the upstream license.

- **Issues with tailscale itself** → [tailscale/tailscale issues](https://github.com/tailscale/tailscale/issues)
- **Issues with the tiny builds** → [tiny-tailscale issues](https://github.com/Joetooley28/tiny-tailscale/issues)
- **Tailscale docs** → [tailscale.com/kb](https://tailscale.com/kb/)

## Legal

WireGuard is a registered trademark of Jason A. Donenfeld.
