# Changelog

All notable changes to EZRecon are recorded here, newest first.

## [2.21.0] — 2026-08-12

Changed
- MyMap's command builder is now universal and line-authoritative. The editable
  `$ nmap` line is the single source of truth: picking a category, script,
  arg, wordlist file, port, target or a scan-option toggle performs a surgical
  edit that touches only its own token, so anything you've typed that the
  builder doesn't model — evasion flags, decoys, output options, exotic targets
  — is preserved. Any valid nmap command runs exactly as written.
- The builder now forms NSE commands correctly, matching the nmap manual and
  the book: `--script-args` always follows `--script`, script names are used in
  their short form, argument values containing spaces or special characters are
  quoted (e.g. `tso-enum.commands="logon applid(TSO)"`), and empty argument
  placeholders are stripped before the scan runs.

Added
- A scan-options row in MyMap with one-tap toggles for `-sV`, `-Pn`, `-n`,
  `--open` and `+force` (force a script to run even when the service isn't
  detected) — the extras a scan, especially a mainframe scan, often needs.
- An Output pop-out (the "Output ⤢" button): a large, scrollable window showing
  the full scan output, with select-and-copy, a Copy all button, and Save to
  file. The inline output pane stays in place; the pop-out gives the big,
  copyable view on demand.

## [2.20.0] — 2026-08-11

Added
- A wordlist directory for MyMap. Under the command line there's now a
  "wordlists:" bar and a Files button that browses a directory of your lists
  (users.txt, passwords.txt, subdomains.txt and the rest) and drops a chosen
  file's full path onto the command. If you've picked a file-type arg such as
  `userdb`, selecting a file completes it to `userdb=/…/users.txt`, and the
  value survives later target and port edits. The directory is configurable in
  Options and remembered.
- A Notepad (key `t`, or the sidebar). A pop-up text editor with a directory
  bar, a file list, New / Open / Save / Copy all, and a filename box. It
  defaults to the same wordlist directory, so a list you create here is exactly
  what MyMap's Files button then offers.
- A structured report you can save as TXT, DOCX or PDF. The `e` export now
  opens a report screen with a live preview — target, a severity summary, key
  findings, findings by category, and the queries used — and buttons to save
  each format, plus the previous md/html/json export. The DOCX and PDF writers
  are pure Python, so no new dependencies were added.
- `--report-format {txt,docx,pdf,all}` on the CLI, to write the same structured
  report from the shell alongside `--report`.

## [2.19.0] — 2026-08-11

Changed
- MyMap is now three picker columns and a description: Category, Script, and a
  new Script-args column, with the script's description from the NSE database on
  the right. The `$ nmap` line is the single editable source of truth — the
  columns just fill it in. Selecting a script-arg inserts it into the line as
  `--script-args name=` ready for a value; picking several merges them, and
  changing the target or ports preserves them. Anything you can pass to nmap you
  can type straight onto the line.

Added
- A proper `--script-args` extractor in the MyMap engine that reads real
  argument names from the NSE source (the `@args` names, the names referenced
  through `get_script_args`, and the common library args — userdb, passdb, the
  brute settings — where they apply), instead of only descriptions.

## [2.18.0] — 2026-08-11

Added
- In-app help and labs. A Help / Labs item on the sidebar, and the F1 key
  anywhere, open a help screen with a topic for every feature: what it is, how
  to use it, and a short hands-on lab against a domain you own. F1 is
  contextual — on the main screen it opens the selected module's topic, and
  inside MyMap it reads the focused widget (the editable command line, the
  script list, the category list) and points you at the right place.
- MyMap now shows each script's documented --script-args on the right, and lets
  you fill them in; they fold into the built command as --script-args.
- MyMap's `$ nmap …` line is now editable. The fields still build a suggested
  command, but you can type into the line directly to add ports, switches or
  anything else before running. Run and Copy use exactly what's on that line,
  which also fixes switches ending up quoted into the hostname.

Changed
- Title is now "eZrecon: Passive and Active Recon"; the sidebar carries a
  "code by Kev Milne · https://offensivesec.org" credit pinned bottom-left.
- MyMap's description pane is narrower, giving the script list more room.

## [2.17.0] — 2026-08-11

