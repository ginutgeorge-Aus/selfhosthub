# SelfHostHub installer & dashboard: MVP design

- **Date:** 2026-10-02
- **Status:** Draft for review
- **Board:** SHH-1

## 1. Goal

A non-technical person installs one Windows app on an old laptop, clicks **Replace YNAB**, and a
few minutes later opens a working budget app over HTTPS from any phone or PC at home. They never
see a terminal, and the project pays **$0** in running costs.

### Success criteria

- A first-time user goes from download to one working app in **under 20 minutes** on a laptop that
  already has virtualization on, or under 35 minutes when the BIOS change is needed.
- All five MVP apps open over HTTPS, with no certificate warning, from an iPhone, an Android phone
  and a second Windows PC on the same Wi-Fi.
- After a Windows Update reboot, all apps come back without the user touching anything.
- We run no servers.

### Non-goals (MVP)

- Access from outside the home network (Phase 2).
- Automatic backups (later; the dashboard says so plainly).
- Automatic app updates (an **Update** button only).
- Multi-container apps, macOS and Linux hosts.
- **Windows 10.** WSL's always-on (`vmIdleTimeout`) and mirrored networking settings are
  Windows 11 only, and Windows 10 support ended in October 2025. Windows 10 laptops are served
  later by a bootable-USB installer that reuses the same agent and catalog (SHH-11).

## 2. MVP apps

Chosen from the [100-app catalog](../../../research/app-catalog.csv): single container, works
offline, opens in a browser, and needs no client app.

| Card | App | Subdomain | Needs HTTPS |
|---|---|---|---|
| Replace Adobe Acrobat | Stirling-PDF | `pdf` | no |
| Replace YNAB | Actual Budget | `budget` | **yes** (browser secure-context APIs) |
| Replace Plex | Jellyfin | `tv` | no |
| Replace Todoist | Vikunja | `tasks` | no |
| Replace Spotify (your music) | Navidrome | `music` | no |

Media cards say plainly that they play **your own** files and don't replace the content catalog.

## 3. Architecture

```
Windows host
├─ SelfHostHub.exe (Tauri: Rust + web UI)
│   ├─ Preflight wizard, dashboard, tray icon
│   ├─ Windows-side work: features, BIOS reboot, WSL import, port forwarding, keep-alive
│   └─ Calls hub-agent over http://127.0.0.1:<agent-port> with a per-install token
└─ WSL2 distro "selfhosthub" (our minimal Debian rootfs)
    ├─ hub-agent (Go): app lifecycle, certificates, DNS updates, health checks
    ├─ Docker Engine (not Docker Desktop)
    ├─ Caddy: <sub>.<name>.duckdns.org → app container, wildcard TLS
    └─ App containers

External, free, third-party: DuckDNS (default) or deSEC (backup), and Let's Encrypt
```

**Split of responsibilities:** the Windows app handles everything that needs Windows APIs or admin
rights. The agent handles everything that happens inside Linux. The two talk only through the
agent's local HTTP API, so the agent can be developed and tested on any Linux machine.

### 3.1 Components

| Unit | Does | Depends on |
|---|---|---|
| `app/` preflight | Detects the system, enables WSL features, handles the BIOS reboot, resumes | Windows CIM, `dism`, `wsl`, `shutdown` |
| `app/` dashboard | Shows cards, install progress, QR codes and errors | Agent API |
| `app/` host service | Port forwarding, firewall rule, startup task, power settings | `netsh`, `schtasks`, `powercfg` |
| `agent/catalog` | Loads and validates manifests | `catalog/*.yaml` |
| `agent/apps` | Install, start, stop, update, uninstall | Docker Engine |
| `agent/proxy` | Generates the Caddy config from installed apps | Caddy admin API |
| `agent/dns` | `DnsProvider` interface, DuckDNS + deSEC implementations | Provider HTTP APIs |
| `agent/certs` | ACME DNS-01 issue and renewal of the wildcard certificate | `agent/dns`, Let's Encrypt |
| `agent/api` | Local HTTP API with token auth | All of the above |
| `distro/` | Builds the rootfs tarball in CI | Debian, Docker, Caddy, agent binary |

## 4. Preflight

The order matters: WSL installs fine with virtualization off but then fails with `0x80370102`.

1. **Detect** (no admin needed):
   - Windows 11 22H2 or later (build ≥ 22621). On Windows 10, stop with "SelfHostHub needs
     Windows 11. A USB installer for this laptop is coming", plus a link to sign up for news.
   - CPU supports VT-x/AMD-V and SLAT (`Win32_Processor`). If not, stop with a plain-English
     "this processor can't run SelfHostHub".
   - Virtualization on: check `Win32_ComputerSystem.HypervisorPresent` **first**, then
     `Win32_Processor.VirtualizationFirmwareEnabled`. The firmware flag reads false whenever Windows'
     own hypervisor is already running.
   - WSL state from `wsl --status`.
   - RAM: warn under 8 GB. Disk: stop under 40 GB free.
