# EZRecon

**Passive reconnaissance for the mainframe hunter.**

A ground-up rebuild of the EZRecon tool from *Hacking Mainframes* (Chapter 6):
a fast, asynchronous OSINT engine wrapped in a phosphor-green [Textual](https://textual.textualize.io)
TUI and a fully scriptable CLI. EZRecon gathers the passive half of a mainframe
engagement — Google dorks, WHOIS, DNS, subdomains, certificate transparency, an
email spider, Shodan banner sweeps, and an AI deep-research OSINT prompt — and
assembles it into a single dossier you can export as a report.

There is **no Nmap** here, on purpose. Active Nmap scanning is its own part of
the book; EZRecon is the passive workhorse that feeds it.

![Python](https://img.shields.io/badge/python-3.10%2B-33ff66?style=flat-square)
![Interface](https://img.shields.io/badge/interface-Textual%20TUI%20%2B%20CLI-ffb000?style=flat-square)
![Engine](https://img.shields.io/badge/engine-async-33ff66?style=flat-square)
![Focus](https://img.shields.io/badge/focus-mainframe%20%2F%20z%2FOS-1f8a3a?style=flat-square)
![Licence](https://img.shields.io/badge/licence-Community%20%26%20Educational%20v1.0-ffb000?style=flat-square)

```
 ______ ___________
 |  ___/___  /  __ \   EZRecon 2.0
 | |__    / /| /  \/   passive recon for the mainframe hunter
 |  __|  / / | |
 | |___./ /__| \__/\   v2.0.0
 \____/\_____/\____/
```

---

## Table of contents

- [Highlights](#highlights)
- [Install](#install)
- [Quick start](#quick-start)
- [The TUI](#the-tui)
- [The CLI](#the-cli)
- [API keys and configuration](#api-keys-and-configuration)
- [Modules](#modules)
- [Reports](#reports)
- [Mainframe fingerprinting](#mainframe-fingerprinting)
- [Architecture](#architecture)
- [Project layout](#project-layout)
- [Chapter 6 traceability](#chapter-6-traceability)
- [Development and testing](#development-and-testing)
- [Roadmap](#roadmap)
- [Legal and scope](#legal-and-scope)
- [Licence](#licence)

---

## Highlights

- **Async engine.** DNS, subdomain brute force, banner grabs and the email
  spider all run concurrently. A sweep that used to take minutes takes seconds.
- **Two front-ends, one engine.** A phosphor-themed Textual TUI for interactive
  work and a subcommand CLI for scripting and CI — both drive the exact same
  recon engine and the same `Session` model, so results are identical.
- **Mainframe-aware throughout.** A z/OS port profile, a Shodan banner-query
  library (TSO, IBM FTP, CICS, Db2/DRDA, NJE), a mainframe Google-dork
  catalogue, and a fingerprint scorer that flags *likely mainframe* and names
  the service.
- **Certificate-transparency enrichment.** Pulls subdomains straight out of
  crt.sh to catch the shadow endpoints a wordlist never will.
- **One-shot auto-recon.** Point it at a domain and it runs the whole passive
  chain into a single dossier.
- **AI-OSINT prompt generator.** Emits the deep-research prompt from Chapter 6,
  pre-filled with the target and everything EZRecon already found.
- **Report export.** Markdown, HTML (phosphor-themed), JSON, and a `queries.txt`
  for the lab deliverable.
- **Opt-in native banner grab.** A connect-and-read across the mainframe port
  profile — no Nmap, no NSE, just a fast asyncio connect sweep.

---

## Install

Requires **Python 3.10+**.

```bash
git clone https://github.com/kmilne40/EZRecon.git
cd EZRecon

# option A: just the dependencies, run in place
python3 -m pip install -r requirements.txt

# option B: install the `ezrecon` command onto your PATH
python3 -m pip install .
```

On Debian/Ubuntu you may need `--break-system-packages` (or use a virtualenv):

```bash
python3 -m venv .venv && source .venv/bin/activate
python3 -m pip install .
```

---

## Quick start

```bash
# launch the interactive TUI
ezrecon
# ...or the book-compatible launcher (identical behaviour)
python3 EZrecon.py

# one-shot passive sweep with the native banner grab, write a report
ezrecon auto example.com --ports --report ./out

# generate an AI deep-research OSINT prompt for a target
ezrecon prompt example.com
```

---

## The TUI

Run `ezrecon` with no arguments. Pick a module from the sidebar, type a target,
and press **Run**. Findings stream into the live table; the status pane shows
progress and timings.

```
┌ MODULES ─────────────┐  EZRecon 2.0   passive recon for the mainframe hunter
│  Auto-Recon          │ ┌────────────────────────────────────────┬──────────┐
│  DNS Records         │ │ target domain or URL                   │   Run    │
│  WHOIS               │ └────────────────────────────────────────┴──────────┘
│  Subdomains          │ ┌ status ───────────────────────────────────────────┐
│  crt.sh (CT)         │ │ ▶ auto on example.com …                            │
│  Email Spider        │ │   · dns   · crt.sh   · subdomain brute   · done    │
│  Banner Grab         │ └───────────────────────────────────────────────────┘
│  Shodan Sweep        │ ┌ results ──────────────────────────────────────────┐
│  Google Dork         │ │ Sev   Category   Key        Value                 │
│  AI Prompt           │ │ high  port       fingerprint LIKELY MAINFRAME 82% │
│                      │ │ med   dork       …          …                     │
│  API keys            │ │ low   subdomain  tso.example.com  10.0.0.4        │
└──────────────────────┘ └───────────────────────────────────────────────────┘
```

### Keybindings

| Key | Action |
|----|--------|
| `r` | Run the selected module against the target |
| `a` | Run Auto-Recon |
| `e` | Export a report (md / html / json / queries.txt) |
| `k` | Open the API-key vault |
| `c` | Clear the results table |
| `q` | Quit |

The mouse works too — click a module, click **Run**, click the checkboxes in
the Google Dork panel. It runs fine over SSH.

---

## The CLI

Every module is a subcommand. Running `ezrecon` with no subcommand opens the TUI.

```bash
ezrecon dns example.com                          # records + SPF/DMARC + AXFR
ezrecon whois example.com
ezrecon subdomains example.com --wordlist my.txt # concurrent brute force
ezrecon crtsh example.com                        # certificate transparency
ezrecon email https://example.com --depth 2      # email spider
ezrecon ports example.com                        # native, non-nmap banner grab
ezrecon ports example.com --ports 21,23,50000    # custom port set
ezrecon shodan "IKJ56700A port:23"               # raw Shodan query
ezrecon shodan --mainframe example.com           # z/OS query library
ezrecon dork example.com --all                   # Google dorks (needs a key)
ezrecon prompt example.com                        # AI deep-research OSINT prompt
ezrecon auto example.com --ports --shodan        # full one-shot sweep
ezrecon config set shodan_api_key YOUR_KEY
ezrecon config show
```

### Command reference

| Command | Purpose | Key options |
|--------|---------|-------------|
| `dns <domain>` | A/AAAA/MX/NS/SOA/TXT/CNAME + SPF/DMARC, reverse, zone transfer | `--report` |
| `whois <domain>` | Registrar, dates, name servers, contacts | `--report` |
| `subdomains <domain>` | Concurrent brute force with wildcard detection | `--wordlist`, `--report` |
| `crtsh <domain>` | Passive subdomains from certificate transparency | `--report` |
| `email <url>` | Bounded async email spider | `--depth`, `--report` |
| `ports <target>` | Native (non-nmap) connect + banner + fingerprint | `--ports`, `--report` |
| `shodan [query]` | Shodan search or the z/OS query library | `--mainframe`, `--report` |
| `dork <domain>` | Mainframe Google-dork catalogue | `--all`, `--report` |
| `prompt <domain>` | Print an AI deep-research OSINT prompt | — |
| `auto <domain>` | One-shot passive sweep into a single session | `--ports`, `--shodan`, `--report` |
| `config set <name> <value>` | Store an API key or setting | — |
| `config show` | Print current keys (masked) and settings | — |
| `tui` | Launch the Textual interface | — |

Add `--report DIR` to any recon command to write `md`, `html`, `json` and
`queries.txt` into `DIR`.

Sample output:

```
                                DNS — example.com
┏━━━━━━━━━┳━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Sev     ┃ Key   ┃ Value                                                      ┃
┡━━━━━━━━━╇━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┩
│ info    │ A     │ 140.82.113.3                                               │
│ info    │ MX    │ 0 example-com.mail.protection.outlook.com                  │
│ low     │ DMARC │ v=DMARC1; p=quarantine; sp=reject; pct=100; …              │
└─────────┴───────┴────────────────────────────────────────────────────────┘
```

---

## API keys and configuration

Google Custom Search (for dorks) and Shodan need API keys. Store them once:

```bash
ezrecon config set google_api_key  YOUR_KEY
ezrecon config set google_cse_id   YOUR_CX
ezrecon config set shodan_api_key  YOUR_KEY
```

Keys are read with this precedence (first hit wins):

1. **Environment variables** — `GOOGLE_API_KEY`, `GOOGLE_CSE_ID`, `SHODAN_API_KEY`
2. **XDG config** — `~/.config/ezrecon/config.json` (written `chmod 600`)
3. **Legacy** — an `api_key.json` in the working directory (book compatibility)

Writes always go to the XDG config file. Tunable settings live there too:

| Setting | Default | Meaning |
|--------|---------|---------|
| `dns_concurrency` | `100` | Concurrent resolvers for the subdomain brute force |
| `port_concurrency` | `200` | Concurrent TCP connects for the banner grab |
| `spider_concurrency` | `20` | Concurrent fetches for the email spider |
| `timeout` | `5.0` | Default network timeout (seconds) |
| `theme` | `phosphor` | TUI theme |
| `output_dir` | *(cwd)* | Where reports are written by default |

---

## Modules

| Module | What it does |
|--------|--------------|
| **Auto-Recon** | Runs the whole passive chain (whois → DNS → crt.sh → subdomains → email → optional ports → optional Shodan) into one session |
| **DNS** | All common record types, SPF and DMARC parsing, reverse lookups, and a zone-transfer (AXFR) attempt against every name server |
| **WHOIS** | Registrar, creation/expiry/updated dates, name servers, contacts (via `python-whois`, with a system-`whois` fallback) |
| **Subdomains** | Concurrent brute force with wildcard-DNS detection so a wildcard zone doesn't report the whole wordlist as live |
| **crt.sh** | Passive subdomain discovery from certificate-transparency logs |
| **Email Spider** | Bounded, depth-limited async crawl that harvests email addresses from `mailto:` links and page text |
| **Banner Grab** | Native, non-Nmap async connect + banner read across the mainframe port profile, with fingerprint scoring |
| **Shodan** | Ships a z/OS banner-query library (TSO `IKJ56700A`, IBM FTP, CICS, Db2/DRDA, NJE) so you can sweep without memorising syntax |
| **Google Dork** | The mainframe dork catalogue (Listing 6-1) as a multi-select: tick the queries you want and fire them at the target |
| **AI Prompt** | Generates the Chapter 6 deep-research OSINT prompt, pre-filled with the target and any findings gathered so far |

---

## Reports

Any recon command with `--report DIR`, the TUI's **export** action (`e`), or
`ezrecon auto ... --report` produces four artefacts:

| File | Contents |
|------|----------|
| `ezrecon_<target>.md` | Human-readable Markdown report with a key-findings section |
| `ezrecon_<target>.html` | The same report, phosphor-themed, ready to hand over |
| `ezrecon_<target>.json` | The full structured `Session` for tooling / re-import |
| `ezrecon_<target>_queries.txt` | Every query EZRecon ran — the lab deliverable |

Example `queries.txt`:

```
# EZRecon queries — example.com
# 2026-08-06T16:27:35Z

dns:example.com
axfr:example.com
crt.sh:%.example.com
subdomain-brute:example.com
portscan:example.com:21,23,992,175,443,448,2809,3270,10007,50000
```

---

## Mainframe fingerprinting

The banner grab hands open ports and banners to a small, explainable scorer. It
weights the classic z/OS signals — the TSO logon prompt (`IKJ56700A`), VTAM /
USS messages, IBM FTP banners, CICS, Db2/DRDA — plus the mainframe port profile,
and produces a 0–100 confidence and a verdict:

| Confidence | Verdict |
|-----------|---------|
| 70–100 | `LIKELY MAINFRAME` |
| 35–69 | `POSSIBLE MAINFRAME` |
| 1–34 | `MAINFRAME SIGNALS` |
| 0 | `NO MAINFRAME SIGNALS` |

The port profile scanned by default: `21, 23, 992, 175, 443, 448, 2809, 3270,
10007, 50000`. Every score comes with its reasons, so nothing is a black box.

---

## Architecture

```
                 ┌──────────────┐        ┌──────────────┐
   Textual TUI ──┤              │        │              │
                 │  ezrecon/    │  uses  │  ezrecon/    │
   CLI  ─────────┤   engine/    ├───────▶│   models.py  │
                 │  (async)     │ returns│  Finding /   │
   your script ──┤              │        │  Session     │
                 └──────┬───────┘        └──────────────┘
                        │ collects into
                        ▼
                 ┌──────────────┐
                 │  report/     │  →  md / html / json / queries.txt
                 └──────────────┘
```

- The **engine** is pure async Python. It never imports Textual or curses, so
  it's unit-testable and reusable from anything.
- Everything discovered becomes a typed `Finding` collected into a `Session` —
  a single source of truth shared by the TUI, the CLI and the reporter.
- Front-ends are thin: they drive the engine and render the same `Session`.

---

## Project layout

```
EZRecon/
├── EZrecon.py                 # book-compatible launcher (python3 EZrecon.py)
├── pyproject.toml             # packaging + `ezrecon` entry point
├── requirements.txt
├── README.md
├── CHANGELOG.md
├── LICENSE                    # Community and Educational Licence v1.0
└── ezrecon/
    ├── __init__.py
    ├── __main__.py            # python3 -m ezrecon
    ├── cli.py                 # argparse subcommands
    ├── config.py              # XDG config + API-key vault
    ├── models.py              # Finding, Session, Severity, PortResult
    ├── fingerprint.py         # mainframe scorer + port profile
    ├── data/
    │   ├── mainframe_dorks.json     # Listing 6-1 dork catalogue
    │   ├── shodan_queries.json      # z/OS banner queries
    │   ├── banner_signatures.json   # port profile + banner regexes
    │   └── subdomains.txt           # mainframe-relevant wordlist
    ├── engine/
    │   ├── base.py            # validators + network primitives
    │   ├── dns_recon.py
    │   ├── whois_recon.py
    │   ├── subdomains.py
    │   ├── crtsh.py
    │   ├── email_scraper.py
    │   ├── ports.py           # native (non-nmap) banner grab
    │   ├── shodan_recon.py
    │   ├── google_dork.py
    │   ├── ai_osint.py
    │   └── pipeline.py        # auto-recon orchestrator
    ├── report/
    │   └── reporter.py        # md / html / json / queries.txt
    └── tui/
        ├── app.py             # Textual application
        └── app.tcss           # phosphor CRT stylesheet
```

---

## Chapter 6 traceability

EZRecon maps directly onto the passive-recon workflow in *Hacking Mainframes*.

| Chapter 6 capability | EZRecon module |
|----------------------|----------------|
| Google dorks, multi-select (Listing 6-1) | Google Dork |
| WHOIS (Listing 6-3) | WHOIS |
| `dig any` / DNS (Listing 6-4) | DNS |
| Subdomain brute force (Listing 6-5) | Subdomains |
| Email spider (Listing 6-6) | Email Spider |
| Shodan mainframe banners (Listing 6-7) | Shodan |
| AI deep-research prompt (Listing 6-8) | AI Prompt |
| CT logs / passive DNS for shadow endpoints | crt.sh |
| Build a target dossier / OSINT report | Reports + `queries.txt` |

---

## Development and testing

The async engine is designed to be driven from tests without a terminal, and
the TUI is exercised headlessly with Textual's pilot harness.

```bash
python3 -m pip install ".[dev]"

# quick smoke: engine + report round-trip
python3 -c "import asyncio; from ezrecon.engine import dns_recon; \
print(asyncio.run(dns_recon.dns_lookup('example.com'))[:2])"
```

Because the engine returns plain `Finding` / `Session` objects, you can mock any
network call and assert on the results, and drive the TUI with
`EZReconApp().run_test()` for click-path and modal tests.

---

## Roadmap

- PHOSPHOR handoff — export discovered TN3270 endpoints in a form
  [PHOSPHOR](https://offensivesec.org) can import, so a found mainframe is one
  keystroke from a live session.
- Additional passive sources (passive DNS history, more CT providers).
- Pluggable output formats and a findings-diff between runs.

---

## Legal and scope

EZRecon connects to live infrastructure when you use the **banner grab**, the
**email spider**, or **auto-recon with `--ports`**. Only point it at assets you
own or are explicitly authorised to test. Unauthorised reconnaissance may be
unlawful in your jurisdiction. You are responsible for staying within your rules
of engagement.

---

## Licence

Released under the **EZRecon Community and Educational Licence v1.0**. Free to
use, modify and distribute for educational, research and authorised security
testing, with attribution and no warranty. See [LICENSE](LICENSE) for the full
text.

Built by [Kev Milne](https://offensivesec.org) — OffensiveSec.org — as the
companion passive-recon tool for *Hacking Mainframes*.
