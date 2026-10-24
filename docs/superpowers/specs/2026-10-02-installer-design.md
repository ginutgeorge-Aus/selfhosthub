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
| Replace Smallpdf / iLovePDF | BentoPDF | `pdf` | no (files processed in the browser; run its air-gapped build with bundled WASM, not the CDN default) |
| Replace YNAB | Actual Budget | `budget` | **yes** (browser secure-context APIs) |
| Replace Plex | Jellyfin | `tv` | no |
| Replace Todoist | Vikunja | `tasks` | no |
| Replace Spotify (your music) | Navidrome | `music` | no |

BentoPDF was picked over Stirling-PDF because it has a simpler tile layout and uses tens of MB of
RAM instead of 1–2 GB. Stirling-PDF (OCR, redaction) becomes a later "PDF Pro" app.

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
    ├─ Docker Engine (not Docker Desktop), started by systemd (`/etc/wsl.conf` sets `[boot] systemd=true`)
    ├─ Caddy: <sub>.<zone> → app container, wildcard TLS (<zone> = the selected provider's name, e.g. <name>.duckdns.org or <name>.dedyn.io)
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
  DuckDNS holds one TXT value at a time, so the bare name is not included. The wildcard covers app
  subdomains only. The dashboard is the desktop app, not a LAN web page (see section 11).
- The private key is generated in the agent and never leaves the laptop. Renewal starts when
  one third of the certificate's lifetime remains (about 30 days for a 90-day certificate, and it
  tracks Let's Encrypt's shorter lifetimes as they roll out), retried daily. The dashboard warns
  14 days before expiry, or at half the remaining lifetime if that is sooner, when renewal keeps
  failing.
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
outages. deSEC cannot take over a `duckdns.org` name, because it only controls `dedyn.io`. So deSEC
is an alternative provider chosen at setup (a different hostname and certificate), not an automatic
failover. During a DuckDNS outage, apps with a LAN-IP fallback link stay reachable (5.2); Actual
Budget does not.

### 5.2 Reaching the apps from other devices

- WSL `networkingMode=mirrored` in `.wslconfig` (Windows 11 22H2+), plus an inbound Windows Firewall rule for the Private profile, limited to the local subnet and TCP
  80/443 plus the app fallback ports. In the Hyper-V firewall it adds scoped allow rules
  (`New-NetFirewallHyperVRule` with `-Direction Inbound -Action Allow -LocalPorts ... -VMCreatorId
  '{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}'`) for those ports only, never a creator-wide default
  allow, because that ID covers every WSL distro on the machine. If the active network is
  classified **Public**, preflight keeps it Public by default. It offers to switch it to Private
  only after the user confirms, in plain words, that this is a trusted home network (never hotel or
  office Wi-Fi), and it never opens the Public profile. The Hyper-V rules are port-scoped, not
  distro-scoped, and WSL distros share one network namespace. Preflight therefore stops if another
  distro already listens on an allowed port, and tells the user that any distro that later binds
  one of those ports would be reachable from the LAN. If mirrored mode fails its self-test, fall
  back to `netsh interface portproxy` with one mapping for each exposed port (80, 443 and every app
  fallback port) and refresh them all whenever the WSL IP changes.
- **"Can my phone reach it?" check:** run from the phone, not the host. The dashboard shows a QR code
  for the app URL; the agent confirms success when Caddy logs a request from another LAN address.
  If nothing arrives, it shows the likely causes (router DNS-rebinding protection, Wi-Fi client
  isolation, firewall) with a fix guide for common routers and the IP fallback links.
- **Fallback:** Caddy also serves each app over plain HTTP on its own LAN port (for example
  `http://<LAN-IP>:8101`), opened in the same firewall rules, so the links work when DNS can't be
  reached. Actual Budget needs a secure context, so it has no fallback and is marked "needs
  internet name" (accepted MVP limitation). The plain-HTTP fallback is LAN-only (Private profile,
  local subnet, no router port forwarding), but anyone on the home network could read its traffic,
  including logins and files, so the dashboard labels every fallback link "not encrypted, use only
  on your home network". This is an accepted trade-off: without it, a DNS outage or a router with
  DNS-rebind protection would take every app offline.

### 5.3 Offline behavior

