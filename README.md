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
     ____                     eZrecon
 ___|_  /_ _ ___ __ ___ _ _   passive recon for the
/ -_)/ /| '_/ -_) _/ _ \ ' \  mainframe hunter
\___/___|_| \___\__\___/_||_| v2.16.0
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
- **Parallel auto-recon with a passive OSINT chain.** The one-shot sweep runs its
  independent stages at the same time and only serialises what genuinely depends
  on earlier results, so wall time is roughly halved. After the core map it chains,
  in data-flow order, a DNS-only takeover check, the Wayback harvest, document
  metadata over the archived documents, and the people harvest. It stays passive
  and keyless throughout; active and keyed features are never fired automatically.
  `--light` skips the chain for a quick core sweep.
- **Four colour themes.** Phosphor green (default), Amber, Light (black on white)
  and Mono (white on black), switchable live in Options and remembered between
  runs. High and critical findings are drawn as a full red row so they stand out.
- **Two front-ends, one engine.** A phosphor-themed Textual TUI for interactive
  work and a subcommand CLI for scripting and CI — both drive the exact same
  recon engine and the same `Session` model, so results are identical.
- **Mainframe-aware throughout.** A z/OS port profile, a Shodan banner-query
  library (TSO, IBM FTP, CICS, Db2/DRDA, NJE), a mainframe Google-dork
  catalogue, and a fingerprint scorer that flags *likely mainframe* and names
  the service.
- **Certificate-transparency enrichment.** Pulls subdomains straight out of
  crt.sh, resolves them, and merges them with the brute-force results so a
  passive hit becomes a confirmed live host.
- **ASN / netblock enrichment.** Groups every resolved IP by ASN, owner and BGP
  prefix via Team Cymru — pure DNS, no key, no external tool — which often
  exposes the organisation's own address space.
- **Google dorks, with or without a key.** Tick the mainframe dork catalogue
  (Listing 6-1) and either fetch results via the Custom Search API, or use the
  no-key **link mode** to get clickable browser URLs — no key, no quota, no ToS
  issue.
- **Editable Shodan query builder.** Tick the mainframe queries you want, edit
  their syntax, or type your own — all from the TUI.
- **Preview pane (no browser needed).** Press `p` on a dork result to resolve it
  to real links (DuckDuckGo, or the Google API when keyed), fetch a page, and see
  where each dork term appears in context — with an optional LLM summary and a
  one-key "open in browser". Clearly gated as active recon.
- **Entity graph with live pivots.** Press `g` for a Maltego-style graph of the
  session — domains, subdomains, IPs, ASNs, emails, ports, dorks and their
  relationships. Select a node and pivot (resolve, scan, spider, Shodan, dork)
  with results folding back into the graph. Exports to a self-contained
  interactive HTML file.
- **Options popup + discovery chaining.** An in-TUI Options screen (`o`) exposes
  the CLI settings — wordlist, crawl depth, ports, preview engine — and when new
  subdomains are found EZRecon offers to chain the top ones into `site:` dork
  links and email-spider seeds (confirmation first, capped, no API quota).
- **Editable dorks & Shodan queries.** Every query in both panels is a tick-box
  plus an editable field, so you can tweak the syntax or add your own.
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

If you grabbed the release archive instead of cloning, it extracts **flat** —
the package files land directly in the current directory, with no wrapper
folder — so extract it into a directory of your choosing and work from there:

```bash
mkdir ezrecon && cd ezrecon
unzip /path/to/EZRecon.zip
python3 -m pip install -r requirements.txt   # or: python3 -m pip install .
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
| `p` | Preview the selected dork result (fetch + analyse) |
| `g` | Open the entity graph (navigate + pivot) |
| `o` | Open the Options popup (wordlist, depth, ports, chaining) |
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
ezrecon asn example.com                          # ASN / netblock enrichment
ezrecon email https://example.com --depth 2      # email spider
ezrecon ports example.com                        # native, non-nmap banner grab
ezrecon ports example.com --ports 21,23,50000    # custom port set
ezrecon shodan "IKJ56700A port:23"               # raw Shodan query
ezrecon shodan --mainframe example.com           # z/OS query library
ezrecon dork example.com --all                   # Google dorks (needs a key)
ezrecon dork example.com --all --links           # no key: clickable search URLs
ezrecon dork example.com --all --fetch           # no key: real Title/Link/Snippet results
ezrecon dork example.com --links --engine duckduckgo
ezrecon prompt example.com                        # AI deep-research OSINT prompt
ezrecon preview "site:example.com filetype:JCL"   # resolve a dork to real results
ezrecon preview "site:example.com" --pages --summary   # fetch pages, terms in context (active)
ezrecon auto example.com --ports --shodan        # full one-shot sweep
ezrecon auto example.com --chain                 # sweep, then chain new subdomains
ezrecon config set shodan_api_key YOUR_KEY
ezrecon config show
```

