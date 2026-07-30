# MyNet

MyNet is an open-source, self-hosted VPN application written in Go. It creates a secure virtual private network that allows computers behind NATs and firewalls to connect and communicate using MyNet-assigned IP addresses. With MyNet, you can ping, SSH, host game servers, and access services across devices without requiring public IPs or complex firewall configurations.

**Website:** [mynet.crosmakesgames.com](https://mynet.crosmakesgames.com)  
**Current Version:** 0.2.0

---

## 1. Overview

MyNet uses a **relay-first** architecture:

* **`mynet`**: The normal app for regular users. Host and join networks. Relay mode is the **default** — no port forwarding needed. P2P (direct) mode is opt-in.
* **`mynet-relay`**: The relay server host. Run this on a public VPS to relay traffic between hosts and clients.

**Default relay:** `relay-na1.crosmakesgames.com:8443`

---

## 2. Installation

### Windows — GUI Installer (Wizard)
1. Download **`MyNet-Setup.exe`** and double-click it.
2. The install wizard walks you through: Welcome → License → Components → Directory → Start Menu → Install → Finish.
3. Choose what to install: MyNet (required), MyNet Relay (optional), Desktop shortcut (optional), Windows Service (optional).
4. MyNet is added to your system PATH — run `mynet` from any terminal.

### Windows — Command-Line Install (No GUI)
```powershell
# Interactive install
.\install-windows.ps1

# Silent install (no prompts, no output)
.\install-windows.ps1 -Silent

# Install everything (relay + service + desktop shortcut)
.\install-windows.ps1 -Silent -Relay -Service -Desktop

# Custom install directory
.\install-windows.ps1 -InstallDir "C:\MyNet"

# Uninstall
.\install-windows.ps1 -Uninstall
```

### Linux
```bash
sudo ./install-linux.sh
```
Copies binaries to `/usr/local/bin`, optionally installs systemd services.

### From Source
Requires Go 1.21+:
```bash
make all          # builds mynet + mynet-relay for Windows and Linux
make installer    # builds MyNet-Setup.exe wizard (needs NSIS 3.x)
make package      # builds everything + creates distribution zip
```

### Service Install (After Binary Install)
```bash
mynet install              # install as system service
mynet install --quiet      # silent install
mynet-relay install        # install relay as service
```

### Uninstall
```bash
# Windows GUI: Add/Remove Programs in Settings, or:
.\install-windows.ps1 -Uninstall

# Windows command-line:
mynet uninstall --full     # removes service + files + PATH + registry

# Linux:
sudo ./uninstall-linux.sh
# or
mynet uninstall --full
```

---

## 3. Quick Start

### Host a Network (Relay Mode — Default)
```bash
mynet host relay
```
Uses the default relay (`relay-na1.crosmakesgames.com:8443`). Prompts for confirmation if no relay is specified. Outputs a **network code** like `ABC123`.

### Join a Network (Relay Mode — Default)
```bash
mynet join relay ABC123
```
Uses the default relay. Or specify a different relay:
```bash
mynet join relay ABC123 relay-na1.crosmakesgames.com:8443
```

### Host P2P (Direct, needs port forwarding)
```bash
mynet start p2p
```

### Join P2P
```bash
mynet join p2p 192.168.1.50
```

### Check Status
```bash
mynet status    # connection status
mynet ip         # your MyNet IP
mynet devices    # list devices on the VPN
```

---

## 4. Commands

### Normal App (`mynet`)

| Command | Description |
|:---|:---|
| `mynet host relay [relay_addr]` | Host a network via relay (default). Outputs a network code. |
| `mynet start p2p` | Host via direct P2P (requires port forwarding or UPnP). |
| `mynet join relay <code> [relay_addr]` | Join a network via relay using the code. |
| `mynet join p2p <ip>` | Join via direct P2P connection. |
| `mynet stop` | Stop hosting. |
| `mynet leave` | Disconnect from a network. |
| `mynet remove` | Full cleanup (stop + leave + firewall + state). |
| `mynet status` | Show connection status. |
| `mynet ip` | Show your MyNet IP address. |
| `mynet devices` | List devices on the VPN. |
| `mynet ping <ip>` | Ping a device on the VPN. |
| `mynet ssh <user@ip>` | SSH into a device. |
| `mynet password set <pw>` | Set a join password (while hosting). |
| `mynet password off` | Disable password requirement. |
| `mynet ban <device_id>` | Ban a device from the network. |
| `mynet unban <device_id>` | Unban a device. |
| `mynet promote <device_id>` | Promote a device to admin. |
| `mynet demote <device_id>` | Demote an admin device. |
| `mynet version` | Show version info. |
| `mynet install [--quiet]` | Install as system service. |
| `mynet uninstall [--full]` | Remove service (--full removes files/PATH/registry too). |
| `mynet help` | Show help. |

**Flags for host/join:**
- `--password <pw>` — set or provide a join password
- `--name <name>` — override the device/server name
- `--daemon` / `-d` — run in background
- `--quiet` / `-q` — silence all output (uses default relay without prompting)

### Relay Host App (`mynet-relay`)

| Command | Description |
|:---|:---|
| `mynet-relay host [max_networks]` | Start relay server (default: 50 networks max). |
| `mynet-relay host 100` | Start with 100 network limit. |
| `mynet-relay stop` | Stop relay server. |
| `mynet-relay remove` | Stop + cleanup firewall. |
| `mynet-relay status` | Show relay status. |
| `mynet-relay install [--quiet]` | Install as system service. |
| `mynet-relay uninstall` | Remove system service. |
| `mynet-relay version` | Show version. |
| `mynet-relay help` | Show help. |

**Flags for relay host:**
- `--addr <addr>` — listen address (default: `0.0.0.0`)
- `--port <port>` — TCP control port (default: `8443`)
- `--data-port <port>` — UDP data port (default: `8444`)
- `--daemon` / `-d` — run in background
- `--quiet` / `-q` — silence all output

---

## 5. How It Works

### Relay Mode (Default)
Both the host and clients connect **outbound** to the relay server. No port forwarding needed on either side. The relay bridges TCP control tunnels and forwards UDP data packets.

1. Host runs `mynet host relay` → connects to relay, gets a network code
2. Clients run `mynet join relay <code>` → connect to the same relay
3. Relay bridges the connection; STUN hole-punching may establish direct P2P for lower latency

### P2P Mode
Direct host-to-client connection. Host listens on TCP 8443 + UDP 8444. Requires port forwarding or UPnP.

1. Host runs `mynet start p2p` → listens on ports, shows public IP
2. Clients run `mynet join p2p <ip>` → connect directly

### Default Relay
When no relay address is specified, MyNet uses `relay-na1.crosmakesgames.com:8443`. The user is prompted to confirm (unless `--quiet` is used).

---

## 6. Architecture

- **Relay-first NAT traversal** — works behind double NAT, CGNAT, and strict firewalls
- **WinTun** on Windows, **`/dev/net/tun`** on Linux
- **SQLite** for device/state management
- **STUN** for P2P hole punching (optional, in relay mode)
- **UPnP** for automatic port forwarding (in P2P mode)
- **IP range:** `100.64.0.0/10` (CGNAT range, RFC 6598)
- **Max networks per relay:** configurable (default 50)

---

## 7. Building from Source

```bash
make all          # build for Windows + Linux
make build-windows  # Windows only
make build-linux    # Linux only
make installer      # NSIS installer (needs makensis)
make clean          # remove build artifacts
make test           # run tests
```

---

## License

MyNet is open-source software licensed under the MIT License.