Added
- MyMap, an nmap script runner, is now a screen of its own (press `n`), replacing
  the old NSE builder. It is a port of the standalone MyMap (Kev Milne, with
  Hubert Januszewski and Sophie Hall): browse the installed NSE scripts by
  category, including the mainframe and service sub-menus (MAINFRAME, SSL, SMB,
  SSH, RDP, DATABASE, VULN, BRUTE, FTP, RPC), or search them by keyword. Selecting
  a script shows its description and builds the nmap command. A large output pane
  runs along the bottom with the interesting lines highlighted (vulnerable,
  out-of-support, weak, deprecated), and a Report button writes a short findings
  paragraph. Speed dials save a command's flags for reuse and are seeded with a
  few mainframe-flavoured defaults; they live in `~/.config/ezrecon/mymap_speed_dial.json`.
  Running a scan is active recon, so it is gated behind an explicit confirm and
  built as an argument list rather than handed to a shell.

Changed
- The CTI RSS reader (press `s`) now uses the full-width layout with a fixed feed
  column, so item titles are readable instead of being squeezed into a narrow box.

## [2.16.2] — 2026-08-11

Added
- You can now add and remove CTI RSS feeds without editing packaged files. Feeds
  live in a user file at `~/.config/ezrecon/rss_feeds.json`, seeded from the
  built-in list the first time you add one, so your feeds survive upgrades. From
  the CLI: `ezrecon rss --add-feed "Name" "https://site/feed/"`, `--remove-feed
  "Name"`, and `--list-feeds`. In the TUI, the RSS reader (press `s`) has a name
  and URL box with an Add feed button. Duplicate names or URLs and non-http URLs
  are rejected with a clear message.

Changed
- `load_feeds` now prefers your user feeds file when it exists and falls back to
  the packaged defaults otherwise, so nothing changes until you add a feed.

## [2.16.1] — 2026-08-11

Fixed
- Keyless dork results through SearXNG came back empty when several dorks were
  run at once. The queries were fired in a burst, which rate-limited SearXNG's
  own upstream engines (Google, DuckDuckGo, Brave, Startpage all report back as
  suspended), so every query after the first few returned nothing even though the
  instance was healthy. The dork runner now paces the SearXNG path the way it
  already paced DuckDuckGo (default two seconds between queries, tunable with
  `config set-setting searx_pace <seconds>`), and a query that comes back empty
  because engines were suspended is retried once after a short pause. When engines
  are genuinely rate-limited the result now names them instead of a bare "no
  results", and the dork screen suggests slowing the run or pinning burst-tolerant
  engines. A genuinely empty query (nothing indexed) is reported as such and is
  not retried.

## [2.16.0] — 2026-08-10

Added
- Colour themes. The TUI now ships four: Phosphor (the default green CRT), Amber,
  Light (black text on white, for bright rooms and projectors) and Mono (white on
  black). Pick one in Options under a new Appearance section, at the top; it
  applies immediately and is remembered between runs. The theme drives every
  surface and the finding colours, so severities stay readable whichever you pick.
- High and critical findings are now drawn as a full red row, not just a red
  severity word, so the things that matter are impossible to miss in a busy table.
- Auto-recon now runs the passive OSINT chain. After the core sweep (whois, DNS,
  crt.sh, subdomains, email, ASN) it chains, in data-flow order, a DNS-only
  takeover check, the Wayback harvest, document-metadata extraction over the
  documents Wayback surfaced, and the people harvest to consolidate everyone. It
  stays passive and keyless throughout: active probes (bucket discovery, takeover
  HTTP confirmation) and keyed lookups (HIBP, the GitHub API) are never fired
  automatically. `ezrecon auto <domain> --light` skips the OSINT chain for a quick
  core map.

Changed
- The banner reads `eZrecon` now, rendered in a cleaner figlet, in the TUI and the
  README.

Docs
- New field and training manual (`EZRecon-Manual.docx`): a field guide, a training
  course and an introduction to OSINT in one, covering every module (what it does,
  how to run it, why you would), a recon methodology and mainframe playbook, and a
  full command, severity, keys and key-binding reference.

## [2.15.0] — 2026-08-06

Added
- Cloud bucket discovery. `ezrecon buckets <domain> [--org …] [--azure]` (and a
  "Cloud Buckets" TUI module) generates candidate bucket names from the target
  name plus environment/content suffixes (backups, db-dumps, terraform-state,
  prod, staging, …) and checks them against AWS S3, Google Cloud Storage and
  (optionally) Azure Blob. A publicly listable bucket is flagged HIGH (data
  exposed); one that exists but is private is MEDIUM (attack surface). Buckets
  become orange nodes in the graph. Probing the providers is active recon, so the
  TUI module runs only with active recon enabled.

