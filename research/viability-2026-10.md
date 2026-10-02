# SelfHostHub viability research (as of 2026-10-02)

Method: live web search/fetch plus direct GitHub API calls and a direct download of the Public Suffix List (PSL). Stars/pushed dates are from api.github.com on 2026-10-02. "n/v" = not verified this session (GitHub API rate-limited mid-run or page unreadable). Search-engine summaries are lower confidence than fetched primary pages; those are flagged.

## 1. Competitor table

| Name | Platform | Windows? | Target user | Remote/HTTPS | Model | Status 2026 | Stars | Source |
|---|---|---|---|---|---|---|---|---|
| CasaOS | Linux app layer (installs on Debian/Ubuntu) | Only via DIY WSL2 guide (wiki: Linux-only; needs systemd, own Docker Engine; "storage mgmt not possible in WSL2") | Hobbyist / semi-technical | No built-in HTTPS; LAN + 3rd-party | Free OSS (Apache-2.0); IceWhale sells hardware | Maintenance mode. Repo pushed 2026-09-28 but latest core release v0.4.15 = 2024-12-19; app store v2.0.0 Jul 2026; IceWhale focus moved to ZimaOS | 37,292 | https://github.com/IceWhaleTech/CasaOS ; https://wiki.casaos.io/en/guides/running-casaos-on-windows-with-wsl2 ; https://shop.zimaspace.com/pages/is-casaos-abandoned-development-status |
| ZimaOS | Dedicated OS (NAS-focused) | No (bare metal/VM) | Semi/non-technical | Remote via vendor tooling (n/v) | Free OS + ZimaBoard/Cube hardware | Active; pushed 2026-09-30 | 2,999 | https://github.com/IceWhaleTech/ZimaOS |
| Umbrel / umbrelOS | Dedicated OS (Debian-based); Umbrel Home hardware | No official Windows support (community WSL2 attempts report instability); supports Pi 4/5, AMD64, Umbrel Home | Non-technical to hobbyist | Tor / Tailscale / local (app-specific) | Free source-available (PolyForm Noncommercial) + Umbrel Home $399 | Very active. umbrelOS 2.0 announced 2026-09-22; repo pushed 2026-09-22 | 12,230 | https://community.umbrel.com/c/announcements/12.md ; https://github.com/getumbrel/umbrel ; https://community.umbrel.com/t/umbrel-on-wsl2-with-windows-11/11681 ; https://umbrel.com/support/getting-started/what-is-umbrel (price via search) |
| Runtipi | App layer on Linux server (Ubuntu 22.04+ / Pi) | Not officially (docs list 64-bit Linux) | Beginner/hobbyist | Built-in Traefik + Let's Encrypt (needs real domain) | Free OSS (GPL-3.0) | Active; pushed 2026-10-01 | 9,679 | https://github.com/runtipi/runtipi ; https://runtipi.io/docs/quick-start |
| Cosmos Cloud / Cosmos Server | Docker app on Linux (also container) | Not a supported path (n/v) | Technical-leaning | Reverse proxy + auto HTTPS via Let's Encrypt, SmartShield | Free OSS + paid "Cosmos Cloud" tier (a mirror listing shows from ~$2.48/mo promo, low confidence) | Active; pushed 2026-09-19; 568 releases | 6,171 | https://github.com/azukaar/Cosmos-Server ; https://releasealert.dev/github/azukaar/Cosmos-Server |
| YunoHost | Debian-based OS/layer | No (Debian VM possible) | Semi-technical, self-host communities | Needs own domain; built-in Let's Encrypt; free *.nohost.me style subdomains | Free OSS (AGPL), donations | Active; pushed 2026-10-02 | 2,977 | https://github.com/YunoHost/yunohost ; https://yunohost.org/donate.gl.html |
| Cloudron | Layer on Ubuntu server | No | Small orgs/semi-technical | Needs domain; auto-cert | Freemium: Free 2 apps; Pro EUR15/mo (monthly) or EUR30 (yearly); Max EUR25/EUR50 | Active commercial (n/v on releases) | n/a (partly closed) | https://www.cloudron.io/pricing.html |
| Sandstorm | Platform on Linux | No | Technical | Wildcard DNS (own domain/sandcats) | Free OSS; community-run | Community-maintained; pushed 2026-09-27 | 7,079 | https://github.com/sandstorm-io/sandstorm |
| StartOS (Start9) | Dedicated OS | No | Privacy/Bitcoin-oriented, semi-technical | Tor, VPN, Start Tunnel, clearnet options | Free OSS (MIT) + hardware: 2026 Server One $749-$1,049 incl. support | Active; 0.4.0 released; 2026 hardware | n/v (repo moved) | https://store.start9.com/products/server-one-2026 ; https://hackernoon.com/solo-satoshi-becomes-start9s-first-us-distributor-bringing-sovereign-computing-home |
| FreedomBox | Debian-based OS | No | Privacy/community | Built-in; domain-based | Free OSS (AGPL) | Active; 26.3 released 2026-02-12 | n/v (salsa.debian) | https://tracker.debian.org/pkg/src:freedombox ; https://discuss.freedombox.org/latest?page=1 |
| HexOS | OS/UX layer on TrueNAS | No (bare-metal install) | Explicitly non-technical | Remote mgmt via deck.hexos.com | Paid lifetime licence: $199 early access, $299 regular | Early access/beta ("1.0 launching soon") | closed | https://hexos.com/ |
| Unraid | Dedicated OS | No | Hobbyist/prosumer | Own tooling (Unraid Connect) / reverse proxy | Paid licence ~$49-$249 (per third-party 2026 comparison) | Active (7.3.x) | closed | https://tech-insider.org/?p=22261 |
| TrueNAS (SCALE/Apps) | Dedicated OS | No | Technical | Manual | Free; enterprise paid | Active (25.10 "Goldeye") | n/v | https://tech-insider.org/?p=22261 |
| Nextcloud AIO | Docker app | YES, officially via Docker Desktop, with caveats | Technical-ish | Requires domain + valid HTTPS (no IP-only / self-signed) | Free OSS | Very active; pushed 2026-10-02 | 10,528 | https://github.com/nextcloud/all-in-one ; raw readme https://raw.githubusercontent.com/nextcloud/all-in-one/main/readme.md |
| Portainer | Docker UI | Yes (Docker Desktop) | Technical | None built-in | Free CE / paid BE | Active; pushed 2026-10-01 | 38,614 | https://github.com/portainer/portainer |
| Dockge | Compose UI | Via Docker | Technical | None | Free | Slower; last push 2026-04-25 | 24,511 | https://github.com/louislam/dockge |
| Homarr | Dashboard | Via Docker | Hobbyist | None | Free | Active (homarr-labs/homarr; old ajnart/homarr is archived) | 4,941 | https://github.com/homarr-labs/homarr |
| Yacht | Docker UI | Via Docker | Hobbyist | None | Free | Active (Yacht-sh/Yacht) | 3,874 | https://github.com/Yacht-sh/Yacht |
| Coolify | PaaS for servers/VPS | No (Linux server) | Developers | Auto HTTPS (domain) | Free self-host; cloud $5/mo | Very active; pushed 2026-10-01 | 62,491 | https://github.com/coollabsio/coolify ; https://dev.to/vikasprogrammer/i-compared-6-platforms-for-deploying-self-hosted-apps-in-2026-3j8 |
| Dokploy | PaaS for servers | No | Developers | Auto HTTPS | Free OSS + paid cloud | Very active; pushed 2026-10-01 | 37,612 | https://github.com/Dokploy/dokploy |
| Easypanel | Server panel | No | Developers | Auto HTTPS | Freemium | n/v | n/v | n/v |
| Synology DSM Package Center | Vendor NAS OS | No (hardware) | Non-technical | QuickConnect / DDNS (vendor) | Hardware purchase | Active (not researched in detail) | closed | n/v |
| Pinokio | Desktop app (Electron-like "AI browser") | YES (Win/Mac/Linux) | Non-technical AI hobbyists | "LAN Wide Web": access apps from other LAN devices; assigns https://appname.localhost-style URLs (Pinokio 5.0, 2025-11-30) | Free OSS (MIT) | Very active; pushed 2026-09-02 | 8,191 (a Jul-2026 blog said 7,797) | https://github.com/pinokiocomputer/pinokio ; https://the-decoder.com/pinokio-5-0-turns-local-machines-into-personal-ai-clouds/ |
| StabilityMatrix | Windows desktop app, AI package manager | YES | Non-technical AI users | Local | Free OSS | Active; pushed 2026-10-01 | 8,861 | https://github.com/LykosAI/StabilityMatrix |
| Home Assistant on Windows | VM appliance (VirtualBox/Hyper-V) or Container | YES but via VM image/Docker | Mixed | Nabu Casa cloud ($) or own | Free OSS; paid cloud | Very active | n/v | https://www.home-assistant.io/installation/windows/ |
| Umbrel/CasaOS "on Windows" DIY | Community WSL2 guides | DIY | Technical | n/a | n/a | Reports of random disconnects, apps not coming back | n/a | https://community.umbrel.com/t/umbrel-on-wsl2-with-windows-11/11681 ; https://wiki.casaos.io/en/guides/running-casaos-on-windows-with-wsl2 |