### Command reference

| Command | Purpose | Key options |
|--------|---------|-------------|
| `dns <domain>` | A/AAAA/MX/NS/SOA/TXT/CNAME + SPF/DMARC, reverse, zone transfer | `--report` |
| `whois <domain>` | Registrar, dates, name servers, contacts | `--report` |
| `subdomains <domain>` | Concurrent brute force with wildcard detection | `--wordlist`, `--report` |
| `crtsh <domain>` | Passive subdomains from certificate transparency (resolved + merged) | `--report` |
| `asn <target>` | ASN / netblock enrichment for a domain or IP (Team Cymru, pure DNS) | `--report` |
| `email <url>` | Bounded async email spider | `--depth`, `--report` |
| `ports <target>` | Native (non-nmap) connect + banner + fingerprint | `--ports`, `--report` |
| `shodan [query]` | Shodan search or the z/OS query library | `--mainframe`, `--report` |
| `dork <domain>` | Mainframe Google-dork catalogue (API fetch or `--links`) | `--all`, `--links`, `--engine`, `--report` |
| `prompt <domain>` | Print an AI deep-research OSINT prompt | — |
| `preview <query>` | Resolve a dork to real results; `--pages` fetches + shows terms in context (active) | `--engine`, `--limit`, `--pages`, `--summary` |
| `graph <session.json>` | Build an interactive HTML entity graph from a saved session | `--out` |
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

Google Custom Search (for dorks) and Shodan need API keys. An Anthropic key is
optional and only used for the preview pane's LLM summaries.

**Setting up Google dorking (Via API) — the reliable, one-time setup**

Keyless dorking via DuckDuckGo works from some networks but DuckDuckGo often
blocks scraping (HTTP 202). For results that always work, use the Google
Custom Search JSON API (free tier: 100 queries/day):

1. Enable the Custom Search API:
   https://console.cloud.google.com/apis/library/customsearch.googleapis.com
2. Create an API key: APIs & Services > Credentials > Create credentials > API key.
3. Create a Programmable Search Engine set to search the **entire web**:
   https://programmablesearchengine.google.com/ — then copy its **Search engine
   ID (cx)**.
4. Save them and verify:

```
ezrecon config set google_api_key YOUR_KEY
ezrecon config set google_cse_id  YOUR_CX
ezrecon config test        # one test query — tells you exactly what's wrong, if anything
```

`ezrecon config test` (or the **Save & test Google** button in the TUI keys
screen) runs a single query and reports precisely: API not enabled, invalid key,
wrong cx, quota exceeded, or all good.

Store the keys once:

```bash
ezrecon config set google_api_key  YOUR_KEY
ezrecon config set google_cse_id   YOUR_CX
ezrecon config set shodan_api_key  YOUR_KEY
ezrecon config set anthropic_api_key YOUR_KEY   # optional — preview summaries
```

Keys are read with this precedence (first hit wins):

1. **Environment variables** — `GOOGLE_API_KEY`, `GOOGLE_CSE_ID`, `SHODAN_API_KEY`
2. **XDG config** — `~/.config/ezrecon/config.json` (written `chmod 600`)
3. **Legacy** — an `api_key.json` in the working directory (book compatibility)

Writes always go to the XDG config file. Tunable settings live there too:

| Setting | Default | Meaning |
|--------|---------|---------|
| `dns_concurrency` | `100` | Concurrent resolvers for the subdomain brute force |
| `brute_timeout` | `2.0` | Per-query DNS timeout for the subdomain brute force (seconds) |
| `dns_nameservers` | *(public)* | Resolvers to use; `null` = fast public resolvers, `[]` = the system resolver, or a custom list |
| `port_concurrency` | `200` | Concurrent TCP connects for the banner grab |
| `spider_concurrency` | `20` | Concurrent fetches for the email spider |
| `timeout` | `5.0` | Default network timeout (seconds) |
| `theme` | `phosphor` | TUI theme |
| `output_dir` | *(cwd)* | Where reports are written by default |
| `anthropic_model` | `claude-haiku-4-5-20251001` | Model used for preview LLM summaries |
| `allow_active_fetch` | `false` | Skip the active-recon confirmation for the preview pane |