2. **Enable features** with one UAC prompt: `dism /enable-feature` for
   `Microsoft-Windows-Subsystem-Linux` and `VirtualMachinePlatform` (`/norestart`), then install
   the bundled WSL MSI. No Microsoft Store dependency.
3. **One reboot.** Write `HKCU\...\RunOnce` → `SelfHostHub.exe --resume` first.
   - Virtualization on: normal reboot.
   - Virtualization off: show a guide matched to the laptop maker and CPU vendor (Intel "VT-x" or
     AMD "SVM Mode"), with a **QR code** to open the guide on a phone. Then
     `shutdown /r /fw /t 0` to boot straight into firmware setup. If `/fw` fails (legacy BIOS),
     show the power-on key for that maker (Dell/Lenovo F2, HP F10, Acer F2, ASUS F2/Del).
4. **Resume:** re-run detect. If virtualization is still off, offer one retry; after the second
   failure, show the help page (no loop). Then run `wsl --update` and `wsl --set-default-version 2`.
5. **Import:** `wsl --import selfhosthub %LOCALAPPDATA%\SelfHostHub\wsl rootfs.tar.gz --version 2`,
   then wait until the agent's `/health` reports OK and `docker info` succeeds.

**Later:** OEM WMI tools (Dell Command | Configure, HP BCU, Lenovo WMI) can switch virtualization
on with no BIOS visit.

## 5. Networking & HTTPS

### 5.1 Name and certificate

- In the wizard, the user picks a name, signs in to DuckDNS (Google, GitHub or other login) and
  pastes the token. **Advanced → Use deSEC instead** is the alternative.
- The agent points `<name>.duckdns.org` at the laptop's LAN IP. Every subdomain resolves to the same IP.
- **Wildcard only:** the agent requests one certificate for `*.<name>.duckdns.org` through DNS-01.
  DuckDNS holds one TXT value at a time, so the bare name is not included. The dashboard lives at
  `home.<name>.duckdns.org`.
- The private key is generated in the agent and never leaves the laptop. Renewal happens at 60
  days, with a dashboard warning 14 days before expiry if renewal keeps failing.
- When the LAN IP changes, the agent calls `SetIP` within 5 minutes.

```go
type DnsProvider interface {
    SetIP(ip string) error
    SetTXT(name, value string) error
    ClearTXT(name string) error
}
```

**Confirmed 2026-10-02:** `duckdns.org` and `dedyn.io` are both on the Public Suffix List, so each
user's name counts separately against Let's Encrypt's rate limits
([viability research](../../../research/viability-2026-10.md)). DuckDNS has a history of multi-hour
outages, so the wizard offers to set up deSEC as an automatic fallback.

### 5.2 Reaching the apps from other devices

- WSL `networkingMode=mirrored` in `.wslconfig` (Windows 11 22H2+), plus an inbound firewall
  rule for the Private profile only. If mirrored mode fails its self-test, fall back to `netsh
  interface portproxy` for 80/443 and refresh it whenever the WSL IP changes.
- **"Can my phone reach it?" check:** the dashboard resolves the name through the router's DNS.
  If the router hides home-network answers (DNS-rebinding protection), it shows a fix guide for
  common routers and the IP fallback links.
- **Fallback:** if DNS can't be reached, the dashboard shows `http://<LAN-IP>:<port>` links
  (Actual Budget is marked "needs internet name").

### 5.3 Offline behavior

Installing an app needs internet. Running apps need no internet while the certificate is valid
(up to 90 days), except for the DNS lookup of the name itself; that's why the fallback links
exist.

## 6. App catalog

One YAML manifest per app in `catalog/`. CI builds a signed `catalog/index.json` from them.

```yaml
id: actual
name: Actual Budget
replaces: [YNAB, Monarch Money]
pitch: "Plan your money. Your data stays on your laptop."
subdomain: budget
image: actualbudget/actual-server:<pinned>
port: 5006
ram_mb: 512
needs_https: true
data: [/data]
health: { path: /, expect: 200, timeout_s: 60 }
media_folders: []
```

**Validation rules** (in CI and in the agent at load time):
- The image tag is pinned; `latest` is rejected.
- The subdomain is unique.
- `ram_mb` is set.
- Exactly one container.

## 7. Install flow

1. Check resources: free RAM against `ram_mb`, and free disk against image size plus 1 GB.
2. Pull the image, with progress shown in MB.
3. Create `/srv/shh/apps/<id>/` and start the container. Media folders the user picked are mounted
   read-only from `/mnt/c/...`.
4. Add the Caddy route `<sub>.<name>.duckdns.org`.
5. Poll the health check, then flip the card to **Open** and show a QR code.

**Uninstall** asks "Keep my data?" with **Yes** as the default.

## 8. Errors

Each error the user can see has a plain-English message and a next action.

