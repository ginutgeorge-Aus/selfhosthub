# SelfHostHub

**Turn an old laptop into your own private cloud, with no terminal required.**

SelfHostHub is a free, open-source Windows app for people who aren't technical. You install it on an old laptop, click **Replace YNAB** or **Replace Plex**, and get a free, open-source alternative running at home, with a padlock (HTTPS) on every phone and PC in the house.

> **Status: design phase.** No release yet. See the [app catalog research](research/app-catalog.csv).

## How it works

- A Windows app (Tauri) runs a setup wizard that checks your laptop, switches on Windows' Linux support (WSL2) and guides you through turning on virtualization if it's off.
- A small Linux environment runs inside Windows, with Docker Engine and our `hub-agent`.
- Each app runs in its own container behind Caddy, with a free HTTPS certificate from Let's Encrypt.
- Free [DuckDNS](https://www.duckdns.org) names (with [deSEC](https://desec.io) as the backup) let every device on your home network find the laptop. **We run no servers and charge nothing.**

## First apps (MVP)

| Replaces | With |
|---|---|
| Adobe Acrobat | Stirling-PDF |
| YNAB | Actual Budget |
| Plex Pass | Jellyfin |
| Todoist | Vikunja |
| Spotify (your own music) | Navidrome |

All of them run on your home network and keep working without internet.

## Requirements

- Windows 10 22H2 or Windows 11
- 8 GB RAM recommended, 40 GB free disk space
- A processor with virtualization support (most laptops from 2012 onwards)

## License

[GPL-3.0](LICENSE). Each hosted app keeps its own license.