Not found: any product named for Docker Desktop "extensions marketplaces" for self-hosted apps (search surfaced none); no LocalAI desktop detail verified; Synology/Easypanel/TrueNAS repo stats not verified.

## 2. Closest competitors: better / worse

- **Pinokio** (closest *pattern*). Better: proven Windows one-click installer UX; LAN sharing built in; MIT; huge momentum. Worse for this use case: AI-app catalogue only; no always-on server model; local-first https names (appname.localhost-style) rather than real public-cert HTTPS to phones (inference from the 5.0 article; verify before relying on it). Sources: https://the-decoder.com/pinokio-5-0-turns-local-machines-into-personal-ai-clouds/
- **Umbrel / CasaOS / Runtipi** (closest *catalogue + UX*). Better: mature app stores, polished UI, large communities (Umbrel 12.2k, CasaOS 37.3k, Runtipi 9.7k stars). Worse: all need a Linux box (wipe the laptop) or an unsupported WSL2 hack. They do not give phone HTTPS with zero setup. Sources in table.
- **Nextcloud AIO on Docker Desktop** (closest *Windows-supported* path). Better: official Windows route. Worse: single app, requires owning a domain/valid HTTPS, Docker Desktop, technical docs.
- **HexOS** (closest *non-technical positioning*). Better: funded team, explicit non-technical target. Worse: bare-metal OS install, paid ($199-$299), NAS-first. https://hexos.com/
- **Docker Desktop + Portainer/Dockge**. Better: existing Windows route and large audiences. Worse: no catalogue-with-plain-English, no HTTPS story, technical.