---

## Modules

| Module | What it does |
|--------|--------------|
| **Auto-Recon** | Runs the whole passive chain (whois → DNS → crt.sh → subdomains → email → optional ports → optional Shodan) into one session |
| **DNS** | All common record types, SPF and DMARC parsing, reverse lookups, and a zone-transfer (AXFR) attempt against every name server |
| **WHOIS** | Registrar, creation/expiry/updated dates, name servers, contacts (via `python-whois`, with a system-`whois` fallback) |
| **Subdomains** | Concurrent brute force with wildcard-DNS detection so a wildcard zone doesn't report the whole wordlist as live |
| **crt.sh** | Passive subdomain discovery from certificate-transparency logs, resolved and merged with the brute-force results |
| **ASN** | Groups every resolved IP by ASN, owner and BGP prefix via Team Cymru (pure DNS, no key) |
| **Email Spider** | Bounded, depth-limited async crawl that harvests email addresses from `mailto:` links and page text |
| **Banner Grab** | Native, non-Nmap async connect + banner read across the mainframe port profile, with fingerprint scoring |
| **Shodan** | A z/OS banner-query library you can drive from the TUI as a tick-box panel — toggle queries, edit their syntax, or add your own |
| **Google Dork** | The mainframe dork catalogue (Listing 6-1), editable per row. **Fetch results (no key)** returns real Title/Link/Snippet hits via DuckDuckGo (DuckDuckGo can block scraping with an HTTP 202 anti-bot response from some networks — if so, use **Via API** with a Google key for reliable results); **link mode** gives clickable URLs; or fetch via the Custom Search API if you have a key |
| **AI Prompt** | Generates the Chapter 6 deep-research OSINT prompt, pre-filled with the target and any findings gathered so far |
| **Preview** | Resolves a dork to real results, fetches a page, and highlights where terms appear in context — optional LLM summary, open-in-browser. Active recon, gated behind a confirmation |

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
| `ezrecon_<target>_graph.html` | Interactive entity graph (open in a browser) |

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

The release archive extracts flat — these files sit at the root, with no
wrapper directory:

```
.
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
    │   ├── base.py            # validators + tuned resolver + pooled HTTP
    │   ├── dns_recon.py
    │   ├── whois_recon.py
    │   ├── subdomains.py
    │   ├── crtsh.py           # CT logs, resolved + merged
    │   ├── asn.py             # ASN / netblock enrichment (Team Cymru DNS)
    │   ├── email_scraper.py
    │   ├── ports.py           # native (non-nmap) banner grab
    │   ├── shodan_recon.py
    │   ├── google_dork.py
    │   ├── ai_osint.py
    │   ├── webintel.py        # preview: results, page fetch, context, LLM summary
    │   ├── graph.py           # entity-graph model (nodes + edges from a session)
    │   ├── pivots.py          # per-node recon pivots (the Maltego loop)
    │   └── pipeline.py        # parallel auto-recon orchestrator
    ├── report/
    │   ├── reporter.py        # md / html / json / queries.txt (+ graph html)
    │   └── graph_html.py      # interactive Cytoscape graph export
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


### Keyless dork results with SearXNG (recommended)

Google's Custom Search JSON API is now closed to new customers, and DuckDuckGo
blocks scraping from many networks. The reliable keyless option is your own
SearXNG metasearch instance — and EZRecon sets it up for you in one command:

```
ezrecon searx setup      # runs SearXNG (JSON API enabled) and points EZRecon at it
ezrecon searx status     # check it's up and the JSON API works
```

If Docker isn't installed, setup offers to install it (Y/N) — Linux/Kali/
Raspberry Pi via the official script, macOS (Intel & Apple Silicon) via Colima +
Homebrew. Add `--yes` for unattended setup. After setup, the keyless **Fetch results** button and `ezrecon dork
<domain> --fetch` use SearXNG automatically. Already have a SearXNG instance?
Point at it with `ezrecon config set-setting searx_url http://host:8080` (make
sure `json` is in its `search.formats`).


### Dangling DNS / subdomain takeover

For each discovered subdomain EZRecon follows the CNAME chain and flags takeover
conditions using the can-i-take-over-xyz fingerprint set (a CNAME to a known
service whose page returns the unclaimed fingerprint, a provider whose target is
NXDOMAIN, or a dangling CNAME to a dead target). A record that merely resolves is
not flagged.