This completes the OSINT expansion: dangling-DNS, NSE builder, Wayback, GitHub
dorking, favicon pivot, doc-metadata, people harvest, HIBP, and cloud buckets.

## [2.14.0] — 2026-08-06

Added
- HIBP breach checking. `ezrecon breach <domain>` (and a "Breach Check" TUI
  module) checks the emails EZRecon has discovered against Have I Been Pwned and
  flags which appear in known breaches and what was exposed. A breach that leaked
  passwords is marked HIGH (a direct credential-stuffing lead); PII-only breaches
  are MEDIUM. `--email` checks specific addresses; `--harvest` gathers emails
  first. Requests are paced and Retry-After is honoured to respect rate limits.
- HIBP and GitHub token fields added to the TUI API-keys screen. Needs
  `hibp_api_key` (config or the keys screen).
- Breached accounts become red breach nodes in the graph, edged to the email
  they compromise.

## [2.13.0] — 2026-08-06

Added
- People / email harvesting (theHarvester-style). `ezrecon harvest <domain>` (and
  a "People Harvest" TUI module) consolidates every email and name EZRecon has
  found - the email spider, document metadata, crt.sh, dork and Wayback snippets -
  deduplicates them with the sources that found each, adds a keyless PGP keyserver
  lookup, infers the organisation's email-naming convention (e.g. {first}.{last}),
  and generates the likely addresses for known names that weren't found directly.
  `--deep` runs subdomains + crt.sh + the email spider first to feed it; `--names`
  expands specific "First Last" names; `--no-pgp` skips the keyserver.
- Emails and inferred people flow into the existing email graph nodes and report.

Changed
- Keyless: the PGP source uses the HKP index on keyserver.ubuntu.com; no new
  dependencies.

## [2.12.0] — 2026-08-06

Added
- Document metadata harvesting (FOCA/metagoofil-style). Fetches the PDFs and
  Office documents EZRecon already finds via Google/GitHub dorks and the Wayback
  harvester, and extracts author and last-modified-by usernames, the authoring
  software and version, and company fields - the internal intel that leaks in
  document metadata. Authors become people nodes in the graph (naming
  conventions, real names); software/versions hint at patch level. `ezrecon
  docmeta <domain>` (auto-discovers docs) or `--url <doc>`; a "Doc Metadata" TUI
  module; and an "Extract doc metadata" pivot on dork/Wayback document nodes.
- Format-aware extraction resolves the creator ambiguity correctly (in Office
  `creator` is a person; in PDF `/Creator` is the authoring app, not a person).

Changed
- No new dependencies: Office files are read with stdlib zipfile+xml, PDFs are
  parsed from the Info dictionary/XMP on the raw bytes (pypdf used only if
  already installed).

## [2.11.0] — 2026-08-06