## 3. Is the gap real?

Narrowly yes, with caveats.
- I found no product that does all of: Windows host + hidden WSL2 Debian + app cards + real HTTPS for phones + zero terminal for self-host apps like Actual/Jellyfin/Vikunja. Search queries for "self-host on old Windows laptop one click" returned only generic Windows bulk installers (Ninite etc.), not self-host tools: https://alternativeto.net/software/ninite/?p=6 . Newer entrants found in 2026 listings were managed hosting (Elestio, PikaPods, InstaPods), i.e. paid cloud, not local: https://dev.to/vikasprogrammer/i-compared-6-platforms-for-deploying-self-hosted-apps-in-2026-3j8
- CasaOS wiki shows the exact concept exists as a DIY guide, so the idea is not novel technically; nobody has productised it: https://wiki.casaos.io/en/guides/running-casaos-on-windows-with-wsl2
- Pinokio shows the productised-installer model works on Windows, but for AI only. The adjacent slot is plausible; a Pinokio-style app could also add self-host catalogue entries, which is a competitive threat (low switching cost for them).
- Differentiator check: (1) "no OS wipe" is real vs Umbrel/CasaOS/HexOS/Unraid/Start9 but not vs Nextcloud AIO / Docker Desktop / Pinokio; (2) plain-English framing is cheap to copy; (3) zero-terminal is shared with Pinokio/Umbrel; (4) HTTPS on phones is the hardest and most defensible differentiator, but see risks.
- Absence of evidence caveat: search engines are weak at finding small GitHub projects; do a GitHub-topic search (wsl2, self-hosted, windows installer) before committing.

