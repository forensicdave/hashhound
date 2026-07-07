# HashHound

**Triage files against multiple malware‑intelligence sources — right from Finder.**

HashHound is a native macOS utility for security researchers, incident responders, and malware
analysts. Select a file in Finder, and HashHound looks it up by hash across **nine** sources at
once — **VirusTotal**, **MalwareBazaar**, **REDS**, **urlscan.io**, **Hybrid Analysis**,
**Team Cymru MHR**, **MalShare**, **AlienVault OTX**, and **Triage** — then shows a single,
glanceable verdict with the detail to dig deeper. No copy‑pasting hashes.

HashHound also talks to **[OpenSourceMalware.com](https://opensourcemalware.com)** (OSM): hand it a
package archive (npm, PyPI, NuGet, VS Code), a dependency **lock file**, or just a package name, and it
checks whether that software is a known‑malicious supply‑chain threat — see
[Checking a package with OpenSourceMalware](#checking-a-package-with-opensourcemalware) below.

**[⬇️  Download the latest release](https://github.com/forensicdave/hashhound/releases/latest)** — macOS 14+, notarized.

---

## Why analysts like it

- **Triage without leaving Finder.** Right‑click any file → get a verdict in seconds. No browser,
  no web UI hopping, no manual hashing.
- **Many sources, one verdict.** It queries every source you've enabled in parallel and shows an
  aggregated headline (worst result wins) plus a per‑source breakdown — so you see *who* flagged
  it and *what* they call it, at a glance.
- **Hash‑first and privacy‑aware.** Files are identified by their **SHA‑256**, computed locally.
  Nothing ever leaves your machine unless you *explicitly* choose to upload or download a sample.
- **Built for a real workflow.** Check many files or paste a batch of hashes at once, keep a
  persistent triage history, export the combined JSON, pipe it straight into your own tooling, or
  pull the sample down to your sandbox.
- **Your keys stay yours.** API keys live in the macOS **Keychain**, never in plaintext, and are
  never written to logs.

If your day involves "is this file known‑bad, and to whom?" — HashHound turns that into a
right‑click.

---

## Features

### Look things up
- **Finder right‑click** — *Lookup in HashHound* on any selected file or files.
- **Scan Files…** — pick one or more files from anywhere; HashHound hashes and looks them up.
- **Drag & drop** — drop one or more files onto the results window to look them up.
- **Scan a folder** — pick or drop a **folder** and HashHound looks up **every file below it,
  recursively** (bundles like `.app` are treated as one item; hidden files are skipped). If a folder
  expands to more than a configurable count (**default 50** — *Settings → Folder Scans*) it asks first,
  since online sources are rate‑limited and large batches take a while.
- **Look Up Hashes…** — paste a batch of **MD5 / SHA‑1 / SHA‑256** hashes (one per line, or
  separated by spaces/commas) and look them up directly — no file needed. Press **⌘Return** to run.
- **Batch friendly** — select many files, or paste many hashes; they all flow into one window.
- **App bundles** — scanning a `.app` (or other bundle) hashes its **main executable**
  (`Contents/MacOS/…`), not the whole folder; the row notes which file it checked.
- **Clipboard aware** — if your clipboard holds a hash when HashHound comes to the front, it
  offers to look it up (toggle in Settings).

### Understand the result
- **Glanceable verdict** per file: 🔴 Malicious · ⚠️ Suspicious · ✅ Clean · 🔎 Seen (no verdict) · 🟦 Not seen.
- **Aggregate across sources** with a clear "flagged by X" summary, plus a **per‑source line**
  (e.g. `VirusTotal 3/72 · MalwareBazaar known · REDS not in dataset`).
- **Expandable detail** — each source is collapsible (use **Expand All / Collapse All**, or click a
  source). VirusTotal's flagged engines and detection ratio, MalwareBazaar's signature/tags,
  REDS's **risk level** and analysis, urlscan.io's referencing URLs, Hybrid Analysis's verdict
  + threat score, and Team Cymru MHR's **AV detection rate** + last‑seen date — each with a link
  to the full report online where available.
- **Accurate "not seen" handling** — MalwareBazaar and REDS are *malware* datasets, so a miss means
  "not a known sample," **not** "clean." HashHound never mislabels that as safe. urlscan.io is
  context (where the file was seen online), not a malware verdict.
- **Scan more on demand** — a configured source that wasn't part of a lookup shows as an *Also scan*
  link on the row; click it to query just that source.
- **OpenSourceMalware** *(needs a free API token)* — a **supply‑chain** malware database. It has **no
  file‑hash lookup**, so HashHound feeds it what it can extract from a scanned file:
  - **Package identity** — scanning an **npm** `.tgz`, **PyPI** `.whl`/`.tar.gz`, **NuGet** `.nupkg`, or
    **VSIX** extracts the package **name + ecosystem** and checks it against OSM.
  - **Lock files** — scanning a **`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `requirements.txt`,
    `Pipfile.lock`, `poetry.lock`,** or **`packages.lock.json`** checks **every dependency at its resolved
    version** and reports which are flagged (a summary + the malicious ones).
  - **Download origin** — the **URL and domain** the file was downloaded from (its quarantine "where
    from") are checked too.
  - **Check by name** — **File → Check in OpenSourceMalware…** (or the *Check in OSM…* button) to check
    a package / URL / domain / IP directly, no file needed. VS Code extension names are resolved to
    `publisher.name` via the Marketplace.
  A hit is 🔴 *Malicious* with OSM's **severity** + affected versions and a link to the OSM report; a
  miss means *"not a known‑malicious asset,"* **not** clean. Add your `osm_` token under **Settings →
  OpenSourceMalware** (and choose whether to check packages, origins, or both). CLI: `--osm`.

### Keep a record
- **Accumulating history** — results pile up in one window (newest first) and **persist across
  quits**, so you have a running record of what you've triaged.
- **Search & filter** — a filter bar narrows the history by name, path, or hash, with a
  **Flagged only** checkbox to show just malicious/suspicious rows.
- **Refresh** any row to re‑query all sources; **Clear** to wipe the history.

### Pivot and integrate
- **Open in &lt;source&gt;** — jump to the full web report on whichever sources have the file
  (VirusTotal, MalwareBazaar, REDS, urlscan.io, Hybrid Analysis).
- **Copy JSON / Export JSON…** — the **combined** report from all sources as one JSON object.
- **Copy Markdown Report** — the same paste‑ready Markdown table the CLI's `--format markdown`
  produces, for tickets and docs.
- **Copy Cleaned JSON** — a trimmed copy of the combined report (drops empty fields, keeps only the
  flagged VirusTotal engines) sized to fit AI context limits.
- **Send to AI** — fills your prompt template with the cleaned report and hands it off: it copies
  the prompt + JSON and opens your chosen AI site to paste into (warning first if the payload is
  large). Set the prompt, AI site, and warning size in Settings.
- **Open in a helper app** — send the report JSON to your preferred viewer/editor.
- **Run CLI** — pipe the full JSON to a command you configure (e.g. `jq`, a Python script) and see
  its output.
- **Run in Terminal** — run the same command in a live Terminal window.

### Submit and analyze
- **Upload to VirusTotal** — submit a file VirusTotal hasn't seen for analysis (explicit, with a
  privacy warning).
- **Submit to MalwareBazaar** — contribute a confirmed‑malware sample (public; gated behind an
  "I confirm this is malware" checkbox, with optional tags and anonymous submission).
- **Upload to REDS** — add a sample to your REDS dataset, with an optional **private** upload.
- **Submit to Hybrid Analysis** — detonate a file in a sandbox **environment you pick** (your
  account's available Windows/Linux/macOS/etc.); refresh later for the verdict.
- After any submission, **Refresh** the row to pick up the new analysis.

### Download samples (for analysis in your sandbox)
- **Encrypted ZIP preferred** — downloads come as a **password‑protected ZIP** (password:
  `infected`) wherever the source supports it, so samples move safely.
- **Download As Raw** (optional, off by default) — for sources that offer it, save the raw binary
  directly. Raw files are saved **without executable permissions** and named by hash
  (`<hash>.vt`, `<hash>.reds`) so they can't be run by a double‑click.
- Every download shows a clear warning — handle malware only in an isolated environment.

### Stay within limits
- **Service Quotas** window (*File → Service Quotas…*) shows your current usage per source
  (VirusTotal API requests, REDS search/upload/download), with a Refresh button.
- **Request throttling** keeps you within VirusTotal's free‑tier rate limit automatically.

### Support & diagnostics
- **Debug logging** (optional, off by default) writes timestamped diagnostics to a log file you can
  send for support. API keys are never logged.

---

## The sources

| Source | What it tells you | Sample download | Submit |
|---|---|---|---|
| **VirusTotal** | 70+ AV engines + sandboxes; detection ratio, names, reputation | Raw binary (premium key) | Upload for analysis |
| **MalwareBazaar** (abuse.ch) | Known‑malware corpus; family signature, tags, first/last seen | Password‑protected ZIP | Public submission |
| **REDS** (RationalEdge DataSet) | Malware dataset with code/feature analysis; status, file type, collections | ZIP (raw fallback) | Upload (private option) |
| **urlscan.io** (Files search) | Where the file has been seen online — the URLs/scans that referenced it (context, not a verdict) | — | — |
| **Hybrid Analysis** (Falcon Sandbox) | Sandbox verdict (malicious/suspicious/clean) + 0–100 threat score, file type, AV scanners, tags | — | Submit for sandbox analysis |
| **Team Cymru MHR** (Malware Hash Registry) | Registry of known‑malware hashes with an AV detection % and last‑seen date | — | — |
| **MalShare** | Community malware repository; file type, SSDEEP, source URLs, daily‑quota view | Raw binary (opt‑in) | Public submission |
| **AlienVault OTX** | Community threat reports ("pulses") referencing the hash; malware families | — | Submit for analysis (choose TLP) |
| **Triage** (tria.ge) | Public sandbox verdict with a 1–10 score, malware family, and tags | — | — |

You choose which sources to enable. Each is independent — HashHound works fine with just one.

---

## Requirements

- **macOS 14 (Sonoma) or later.**
- **API keys** for the sources you want to use:
  - **VirusTotal** (free) — <https://www.virustotal.com/gui/my-apikey>
  - **MalwareBazaar (abuse.ch Auth‑Key)** (free) — <https://auth.abuse.ch/>
  - **REDS** — <https://reds.rationaledge.io/> — access may not be free; contact RationalEdge to request access
  - **urlscan.io** — <https://urlscan.io/user/apikey/> — Files (hash) search requires a **paid** urlscan.io plan
  - **Hybrid Analysis** (Falcon Sandbox) (free) — <https://hybrid-analysis.com/my-account?tab=%23api-key-tab>
  - **Team Cymru MHR** (free) — sign up at <https://www.team-cymru.com/mhr> to get a **username and
    password**, then enter both under **Settings → Team Cymru MHR** and click **Test**. (MHR uses
    HTTP Basic auth — a username and password — not a single API key.)
  - **MalShare** (free) — <https://malshare.com/register.php>
  - **AlienVault OTX** (free) — <https://otx.alienvault.com/api>
  - **Triage** (free researcher account) — <https://tria.ge/account>

---

## Checking a package with OpenSourceMalware

OpenSourceMalware (OSM) checks a package's **identity** — its `name` + `ecosystem` — against a curated
supply‑chain malware database. It is **not** analysing the code; it answers *"is this published package
known‑malicious?"* A hit is a strong signal; a miss (🟦 *Not in OSM*) means *"not a **known**‑bad
package,"* **not** "this code is safe." HashHound checks **files**, so the workflow is: get the package
as a file, then scan it.

**Setup (once):** Settings → **OpenSourceMalware** → paste your `osm_` token → **Save** → **Test**, and
keep **Check package archives** on.

**1. Work out which _version_ you'd actually get.** Malicious releases usually target **specific
versions**, so if you haven't pinned one, first find the version an install would resolve to (the latest,
or whatever a tag/range picks) — that's the artifact worth checking:

```bash
# PyPI — list available versions (newest first) / see the latest:
pip index versions requests
python3 -m pip download requests --no-deps -d ./out   # downloads exactly what the resolver picks

# npm — the version a plain install would take, a dist-tag, or all versions:
npm view requests version
npm view requests dist-tags
npm view requests versions

# NuGet:
nuget list <PackageId>
```

**2. Fetch that exact version as a file:**

```bash
npm pack requests@2.31.0                       # → requests-2.31.0.tgz
pip download requests==2.31.0 --no-deps -d out # → requests-2.31.0-*.whl (or .tar.gz)
# NuGet: download the .nupkg; VS Code: download the .vsix
```

**3. Scan the file in HashHound** — right‑click → *Lookup in HashHound*, *Scan Files…*, or drag it onto
the window. HashHound sniffs the archive **by content** (so a package renamed to its hash, or with no
extension, still works), extracts the identity, and queries OSM. CLI: `hashhound --osm ./requests-2.31.0.tgz`.

**4. Read the result** in the OpenSourceMalware section:
- 🔴 **Malicious** — shows OSM's **severity**, the **affected versions**, and a link to the OSM report,
  and feeds the overall verdict. **Compare your version against the affected‑versions list** — the package
  may be compromised only in certain releases, so confirm yours is one of them.
- 🟦 **Not in OSM** — not a known‑malicious package (still not a "clean" verdict).

Supported ecosystems: **npm** (`.tgz`), **PyPI** (`.whl` / `.tar.gz`), **NuGet** (`.nupkg`), **VS Code**
(`.vsix`). OSM's lookup is case‑sensitive, so HashHound also tries a few publisher/name casings. HashHound
queries at the **package level** (any version) to maximise detection, then shows OSM's affected versions so
you can judge whether the exact version you'd install is affected.

### Check a whole project at once — scan its lock file

The steps above are for one package. To audit an entire dependency tree, just **scan the project's lock
file** — `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `requirements.txt`, `Pipfile.lock`,
`poetry.lock`, or `packages.lock.json`. HashHound reads **every dependency at its resolved version**
(so the "which version?" step is answered for you) and checks each against OSM, reporting a summary plus
the flagged ones:

```bash
hashhound --osm ./package-lock.json      # or drag it onto the window / right-click in Finder
# → OpenSourceMalware: "1 malicious dependency — Checked 812 of 812 — 1 flagged"
```

Checks are batched and stop early if OSM rate‑limits, and very large trees are capped (with a note). Only
the flagged dependencies are listed; a clean tree just gets a 🟦 "No known‑malicious dependencies" headline.

### Check by name (no download)

**File → Check in OpenSourceMalware…** (or the *Check in OSM…* button) opens a paste box to check package
names / URLs / domains / IPs directly. Press **⌘↵** to run.

OSM itself matches **exact** names, but HashHound can resolve looser input first:

- **Partial / keyword search** — append a `*` (e.g. `express*`, `newtonsoft*`) and HashHound resolves
  candidates via the ecosystem's own registry (**npm** and **NuGet** search, the **VS Code** Marketplace),
  then checks each exact match against OSM. A small candidate cap keeps it within OSM's rate limit, and
  every matched package is listed in the result, so you can see exactly what was checked.
- **VS Code** — just enter the extension name; HashHound resolves the `publisher.name` via the Marketplace.
- **PyPI** — exact lookups also try the PEP 503 normalized name (case‑ and `-`/`_`/`.`‑insensitive), so
  `Flask_SQLAlchemy` finds `flask-sqlalchemy`. A `name*` search (e.g. `ibo2*`) matches against a
  locally‑cached copy of PyPI's package index, since PyPI has no live search API — the **first** such
  search downloads the index (~40 MB, then cached and refreshed daily in the background). It only finds
  packages **still live on PyPI**; ones already removed after being flagged won't appear.

> **Tip:** to check a name **without downloading**, you can query OSM directly:
> `curl -H "Authorization: Bearer $OSM_TOKEN" "https://api.opensourcemalware.com/functions/v1/check-malicious?report_type=package&resource_identifier=<name>&ecosystem=npm"`

---

## Install

1. **Move `HashHound.app` to your `Applications` folder** and launch it.
   - On first launch, if macOS shows a Gatekeeper prompt, approve HashHound in **System Settings →
     Privacy & Security** (or right‑click the app → **Open**).
2. **Enable the Finder extension:** **System Settings → General → Login Items & Extensions** →
   turn on **HashHound Finder**.
3. **Add your API keys:** open HashHound's **Settings** (it opens automatically on first run), paste
   your keys for the sources you want, and click **Test** to confirm each one. Keys are saved to
   your Keychain.

That's it — right‑click a file in Finder and choose **Lookup in HashHound**.

---

## Using HashHound

- **From Finder:** right‑click a file → *Lookup in HashHound*.
- **From the app:** *File → Scan Files…* to pick files, or *File → Look Up Hashes…* to paste hashes.
- **In the results window:** read the headline verdict, expand a source for detail, click a report
  link, or use the per‑row actions (Refresh, Copy/Export JSON, Run CLI/Terminal, Submit/Upload,
  Download). History stays until you press **Clear**.
- **Quotas:** *File → Service Quotas…*.
- **Settings (⌘,):** keys, rate limit, external‑program command + helper app, **AI** prompt/site +
  paste‑size warning, **command‑line‑tool install**, raw‑download toggle, and debug logging.

### A good first test

The **EICAR test file** is a harmless, industry‑standard antivirus test sample that every engine
recognizes — ideal for confirming your keys work and seeing the sources light up. Its SHA‑256:

```
275a021bbfb6489e54d471899f7db9d1663fc695ec2fe2a2c4538aabf651fd0f
```

Use **File → Look Up Hashes…**, paste it, and press **⌘Return**.

---

## Command line (`hashhound`)

HashHound bundles a `hashhound` command‑line tool that uses the same engine and your same API keys,
and outputs **JSONL** (one JSON object per line) — handy for scripting and pipelines.

**Install it:** **Settings → Command Line Tool → Install Command Line Tool…** (it symlinks
`hashhound` into `/usr/local/bin`, asking for your admin password only if needed). Then:

```
hashhound --help
```

Examples:

```
# a single hash (MD5 / SHA-1 / SHA-256)
hashhound 275a021bbfb6489e54d471899f7db9d1663fc695ec2fe2a2c4538aabf651fd0f

# one or more files (each hashed locally; a .app uses its executable)
hashhound /path/to/sample.bin ~/Downloads/suspect.dmg

# a whole directory, recursively, via VirusTotal + Hybrid Analysis only
hashhound -r --sources virustotal,hybridanalysis ~/quarantine

# a list of hashes piped in, written to a file as well as stdout
cat hashes.txt | hashhound --stdin -o results.jsonl

# scripting: exit non-zero if anything is malicious
hashhound --fail-on-malicious *.bin && echo "all clear"
```

Output defaults to **JSONL**; use `-f/--format markdown` or `-f/--format text` (or the `--markdown` /
`--text` shortcuts) for a readable report or a compact aligned summary.


Options: `-s/--sources`, `-o/--out`, `-f/--format`, `-r/--recursive`, `--stdin`, `--raw`, `--debug`,
`--fail-on-malicious`, `-q/--quiet`, `-h/--help`, `--version`. Run `hashhound --help` for the full list.

**API keys:** read from the HashHound Keychain entries (allow access when prompted), or — for
headless/CI use — from environment variables: `HASHHOUND_VIRUSTOTAL_API_KEY`,
`HASHHOUND_MALWAREBAZAAR_API_KEY`, `HASHHOUND_REDS_API_KEY`, `HASHHOUND_URLSCAN_API_KEY`,
`HASHHOUND_HYBRIDANALYSIS_API_KEY`, `HASHHOUND_MALSHARE_API_KEY`, `HASHHOUND_OTX_API_KEY`,
`HASHHOUND_TRIAGE_API_KEY`. Team Cymru MHR uses HTTP Basic — set
`HASHHOUND_CYMRU_API_KEY="username:password"` (or `HASHHOUND_CYMRU_USERNAME` + `HASHHOUND_CYMRU_PASSWORD`).

---

## Privacy & safety

- **Hash‑only by default.** Hashes are computed on your Mac; HashHound only sends the hash to look
  it up. Your file is **never** uploaded unless you explicitly press an Upload/Submit button.
- **MalwareBazaar submissions are public** — only submit confirmed malware; the app makes you
  confirm.
- **Sample downloads are real malware.** ZIPs are password‑protected (`infected`); raw downloads
  are opt‑in, saved without executable permissions, and named by hash. Only handle samples in an
  isolated/sandboxed environment.
- **API keys** are stored in the macOS Keychain and are never included in the debug log.

---

## Troubleshooting

**The right‑click menu doesn't appear.**
Make sure **HashHound Finder** is turned on in **System Settings → General → Login Items &
Extensions**, then relaunch Finder (log out/in if needed). The menu item appears in the right‑click
menu — sometimes under **Quick Actions**, so scroll the whole menu.

**"No source configured."**
Open **Settings** and add at least one API key, then use the **Test** button to confirm it.

**A source shows an error.**
- *VirusTotal:* "Wrong API key" → re‑paste your key. "Rate limit reached" → you've hit the free
  tier (~4/min, 500/day); wait, or raise the throttle in Settings. Downloading a sample needs a
  **premium** VirusTotal key.
- *MalwareBazaar:* requires a free **Auth‑Key** from abuse.ch. "Auth‑Key rejected" → re‑paste it and
  watch for stray spaces.
- *REDS:* check your token in Settings; some actions (like private uploads) require a premium plan.

**Capture diagnostics for support.**
Turn on **Settings → Diagnostics → Turn on debugging**, reproduce the problem, then click **Reveal
Log in Finder** and send the log file (`~/Library/Logs/HashHound/HashHound.log`). It contains
timestamps, hashes, and request status codes — **never your API keys**.

---

## A note on hash types

SHA‑256 gives the **fullest coverage** — every source supports it. MD5 and SHA‑1 lookups work on
every source except **REDS (SHA‑256‑only)**, which will show "Needs SHA‑256" for an MD5/SHA‑1.
When in doubt, look up by SHA‑256 — and note that scanned files get their MD5/SHA‑1 computed
locally anyway (shown under **File Info**), so you can pivot either way.

---

## Disclaimer

HashHound is provided **as‑is, without warranty of any kind — use at your own risk.** It is a triage
aid: always corroborate verdicts and handle malware samples responsibly in an isolated environment.

You supply your **own API keys** and are responsible for using each service within its terms of
service and rate limits. HashHound is **not affiliated with or endorsed by** VirusTotal,
MalwareBazaar (abuse.ch), REDS (RationalEdge), urlscan.io, Hybrid Analysis (CrowdStrike),
Team Cymru, MalShare, AlienVault OTX, or Triage (Recorded Future); all trademarks belong to
their respective owners.
