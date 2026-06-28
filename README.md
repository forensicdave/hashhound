# HashHound

**Triage files against multiple malware‑intelligence sources — right from Finder.**

HashHound is a native macOS utility for security researchers, incident responders, and malware
analysts. Select a file in Finder, and HashHound looks it up by hash across **VirusTotal**,
**MalwareBazaar**, **REDS**, **urlscan.io**, and **Hybrid Analysis** at once, then shows a single,
glanceable verdict with the detail to dig deeper. No copy‑pasting hashes into several websites.

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
- **Look Up Hashes…** — paste a batch of **MD5 / SHA‑1 / SHA‑256** hashes (one per line, or
  separated by spaces/commas) and look them up directly — no file needed. Press **⌘Return** to run.
- **Batch friendly** — select many files, or paste many hashes; they all flow into one window.
- **App bundles** — scanning a `.app` (or other bundle) hashes its **main executable**
  (`Contents/MacOS/…`), not the whole folder; the row notes which file it checked.

### Understand the result
- **Glanceable verdict** per file: 🔴 Malicious · ⚠️ Suspicious · ✅ Clean · 🔎 Seen (no verdict) · 🟦 Not seen.
- **Aggregate across sources** with a clear "flagged by X" summary, plus a **per‑source line**
  (e.g. `VirusTotal 3/72 · MalwareBazaar known · REDS not in dataset`).
- **Expandable detail** — each source is collapsible (use **Expand All / Collapse All**, or click a
  source). VirusTotal's flagged engines and detection ratio, MalwareBazaar's signature/tags,
  REDS's **risk level** and analysis, urlscan.io's referencing URLs, and Hybrid Analysis's verdict
  + threat score — each with a link to the full report online.
- **Accurate "not seen" handling** — MalwareBazaar and REDS are *malware* datasets, so a miss means
  "not a known sample," **not** "clean." HashHound never mislabels that as safe. urlscan.io is
  context (where the file was seen online), not a malware verdict.
- **Scan more on demand** — a configured source that wasn't part of a lookup shows as an *Also scan*
  link on the row; click it to query just that source.

### Keep a record
- **Accumulating history** — results pile up in one window (newest first) and **persist across
  quits**, so you have a running record of what you've triaged.
- **Refresh** any row to re‑query all sources; **Clear** to wipe the history.

### Pivot and integrate
- **Open in &lt;source&gt;** — jump to the full web report on whichever sources have the file
  (VirusTotal, MalwareBazaar, REDS, urlscan.io, Hybrid Analysis).
- **Copy JSON / Export JSON…** — the **combined** report from all sources as one JSON object.
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
  paste‑size warning, raw‑download toggle, and debug logging.

### A good first test

The **EICAR test file** is a harmless, industry‑standard antivirus test sample that every engine
recognizes — ideal for confirming your keys work and seeing the sources light up. Its SHA‑256:

```
275a021bbfb6489e54d471899f7db9d1663fc695ec2fe2a2c4538aabf651fd0f
```

Use **File → Look Up Hashes…**, paste it, and press **⌘Return**.

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
VirusTotal, MalwareBazaar, urlscan.io, and Hybrid Analysis; **REDS is SHA‑256‑only** and will show
"Needs SHA‑256" for an MD5/SHA‑1. When in doubt, look up by SHA‑256.

---

## Disclaimer

HashHound is provided **as‑is, without warranty of any kind — use at your own risk.** It is a triage
aid: always corroborate verdicts and handle malware samples responsibly in an isolated environment.

You supply your **own API keys** and are responsible for using each service within its terms of
service and rate limits. HashHound is **not affiliated with or endorsed by** VirusTotal,
MalwareBazaar (abuse.ch), REDS (RationalEdge), urlscan.io, or Hybrid Analysis (CrowdStrike); all
trademarks belong to their respective owners.