Installing an app needs internet. Running apps need no internet while the certificate is valid
(up to 90 days), except for the DNS lookup of the name itself; that's why the fallback links
exist. Actual Budget has no fallback link, so it needs its hostname to resolve; it is the one app
that can stop working during a DNS outage (accepted MVP limitation, issue #4).

**Before beta (decided 2026-10-25):** an opt-in **backup name** under Advanced. The user also
registers a deSEC `<name>.dedyn.io`; the agent keeps both A records and both wildcard certificates,
and the dashboard shows a "backup link" for each app. This is the only option that works on phones
with no router or device changes and stays at $0. Rejected: a local DNS resolver on the laptop (needs
a router DHCP change, and the whole house loses DNS when the laptop sleeps) and hosts-file guidance
(phones can't edit hosts files). Catch: browser data is per-origin, so switching to the backup link
means signing in again and re-syncing from the server, which holds the data. Both providers down at
once is still unhandled.

## 6. App catalog

One YAML manifest per app in `catalog/`. CI builds a signed `catalog/index.json` from them.

```yaml
id: actual
name: Actual Budget
replaces: [YNAB, Monarch Money]
pitch: "Plan your money. Your data stays on your laptop."
subdomain: budget
image: actualbudget/actual-server@sha256:<digest>   # tag kept in a comment for humans
port: 5006
ram_mb: 512
needs_https: true
data: [/data]
health: { path: /, expect: 200, timeout_s: 60 }
media_folders: []
```

**Validation rules** (in CI and in the agent at load time):
- The image is pinned by immutable digest (`@sha256:...`); tags, including `latest`, are rejected.
  CI resolves and records the digest, so the signed catalog covers the exact image bytes.
- The subdomain is unique.
- `ram_mb` is set.
- Exactly one container.

## 7. Install flow

1. Check resources: free RAM against `ram_mb`, and free disk against image size plus 1 GB.
2. Pull the image, with progress shown in MB.
3. Create `/srv/shh/apps/<id>/` and start the container. Media folders the user picked are mounted
   read-only from `/mnt/c/...`.
4. Add the Caddy route `<sub>.<zone>`, where `<zone>` is the hostname of the DNS provider chosen at setup (DuckDNS or deSEC). Routes, certificate names, QR links and fallback links are all generated from it.
5. Poll the health check, then flip the card to **Open** and show a QR code.

**Uninstall** asks "Keep my data?" with **Yes** as the default.

## 8. Errors

Each error the user can see has a plain-English message and a next action.

| Cause | Message | Action |
|---|---|---|
| No internet during install | "Connect to the internet to install. Apps work offline afterwards, except Actual Budget, which needs its web name to resolve." | Try again |
| Disk full | "Needs 2 GB free. You have 0.8 GB." | Open Storage settings |
| Health check timeout | "Actual Budget didn't start." | Restart · Copy details for help |
| DNS provider down | "Couldn't reach DuckDNS. Apps with fallback links still work at …" (LAN-IP links) | Retry · Switch to deSEC |
| Renewal failing | Warning 14 days before expiry | Fix now |
| Agent not responding | "SelfHostHub's engine stopped." | Restart engine |

"Copy details for help" gathers versions, the last 200 log lines and preflight results, with
tokens and IPs redacted.

## 9. Keep-alive

- A scheduled task registered for the installing Windows user (the distro and `.wslconfig` are
  per-user), triggered at startup and at logon, runs elevated, uses the S4U logon type (so it runs
  whether or not that user is logged on, without storing a password), and does two steps in order:
  1. `wsl --mount --vhd %LOCALAPPDATA%\SelfHostHub\data.vhdx --bare` re-attaches the data disk,
     because WSL detaches mounted disks whenever it shuts down.
  2. `wsl -d selfhosthub` starts the distro. A systemd mount unit mounts the disk by filesystem
     label at `/srv/shh`, and Docker and the agent start only after that mount succeeds
     (`RequiresMountsFor=/srv/shh`), so apps never write to an unmounted directory.
- `.wslconfig` is user-global, so it affects every WSL distro the user has. The installer backs up
  the existing file, merges only the keys it needs (`vmIdleTimeout=-1`, `networkingMode=mirrored`),
  tells the user if other distros exist, and restores the original on uninstall.
- `vmIdleTimeout=-1` is **not trusted on its own**, because WSL has been reported to stop services
  anyway (microsoft/WSL#13291). A Windows-side **watchdog** in the host service checks the agent's
  `/health` every 30 s and restarts the distro (re-attaching the data disk first) after 3 misses.
- `powercfg` (**Always on** mode only): no sleep on AC power, and closing the lid does nothing on AC.
  Switching to **Daytime only** restores the user's original policy and applies the sleep schedule.
  The user's original settings are restored on uninstall.
- Target: all apps healthy within 2 minutes of the laptop powering on.
- **Battery care:** a laptop on charge 24/7 wears its battery and can swell it. The wizard detects
  the maker and shows the charge-limit step: Lenovo Vantage "Conservation mode", Dell "Primarily AC
  use", ASUS "Battery Health Charging", HP "Battery Care". Where a charge limit can be set from
  Windows (WMI), offer a one-click button.
- **Power modes** (Settings):
  - **Always on** (default)
  - **Daytime only**: sleep from 1am to 6am by default, user-adjustable, with a wake timer through
    a scheduled task with `WakeToRun`. `WakeToRun` is unreliable on Modern Standby (S0) laptops, so
    preflight reads `powercfg /a` and offers this mode only when S3 sleep is available; otherwise the
    option is shown as unavailable with the reason. Apps are unreachable while the laptop sleeps, and
    the dashboard says so.
  - No Wake-on-LAN: it's unreliable over Wi-Fi.
- **Placement tip** in the wizard: plugged in, lid closed is fine, keep it somewhere with airflow.

## 10. Updates

| What | Mechanism |
|---|---|
| Windows app | Tauri updater from GitHub Releases, with a "restart to update" prompt |
| Distro | Shipped with app releases. App data lives on a second WSL-attached VHDX (`%LOCALAPPDATA%\SelfHostHub\data.vhdx`, mounted at `/srv/shh`), separate from the replaceable system distro, so replacing the distro keeps it |
| Catalog | Signed `index.json` fetched daily |
| Apps | An **Update** button appears after CI passes for a new pinned tag. The agent stops the app's container first, so databases such as Vikunja's SQLite are quiescent, then copies the data folder and verifies the copy before replacing the image. A failed update restores that copy. |

**Code signing:** apply to the SignPath Foundation for free OSS signing. Without it, SmartScreen
blocks non-technical users.

## 11. Security

- The agent API listens on `127.0.0.1` only and requires a random token stored in the user's profile.
- The dashboard is the desktop app and talks to the agent over `127.0.0.1`. No dashboard route exists
  in Caddy, so it is not exposed on the LAN in the MVP; only app subdomains are.
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

- Verify the startup task: WSL's support for starting a distro with no interactive user session is
  unconfirmed (microsoft/WSL#7022). Prototype on real laptops; fall back to an at-logon trigger plus
  Windows auto-logon guidance if S4U does not work. Tracked in issue #5.
- Confirm that BentoPDF's air-gapped build works fully offline (no CDN fetches).
- ~~Confirm that Actual Budget works behind Caddy on a subdomain with no further headers.~~ Resolved
  (SHH-2). Tested `actualbudget/actual-server:26.10.0` behind `caddy:2.11.7` on a `tls internal`
  subdomain in headless Chromium: the app loads, the demo budget opens, and `isSecureContext` and
  `crossOriginIsolated` are both `true`. No extra headers are needed. actual-server sets its own
  COOP/COEP headers and Caddy passes them through. The Caddyfile is just:
  ```
  budget.<name>.duckdns.org {
      reverse_proxy actual:5006
  }
  ```
  Do not add header overrides or strip headers in the generated config. The only console error was a
  service worker certificate error, caused by the untrusted self-signed test cert. Not re-tested with
  a real Let's Encrypt cert.
- SignPath Foundation application: start alongside milestone 1, because approval takes weeks.
- Backups are a non-goal for the MVP, but the viability research recommends pulling a simple USB
  backup into the beta. Decide before SHH-10.
- Build the opt-in deSEC backup name for Actual Budget before beta (5.3, issue #4).
- Validate with about 20 real non-technical users before growing the catalog beyond 5 apps.