| Cause | Message | Action |
|---|---|---|
| No internet during install | "Connect to the internet to install. Apps work offline afterwards." | Try again |
| Disk full | "Needs 2 GB free. You have 0.8 GB." | Open Storage settings |
| Health check timeout | "Actual Budget didn't start." | Restart · Copy details for help |
| DNS provider down | (silent) automatic failover to deSEC if configured; otherwise "Couldn't reach DuckDNS. Apps still work at …" | Retry · Set up deSEC |
| Renewal failing | Warning 14 days before expiry | Fix now |
| Agent not responding | "SelfHostHub's engine stopped." | Restart engine |

"Copy details for help" gathers versions, the last 200 log lines and preflight results, with
tokens and IPs redacted.

## 9. Keep-alive

- A scheduled task at system startup (whether or not anyone is logged on) runs `wsl -d selfhosthub`,
  and the agent starts its apps.
- `.wslconfig`: `vmIdleTimeout=-1`. This is **not trusted on its own**, because WSL has been reported
  to stop services anyway (microsoft/WSL#13291). A Windows-side **watchdog** in the host service checks
  the agent's `/health` every 30 s and restarts the distro after 3 misses.
- `powercfg`: no sleep on AC power, and closing the lid does nothing on AC. The user's settings are
  restored on uninstall.
- Target: all apps healthy within 2 minutes of the laptop powering on.
- **Battery care:** a laptop on charge 24/7 wears its battery and can swell it. The wizard detects
  the maker and shows the charge-limit step: Lenovo Vantage "Conservation mode", Dell "Primarily AC
  use", ASUS "Battery Health Charging", HP "Battery Care". Where a charge limit can be set from
  Windows (WMI), offer a one-click button.
- **Power modes** (Settings):
  - **Always on** (default)
  - **Daytime only**: sleep from 1am to 6am by default, user-adjustable, with a wake timer through
    a scheduled task with `WakeToRun`. Apps are unreachable while the laptop sleeps, and the dashboard
    says so.
  - No Wake-on-LAN: it's unreliable over Wi-Fi.
- **Placement tip** in the wizard: plugged in, lid closed is fine, keep it somewhere with airflow.

## 10. Updates

| What | Mechanism |
|---|---|
| Windows app | Tauri updater from GitHub Releases, with a "restart to update" prompt |
| Distro | Shipped with app releases; replaced in place, app data kept |
| Catalog | Signed `index.json` fetched daily |
| Apps | An **Update** button appears after CI passes for a new pinned tag. The data folder is copied first so a failed update can be rolled back. |

**Code signing:** apply to the SignPath Foundation for free OSS signing. Without it, SmartScreen
blocks non-technical users.

## 11. Security

- The agent API listens on `127.0.0.1` only and requires a random token stored in the user's profile.
- The dashboard is not exposed on the LAN in the MVP; only app subdomains are.
- The firewall rule covers the Private profile only.
- The DNS token and TLS key are stored with Windows DPAPI on the host and as `0600` files inside the distro.
- No router port forwarding, ever, in the MVP.

## 12. Testing

| Layer | What | Where |
|---|---|---|
| Agent unit | Manifest validation, Caddy config generation, `DnsProvider` against a fake server, renewal timing | CI, every PR |
| Agent integration | Boot the agent with Docker, install all 5 apps, health checks, Let's Encrypt **staging** certificate through a test DuckDNS name (secret) | CI Linux, every PR |
| Preflight unit | Table-driven tests over faked CIM/`wsl` outputs (hypervisor present with the firmware flag false, legacy BIOS, old build, …) | CI Windows (no WSL needed) |
| Manual release checklist | Win11 22H2+ on ≥2 laptop models; Intel + AMD; virtualization off → on; reboot survival; iPhone + Android + PC open every app over HTTPS | Real laptops; GitHub runners can't run WSL2 |

## 13. Build order

Each milestone can be demonstrated on its own.

1. Agent on plain Linux: install, open and stop the 5 apps over the API; Caddy over HTTP.
2. HTTPS: DuckDNS, then deSEC, the wildcard certificate and renewal.
3. Distro image built in CI and imported by hand on Windows.
4. Preflight wizard.
5. Dashboard.
6. Keep-alive, watchdog and networking (mirrored mode, portproxy fallback, phone-reach check).
7. Beta: signed installer, about 5 testers on real old laptops.

## 14. Repo layout

```
agent/      Go service (hub-agent)
app/        Tauri app (src/ = web UI, src-tauri/ = Rust)
distro/     rootfs build scripts
catalog/    app manifests + generated index.json
docs/       user docs + specs
research/   app catalog research
```

## 15. Open items

- Confirm that Actual Budget works behind Caddy on a subdomain with no further headers.
- SignPath Foundation application: start alongside milestone 1, because approval takes weeks.
- Backups are a non-goal for the MVP, but the viability research recommends pulling a simple USB
  backup into the beta. Decide before SHH-10.
- Validate with about 20 real non-technical users before growing the catalog beyond 5 apps.