```
ezrecon takeover <domain>            # passive (CNAME chain + NXDOMAIN)
ezrecon takeover <domain> --http     # active: HTTP-confirm candidates
ezrecon takeover --update            # refresh provider fingerprints
```

In the TUI use the "Dangling DNS" module, or the "Check takeover" pivot on a
subdomain node. HTTP confirmation runs only with `--http` (CLI) or active recon
enabled (TUI).

### nmap / NSE builder

Press `n` in the TUI for a builder that reads your installed nmap: tick scripts
and only that script's `--script-args` appear, with a mainframe filter, book
presets, search, live command preview, and a Refresh button
(`--script-updatedb`). It composes and copies the command; run privileged scans
in your own shell.

```
ezrecon nse list --mainframe                 # book scripts present on this box
ezrecon nse args drda-brute                  # that script's documented args
ezrecon nse build 10.0.0.1 --preset db2-drda # build from a book preset
```


### Wayback historical URLs & GitHub dorking

```
ezrecon wayback <domain>              # archived URLs (flags JCL/COBOL/.env/.git/backups)
ezrecon github <domain> --org <org>   # GitHub code-search dork links (keyless)
ezrecon github <domain> --org <org> --fetch   # run via API (needs github_token)
```

Both are keyless in link mode. Wayback pulls from the Internet Archive CDX API
and flags interesting paths (with extra weight on mainframe artefacts). GitHub
dorking scopes code-search to the target; add a token for direct API results.
Both appear as TUI modules ("Wayback URLs", "GitHub Dorks") too.


### Favicon-hash pivot

```
ezrecon favicon <host>            # hash the favicon, print the Shodan query
ezrecon favicon <host> --shodan   # pivot to co-located hosts (needs Shodan key)
```

Also a "Favicon hash pivot" action on domain/subdomain/IP nodes in the graph.
Finds origin servers behind a CDN, dev/staging clones, and phishing copies that
reuse the same icon. No mmh3 dependency (vendored pure-Python MurmurHash3).


### CTI RSS reader

Press `s` in the TUI for a security-news reader (Krebs, BleepingComputer, Planet
Mainframe, and 10 more). Items mentioning your current target or mainframe
keywords are starred. Open items in the browser or email a selection.

```
ezrecon rss                         # list latest items per feed
ezrecon rss --highlight mainframe,RACF
ezrecon rss --email --to you@example.com   # needs SMTP config
```

SMTP is read from the config vault (no credentials in files):
```
ezrecon config set-setting smtp_host mail.example.com
ezrecon config set-setting smtp_from you@example.com
ezrecon config set-setting smtp_to dest@example.com
ezrecon config set smtp_username you@example.com
ezrecon config set smtp_password <password>
```


### Document metadata harvesting

```
ezrecon docmeta <domain>          # find docs via dorks+wayback, extract metadata
ezrecon docmeta --url https://x/report.pdf   # a specific document
```

Extracts author/last-modified-by usernames, authoring software + version, and
company fields from public PDFs and Office docs. Authors become people nodes in
the graph. Also a "Doc Metadata" TUI module and an "Extract doc metadata" pivot
on document nodes. No new dependencies (stdlib zip/xml; pypdf optional).


### People / email harvesting

```
ezrecon harvest <domain>                 # consolidate + PGP + infer naming
ezrecon harvest <domain> --deep          # run subdomains+crt.sh+spider first
ezrecon harvest <domain> --names "John Smith, Jane Doe"
```

Aggregates every email/name EZRecon has found (spider, docmeta, crt.sh, dorks,
Wayback), deduplicates with per-source tags, adds a keyless PGP keyserver lookup,
infers the org's email format, and generates likely addresses for known names.
Also a "People Harvest" TUI module.


### HIBP breach checking

```
ezrecon breach <domain>              # check emails already found
ezrecon breach --email john@x.com    # a specific address
ezrecon breach <domain> --harvest    # gather emails first, then check
```

Checks discovered emails against Have I Been Pwned (needs `hibp_api_key`).
Password-exposing breaches are flagged HIGH. Also a "Breach Check" TUI module;
breached accounts show as red nodes in the graph.


### Cloud bucket discovery

```
ezrecon buckets <domain>              # probe S3 + GCS
ezrecon buckets <domain> --org "SighberBank" --azure
```

Generates bucket names from the target + environment suffixes and checks S3/GCS/
Azure. Public (listable) buckets are flagged HIGH, private-but-existing MEDIUM.
Also a "Cloud Buckets" TUI module (active recon only, since it probes providers).