## 4. Demand evidence

- r/selfhosted: ~812K members (May 2026, via a tracker snippet; other trackers showed 639K and 789K). Low-medium confidence; reddit.com not fetchable. https://gummysearch.com/r/selfhosted
- awesome-selfhosted: 323.3k stars, 15.1k forks (fetched 2026-10-02). https://github.com/awesome-selfhosted/awesome-selfhosted
- selfh.st 2025 survey: 4,081 completed responses; Linux used by 81%; Proxmox 45%; Home Assistant OS 29%; Raspberry Pi OS 24%; Docker ~9 in 10. https://selfh.st/survey/2025-results/ and https://linuxiac.com/self-hosters-confirm-it-again-linux-dominates-the-homelab-os-space/ . Implication: the existing audience is Linux-heavy and technical; the survey shows no evidence of a large Windows-first self-hoster segment. I could not extract a Windows share from the page (chart data is in a JSON on GitHub); not verified.
- Pinokio adoption: 8,191 GitHub stars; a third-party blog claims 31,832 Windows installer downloads in the first week of v8.0.40 (22 July 2026), low confidence single source. https://pasqualepillitteri.it/en/news/9050/pinokio-ai-browser-install-guide . Evidence that non-technical Windows users install one-click local apps, but AI hype is the pull, not subscription replacement.
- Subscription fatigue: marketing-grade claims ("40% of consumers overwhelmed by subscriptions", "self-hosting surged 40% since 2023", "$85.2B market by 2034") from vendor blogs; treat as directional only. https://blog.elest.io/self-hosting-in-2026-why-the-85b-market-boom-changes-everything/ ; https://www.webpronews.com/self-hosting-surges-in-2026-market-to-reach-85-2b-by-2034/
- Non-technical demand is served by paid hosts and hardware: Umbrel Home $399, Start9 Server One $749+, HexOS $199 early access. People do pay for easy.
- Old Windows hardware: Windows 10 still 27.83% of Windows desktops (Sept 2026), Windows 11 71.44%. https://gs.statcounter.com/os-version-market-share/windows/desktop/worldwide

## 5. Technical risks (with sources)

### Public Suffix List (verified directly)
Downloaded https://publicsuffix.org/list/public_suffix_list.dat on 2026-10-02 (16,501 lines). Exact lines:
- Line 13022-13024: `// DuckDNS : http://www.duckdns.org/` / `// Submitted by Richard Harper <richard@duckdns.org>` / `duckdns.org` -> **DuckDNS IS on the PSL (private section).**
- Line 12908-12910: `// deSEC : https://desec.io/` / `// Submitted by Peter Thomassen <peter@desec.io>` / `dedyn.io` -> **deSEC dedyn.io IS on the PSL.**
Consequence: Let's Encrypt uses the PSL to define the "registered domain", so each `name.duckdns.org` is its own registered domain; the 50-certs-per-registered-domain-per-7-days limit applies per user subdomain, not across all DuckDNS users. https://letsencrypt.org/docs/rate-limits/ (50 certs/registered domain/7 days; 300 new orders/3h/account; 5 duplicate certs/7 days; 5 auth failures/identifier/hour). Residual risk: 5 duplicate certs/week per exact name set; a buggy reinstall loop can hit it. Per-account (300 orders/3h) and shared LE infrastructure limits still apply to a fleet.