Added
- CTI RSS reader (integrated from Kev's cti-rss.py, reworked). Press `s` in the
  TUI for a phosphor-styled reader over 13 security feeds (Krebs, BleepingComputer,
  The Hacker News, Dark Reading, Planet Mainframe, and more): switch feeds, open
  items in the browser, and email a selection. Items mentioning the current target
  or mainframe keywords (z/OS, RACF, COBOL, CICS, IMS, LPAR) are starred and
  counted, so the news that matters to what you're recon-ing floats to the top.
  CLI: `ezrecon rss [--feed …] [--highlight …] [--email --to … --subject …]`.
- Email uses stdlib smtplib with SMTP settings from EZRecon's config vault
  (`config set-setting smtp_host/smtp_port/smtp_from/smtp_to`, `config set
  smtp_username/smtp_password`) — nothing hard-coded.
- New `config set-setting` command for non-secret settings.

Changed
- The RSS reader has no new dependencies: it uses the existing aiohttp (not httpx)
  and a stdlib RSS 2.0 / Atom parser (feedparser is used only if already
  installed), and fetches all feeds concurrently.

## [2.10.0] — 2026-08-06

Added
- Favicon-hash pivot. Hashes a target's favicon the way Shodan does
  (MurmurHash3 of the base64-encoded icon) and builds the
  `http.favicon.hash:<hash>` query to find every other host serving the same
  icon - origin servers behind a CDN, staging/dev clones, and phishing copies.
  Available as a "Favicon hash pivot" on domain, subdomain and IP nodes in the
  graph, and as `ezrecon favicon <host> [--shodan]`. With a Shodan key it pivots
  straight to co-located hosts; without one it still gives you the hash and the
  ready-to-paste query. Yellow `favicon` nodes in the graph.
- MurmurHash3 is vendored in pure Python (validated byte-for-byte against the
  mmh3 package), so there's no C-extension dependency; if mmh3 is installed it's
  used as a faster path.

## [2.9.0] — 2026-08-06

Added
- Wayback Machine / CDX historical URL harvesting. `ezrecon wayback <domain>`
  (and a "Wayback URLs" TUI module) pulls every archived URL under a domain from
  the Internet Archive CDX API and flags the interesting ones - config/secret
  files, backups, exposed .git, admin paths, data endpoints, and, weighted for
  this tool's audience, mainframe artefacts (JCL/COBOL/copybooks/REXX/HLASM).
  Keyless. The archive reveals deleted or unlinked files that live scanning
  misses.
- GitHub code-search dorking. `ezrecon github <domain> [--org <org>]` builds
  code-search queries scoped to the target - JCL/COBOL, connection strings,
  .env/config/keys, hardcoded passwords - and returns them as clickable links
  (keyless, like the Google dork link mode). With a token
  (`ezrecon config set github_token <PAT>`), `--fetch` runs them through the
  code-search API and records concrete repo/path hits. A "GitHub Dorks" TUI
  module emits the links.
- Both feed the findings table, the entity graph (new node kinds: purple
  `wayback`, grey `github`), and the report.

## [2.8.0] — 2026-08-06

Added
- Dangling DNS / subdomain-takeover detection. For each discovered subdomain
  EZRecon follows the CNAME chain and flags takeover conditions using provider
  fingerprints from the can-i-take-over-xyz project: a CNAME to a known service
  whose page returns the unclaimed fingerprint, a provider whose target is
  NXDOMAIN (e.g. Azure), or a dangling CNAME to a dead target. A record that
  merely resolves is not flagged - if an attacker already claimed it, it resolves
  fine - so detection relies on fingerprints and NXDOMAIN, not resolution alone.
- New "Dangling DNS" module in the TUI, a "Check takeover" pivot on subdomain
  nodes, red takeover nodes in the entity graph, and takeover findings in the
  report. CLI: `ezrecon takeover <domain> [--http] [--update]`.
- `ezrecon takeover --update` refreshes the provider fingerprints from the
  upstream can-i-take-over-xyz project (vendor status changes often).

Notes
- The passive checks (CNAME chain, NXDOMAIN) need no confirmation; the HTTP
  fingerprint confirmation is active recon, so it runs only with `--http` (CLI)
  or when active recon is enabled (TUI). HIGH severity means a fingerprint or
  NXDOMAIN-confirmed candidate; MEDIUM means an unconfirmed candidate to verify.

Added
- nmap / NSE command builder. A pop-up window (press `n`) and CLI (`ezrecon nse
  …`) that reads your installed nmap: the script list comes from script.db and is
  always current, and each script's `--script-args` are parsed from its .nse
  source, so once you tick a script only that script's args are offered. Includes
  a mainframe filter (auto-finds the book's TN3270/CICS/DB2/DRDA/TSO/VTAM
  scripts), a search box, book presets (Mainframe sweep, DB2/DRDA, TN3270), a
  live command preview, and a Refresh button that runs `nmap --script-updatedb`
  so newly-added scripts appear. The builder composes and copies the command;
  running privileged scans stays in your own shell.
- CLI: `ezrecon nse list [--mainframe] [--category]`, `nse args <script>`,
  `nse presets`, `nse build <target> [--preset …|--script …]`, `nse updatedb`.

## [2.7.2] — 2026-08-06

Fixed
- `ezrecon searx setup` failing with "System has not been booted with systemd as
  init system" on environments without systemd (WSL, containers, chroots). It no
  longer calls `systemctl` blindly: it detects whether systemd is actually PID 1
  and otherwise starts the Docker daemon via `service docker start`, then
  `/etc/init.d/docker`, then `dockerd` directly. On WSL it explains how to enable
  systemd (or use `service`/Docker Desktop integration), and if Docker genuinely
  can't run in the environment it points you at using an external SearXNG
  (`ezrecon config set-setting searx_url http://host:8080`).

## [2.7.1] — 2026-08-06

Changed
- `ezrecon searx setup` now offers to install Docker for you (Y/N) instead of
  just printing commands, and works across platforms:
  - Linux / Raspberry Pi OS / Kali — the official get.docker.com script (sudo).
  - macOS, Intel and Apple Silicon — Colima + the Docker CLI via Homebrew
    (offering to install Homebrew first if needed). The SearXNG image is
    multi-arch, so the container runs natively on both.
  It also starts the daemon (Colima VM / docker service) if it isn't running,
  and transparently uses sudo on Linux before a re-login takes effect. Add
  `--yes` to skip all prompts for unattended setup.

## [2.7.0] — 2026-08-06

Added
- SearXNG support — reliable, keyless, self-hosted dork results. Google closed
  its Custom Search JSON API to new customers and DuckDuckGo blocks scraping, so
  the durable keyless path is your own metasearch instance. `ezrecon searx setup`
  does it in one command: it generates a settings.yml with the JSON API enabled
  (SearXNG ships with it OFF, which otherwise 403s every API call), runs the
  official SearXNG container, points EZRecon at it, and verifies with a test
  query. `ezrecon searx status` / `ezrecon searx stop` manage it.
- When a SearXNG instance is configured, the keyless "Fetch results" button and
  `ezrecon dork --fetch` use it automatically (falling back to DuckDuckGo if it
  isn't set up). Same Title/Link/Snippet view; the header shows the source.

Notes
- Google's Custom Search JSON API is closed to new customers and retires on
  1 Jan 2027; the "Via API" path only works for accounts with prior access.
  EZRecon now says so plainly instead of sending you round the setup loop.
- `ezrecon searx setup` needs Docker; if it's missing, EZRecon prints the exact
  install commands. You can also point at any SearXNG (JSON enabled) with
  `ezrecon config set-setting searx_url http://host:8080`.

## [2.6.0] — 2026-08-06

Added
- Proper Google Custom Search setup path — the reliable way to fetch dork
  results (DuckDuckGo blocks keyless scraping on many networks). `ezrecon config
  test` (and a **Save & test Google** button in the TUI keys screen) runs one
  test query and tells you exactly what's wrong: API not enabled (with the
  console link to enable it), invalid key, wrong Search engine ID (cx), quota
  exceeded, or all good.
- Actionable error messages everywhere the API is used. Google's cryptic 400/403
  responses are translated into the specific fix, in both the CLI and the "Via
  API" results view, instead of a raw "HTTP 403".

Changed
- API keys and the cx are sanitised on entry (whitespace, accidental quotes and
  embedded newlines stripped) so a stray pasted character no longer causes a
  silent 400.
- README now has step-by-step Google API setup.

## [2.5.2] — 2026-08-06

Fixed
- Keyless dork fetch getting HTTP 202 from DuckDuckGo on every query. That is
  DuckDuckGo's anti-bot response to a request it distrusts, not a rate-limit.
  EZRecon now presents as a normal browser (realistic User-Agent, POST with the
  right headers), retries with backoff, paces requests so a burst of dorks
  doesn't trip the block, and falls back to DuckDuckGo's lite endpoint. If every
  query is still blocked, it says so plainly and points you at the API path,
  rather than mislabelling it a rate-limit.

Note: keyless scraping is inherently fragile — DuckDuckGo actively blocks it and
some networks will stay blocked. For guaranteed results, add a Google Custom
Search key and use "Via API" (same Title/Link/Snippet view).

## [2.5.1] — 2026-08-06

Changed
- The Google Dork panel's "Via API" button now renders results in the same
  readable Title / Link / Snippet list as the keyless "Fetch results" path,
  instead of dumping rows into the findings table. Both fetch paths share one
  screen; the only difference is the source (DuckDuckGo vs the Custom Search API).

## [2.5.0] — 2026-08-06

Added
- Keyless dork results. The Google Dork panel has a new **Fetch results (no key)**
  button that pulls real Title / Link / Snippet results for each selected dork
  via DuckDuckGo — no API key, no quota, no 403s — and shows them as a readable
  numbered list ("Searching for: <dork>" then Title/Link/Snippet per hit). The
  hits are also recorded as findings, so they flow into the table, graph and
  report. On the CLI: `ezrecon dork <domain> --fetch`. Fetching is active recon,
  so it goes through the usual one-time confirmation.

## [2.4.1] — 2026-08-06

Fixed
- Tick boxes now render a clear tick. The checkbox glyph was being clipped to an
  ellipsis by a too-narrow column and drawn in a near-invisible colour; ticked
  boxes now show a dark tick on a bright-green box and unticked boxes read as an
  empty box, in the dork, Shodan and options panels.

## [2.4.0] — 2026-08-06

Added
- Entity graph with live pivots (a Maltego-style view). Every finding becomes a
  node — domain, subdomain, IP, ASN/netblock, email, port/service, dork — with
  typed edges (resolves-to, announced-by, has-email, runs, found-via), and it
  deduplicates naturally so ten subdomains sharing one netblock show as one ASN
  node. In the TUI, press `g`: navigate nodes, see each node's connections, and
  press Enter to pivot — run a recon action scoped to that node (resolve, port
  scan, spider, reverse DNS, ASN, Shodan host/net, dork links) whose results
  fold back into the graph, which redraws. Active pivots go through the same
  active-recon confirmation as the preview pane.
- Interactive HTML graph export. A self-contained, force-directed, draggable/
  zoomable graph (Cytoscape, phosphor-themed, with an offline fallback) — from
  the graph screen's `x` key, the CLI `ezrecon graph <session.json>`, or as a
  `*_graph.html` file in every report bundle. Clicking a node shows its details
  and the pivots available for it.

The graph and pivots are pure re-renderings of the session you already have, so
they slot in next to chaining without collecting anything new on their own.

## [2.3.0] — 2026-08-06

Added
- Options screen. Press `o` (or the sidebar button) for a popup that exposes the
  search settings that were previously CLI-only: subdomain wordlist path, DNS
  concurrency and brute timeout; email-spider seed URL, crawl depth, max pages
  and concurrency, plus a field to record known email addresses in the dossier;
  the banner-grab port set and concurrency; the preview engine and result limit;
  and the chaining toggle and cap. The TUI now drives the engine with these
  values instead of hardcoded defaults.
- Discovery chaining. When a run turns up new subdomains, EZRecon can feed the
  top ones straight back into the workflow: each becomes a `site:<host>` dork
  link and an email-spider seed, so shadow endpoints get dorked and crawled too.
  It shows a confirmation first — the exact hosts, how many dork links, how many
  spider seeds — before doing anything, is capped (default 10), and never spends
  API quota (link dorks only). Available in the TUI automatically after a
  subdomains/crt.sh/auto run, and on the CLI as `ezrecon auto --chain
  [--chain-cap N]`.

## [2.2.0] — 2026-08-06

Added
- Preview pane. Press `p` on a dork result in the TUI to open a preview window:
  it resolves the dork to real result links (DuckDuckGo when no key, the Google
  API when one is configured), fetches a chosen page, and shows the page title,
  status, and — the useful part — every place a dork term (JCL, LPAR, RACF,
  EBCDIC, an email, etc.) appears, highlighted in surrounding context. `o` opens
  the full page in a browser, `s` adds a natural-language summary (optional,
  needs an Anthropic key), `Esc` closes it. Also available from the CLI as
  `ezrecon preview "<query>" --pages [--summary]`.
- Optional LLM summaries. With an Anthropic API key set, the preview pane can
  summarise a page and point out where the notable material is. Extractive
  (in-context) analysis is always on and needs no key.
- Editable dorks. Each dork in the panel is now an editable field next to its
  tick box, so you can tweak any query — or type your own — before running or
  turning it into links. The Shodan panel already worked this way.

Changed
- Checkboxes show a tick (✓) when on instead of an X, in the dork and Shodan
  panels.
- The preview pane reaches out to third-party search engines and pages, so it
  is active, not passive. It is gated behind a one-time confirmation (with a
  "don't ask again" option stored as `allow_active_fetch`).

New keys/settings: `anthropic_api_key` (optional, for summaries),
`anthropic_model`, and `allow_active_fetch`.

## [2.1.1] — 2026-08-06

Added
- No-key dork mode. The Google Dork panel now has an **Open as links** button
  (and the CLI a `--links` flag) that builds clickable search URLs for every
  selected dork instead of calling the API — no key, no quota, no terms-of-
  service issue. Pick the engine with `--engine google|bing|duckduckgo|
  startpage`. Links are clickable in the TUI and in the exported md/html
  reports; the raw dork strings still go to `queries.txt`.

Fixed
- Google dork errors now show Google's actual reason. Previously a failed
  Custom Search request surfaced only a bare "400 Bad Request"; the module now
  reads the API's JSON error body and reports the real message (e.g. "API key
  not valid" or "project does not have access to Custom Search JSON API"),
  reports it once instead of repeating it for all twenty dorks, and points at
  the no-key link mode as the fix.

## [2.1.0] — 2026-08-06

A round of enrichment and speed work on top of the 2.0 rebuild.

Added
- ASN / netblock enrichment. New module that takes every IP we've resolved and
  looks up its ASN, owner and BGP prefix through Team Cymru's DNS service —
  pure DNS, no API key, no external tool. It groups hosts by netblock, which
  often exposes the organisation's own address space where mainframes cluster.
  Available as `ezrecon asn` and folded into auto-recon by default.
- crt.sh verification. Certificate-transparency names are now resolved
  concurrently and merged with the brute-force results, so a passive hit
  becomes a confirmed live host with IPs instead of just a name. Duplicates
  across the two sources are collapsed.
- Shodan query builder in the TUI. The Shodan module now opens a tick-box panel
  where every mainframe query has its own checkbox and an editable field — tick
  the ones you want, tweak the syntax, or type your own in the blank rows.

Changed
- Auto-recon runs its independent stages (whois, DNS, crt.sh, subdomains,
  email, Shodan) concurrently now instead of one after another, then does the
  dependent stages (banner grab, ASN) afterwards. Roughly halves wall time.
- DNS now defaults to fast public resolvers (1.1.1.1 / 8.8.8.8 / 9.9.9.9) with
  a shared cache, and the subdomain brute force uses a shorter per-query
  timeout (2s) — both configurable. A non-existent name no longer costs 5s.
- crt.sh and the email spider share a pooled HTTP session so connection and TLS
  setup is paid once.

New settings (`ezrecon config`): `brute_timeout` (default 2.0) and
`dns_nameservers` (default: public resolvers; set to `[]` for the system
resolver).

## [2.0.0] — 2026-08-06

Complete ground-up rebuild. The original tool was a single curses menu that
imported modules that were never shipped, so it wouldn't start; every scan it
did run went one host at a time. We threw that out and built EZRecon around a
proper async engine with two front-ends on top.

Added
- Async recon engine. DNS, subdomain brute force, banner grabs, and the email
  spider all run concurrently now, so a sweep that used to take minutes takes
  seconds.
- Textual TUI with a phosphor-green theme. Sidebar modules, a live findings
  table, keyboard and mouse, and it works fine over SSH.
- Scriptable CLI. Every module is a subcommand (`ezrecon dns`, `ezrecon auto`,
  and so on), and running `ezrecon` with no arguments opens the TUI.
- Google Dork multi-select. The mainframe dork catalogue from Chapter 6 is
  built in; tick the ones you want and fire them at a target.
- DNS module. A/AAAA/MX/NS/SOA/TXT/CNAME plus SPF and DMARC parsing, reverse
  lookups, and a zone-transfer (AXFR) attempt against every name server.
- Subdomain discovery. Concurrent brute force with wildcard-DNS detection so a
  wildcard zone doesn't report the whole wordlist as live.
- crt.sh enrichment. Pulls subdomains straight out of certificate-transparency
  logs to catch the shadow endpoints a wordlist never will.
- Shodan module. Ships the z/OS banner query library (TSO, IBM FTP, CICS,
  Db2/DRDA, NJE) so you can sweep for mainframes without memorising syntax.
- Native banner grab (non-nmap). An opt-in async connect-and-read across the
  mainframe port profile, with a fingerprint scorer that flags "likely
  mainframe" and names the service.
- AI-OSINT prompt generator. Emits the deep-research prompt from Chapter 6,
  pre-filled with the target and anything the tool already found.
- Auto-recon. One command runs the whole passive chain into a single session.
- Report export. Markdown, HTML (phosphor-themed), JSON, and a `queries.txt`
  for the lab deliverable.
- API-key vault. Keys live in `~/.config/ezrecon/config.json` (with env-var and
  legacy `api_key.json` fallbacks) instead of being scattered around.

Changed
- Dropped the Nmap component entirely. Active Nmap scanning is its own section
  of the book; EZRecon is the passive-recon workhorse.
- Reworked the engine so it never touches the UI, which means it's unit-tested
  and reusable from the CLI, the TUI, or your own scripts.

Fixed
- Tool no longer crashes on launch from missing imports.
- Run button no longer overflowed off the right edge of the terminal.