### WSL2 always-on reliability
- `vmIdleTimeout` default 60000 ms; `instanceIdleTimeout` default 15000 ms (-1 disables); **`vmIdleTimeout`, `[boot] command`/`systemd` boot settings, `dnsTunneling`, `autoProxy` are marked Windows 11 only; mirrored networking needs Windows 11 22H2+.** https://learn.microsoft.com/en-us/windows/wsl/wsl-config (page updated 2026-09-16). Windows 10 is still ~28% of desktops, and old laptops skew Win10, so the target machine class is the worst-supported one. Win10 left mainstream support 2025-10-14; ESU extends security patches only to Oct 2027 per search snippets (low confidence on exact dates; see https://winaero.com/microsoft-has-ended-support-for-windows-10).
- Even with `vmIdleTimeout=-1`, users report services inside WSL being idled out (WSL 2.5.9.0, regression since 2.5.7, reported 2025-07-26). https://github.com/microsoft/WSL/issues/13291 . Workaround: keep a persistent process alive (e.g. `sleep infinity`). https://github.com/microsoft/WSL/issues/40363
- WSL/Docker start only after user login by default; needs a scheduled task "run whether user is logged on or not". After hibernate/resume, Docker networking sometimes needs restarts (`Restart-Service vmms`). https://forums.docker.com/t/how-to-automatically-start-docker-in-windows-10-without-user-login/140086?page=2 ; https://forums.docker.com/t/docker-2-0-5-0-not-responding-and-not-loading-containers-after-sleeping-windows-10/76773 ; https://github.com/microsoft/WSL/discussions/14261 (production use without login; no Microsoft staff reply found).
- Community reports that Umbrel on WSL2 "constantly" disconnects. https://community.umbrel.com/t/umbrel-on-wsl2-with-windows-11/11681
- Windows Update reboots, sleep/lid-close on laptops, battery/thermal behaviour are product-level risks you must handle (power plan, "do nothing on lid close", active hours); none solved by WSL.

### LAN access from WSL
- Default NAT mode: only localhost forwarding from Windows; other LAN devices cannot reach containers without port proxy/firewall rules. Mirrored mode enables LAN access but needs Hyper-V firewall inbound rule (`Set-NetFirewallHyperVVMSetting ... -DefaultInboundAction Allow`) and Win11 22H2+; users report LAN access failing even though ping works. https://learn.microsoft.com/windows/wsl/networking ; https://community.norton.com/t/trying-to-access-http-server-running-on-wsl-2-distro-configured-in-mirrored-networking-mode/361198
- Practical design implication: run a Windows-side `netsh interface portproxy`/service or a Windows-side reverse proxy for 443; handle IP changes at each WSL restart on NAT mode.

### Docker Engine in WSL
- Works (CasaOS wiki documents systemd + Docker Engine in WSL2) but needs systemd=true and `wsl --shutdown` cycles; Docker Desktop must not be co-installed or it conflicts. https://wiki.casaos.io/en/guides/running-casaos-on-windows-with-wsl2 ; https://learn.microsoft.com/en-us/windows/wsl/wsl-config
- WSL2 disk is a VHDX with no storage-management layer (CasaOS wiki: "Storage management is not possible"); media libraries on Windows drives via /mnt/c are slow (DrvFs). Not benchmarked here.

### DuckDNS reliability
- No official status page found; third-party monitors show it operational as of 2026-09-10 but with user reports. https://statusgator.com/services/duck-dns
- Repeated community reports of multi-hour outages (e.g. "out for 3-4h") and SERVFAIL; "downtime several times a year"; users move to deSEC, Cloudflare or a $ domain. https://lemmy.linuxuserspace.show/post/51501 ; https://community.home-assistant.io/t/duckdns-down/186385 ; https://forum.jellyfin.org/t-reliable-duckdns-alternative (thread has no dated incidents)
- Single-operator free service hosted on AWS (per StatusGator). Because your app has no servers, the DNS provider is your hidden single point of failure; a DuckDNS outage = every user's phone loses HTTPS access simultaneously. Offer deSEC (on PSL, non-profit) as a fallback.
- Design issue: HTTPS to a private LAN IP requires DNS-01 plus a public A record pointing at a private IP; many routers' DNS rebinding protection blocks that answer, and DuckDNS nameservers can be unreachable from some networks. https://gitea.kelinreij.duckdns.org/kelin/EZ-Homelab/src/tag/v0.1.2/docs/troubleshooting/SSL-CERTIFICATES-DUCKDNS.md ; https://community.homey.app/t/technical-the-pros-and-cons-of-dns-rebinding-protection/59889?page=2 . This directly threatens differentiator (4). Also: all your certificates (hostnames) are public in Certificate Transparency logs.

### Microsoft licensing
- WSL is MIT-licensed since 2025-05-19 (some filesystem pieces still proprietary as of Sept 2025). https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux ; https://en.linuxadictos.com/WSL-2-becomes-the-first-open-source-version-of-the-Windows-Subsystem-for-Linux..html
- Debian on WSL is distributed via the Store or sideload; redistributing a rootfs is a Debian/GPL matter, not Microsoft. Windows itself on an end-user's own laptop: no extra licence issue found. Risk is Windows Home vs Pro: WSL2 works on Home, but BIOS virtualization must be on. Searches returned no explicit Microsoft ban on server-style use of WSL on client Windows; the "WSL not for production" statement is historic and unconfirmed here. A lawyer check is cheap if you ever bundle Docker Desktop (not planned: you use Docker Engine, which is Apache-2.0).
- No Microsoft licensing blocker identified; confidence moderate (not primary-source verified).

### Support burden
- Failure modes are multi-layered (BIOS virtualization, WSL version, Win10/11 feature gaps, router DNS rebinding, DuckDNS, LE limits, antivirus, sleep/updates). A non-technical user cannot self-diagnose any of them, and a $0, no-servers model has no revenue to support them. Competitors that serve non-technical users monetise via hardware (Umbrel Home $399, Start9 $749+ incl. support, HexOS $199-$299). Evidence: pricing links above. This is the biggest non-technical risk.
- Every self-hosted app update, data-loss event (no backups by default, single laptop disk) and security issue (apps exposed on home network, token for DuckDNS stored on disk) lands on you reputationally.

## 6. Verdict: RISKY (viable as a narrow, honest MVP; not as the pitched "works for anyone's old laptop")

Reasons: gap exists but is narrow; Windows 11 + always-on WSL2 + DuckDNS HTTPS is feasible, but the target hardware (old laptops, often Win10) is the least supported by WSL2, and phone HTTPS depends on third-party DNS and router behaviour that you cannot control. Demand evidence shows the self-hoster community is large but Linux/technical; non-technical Windows demand is shown only for AI tools (Pinokio).

### Three changes that most raise the odds
1. **Require/detect Windows 11 22H2+ and gate on a preflight check** (virtualization on, WSL version, systemd, mirrored/portproxy works, power plan). Say plainly "old Win10 laptops are not supported" or provide a degraded mode; run a Windows-side watchdog/scheduled task (start at boot without login, keep-alive process, restart WSL on failure, disable sleep) instead of trusting `vmIdleTimeout`.
2. **De-risk HTTPS/DNS**: use deSEC (dedyn.io, on PSL, non-profit) as primary or at least as an automatic fallback to DuckDNS; implement DNS-01 with a Windows-side reverse proxy (Caddy) that terminates TLS; ship a built-in "can my phone reach it?" diagnostic including DNS-rebinding detection with a one-click fix guide; consider optional Tailscale for off-home access instead of exposing anything.
3. **Narrow and ship the Pinokio-style wedge first**: 3 to 5 apps (Actual, Jellyfin, Navidrome, Vikunja, Stirling-PDF) with built-in backup (to USB/second drive), a one-click "export my data" and a diagnostics bundle, then validate with 20 real non-technical users before building the catalogue. Consider partnering/contributing to Pinokio or Runtipi catalogues (reusing their app definitions) rather than building a new store; and keep the "replace X" copy as the only truly cheap-to-copy differentiator.

## Gaps / unverified
- Windows share and DuckDNS usage in the selfh.st 2025 survey (data in JSON, not read); exact r/selfhosted count (reddit blocked); Synology/Easypanel/TrueNAS app details; Start9 and FreedomBox star counts; Cosmos Cloud pricing (single mirror source); Unraid pricing (account.unraid.net not fetched); whether Pinokio's HTTPS domains work on phones; whether Windows 10 supports WSL idle settings beyond the Learn page's footnotes; Microsoft licensing from a primary legal source.
