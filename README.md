# ◊ FallBooksOnboard

**Live:** [sjgant80-hub.github.io/fallbooksonboard](https://sjgant80-hub.github.io/fallbooksonboard/)

**Sovereign accountancy-firm client onboarding · AML CDD · PSC + beneficial-ownership capture · engagement letter draft · audit chain · single HTML file.**

`fallbooksonboard` is the AML/onboarding anchor of the `fallbooks` bundle for UK accountancy practices (1–10 person firms; ICAEW / ACCA / AAT / CIMA / HMRC-supervised). It captures every client per the shared `BookClient` schema and broadcasts on `BroadcastChannel('fall-books')` so `fallbooks`, `fallbookspaper`, and `fallbookspractice` stay in sync.

- Prime: **773**
- Version: **1.0.0**
- Mesh: `fall-books` + `fall-signal`
- Storage: IndexedDB (`fallbooksonboard.v1`) — never leaves the device
- License: MIT

---

## For practitioners

### What this does

1. **Multi-step onboarding wizard** (11 steps) capturing the full `BookClient` shape: entity type · identity · contact · beneficial ownership + PSC · services engaged · CDD core + risk grade · source of funds + business profile · PEP + sanctions · documents (SHA-256 hashed) · engagement-letter inputs · commit.
2. **Per-client AML risk grade** (low / medium / high) — auto-suggested from PEP, cash-intensive sector, high-risk geography, entity complexity, sanctions result; overridable with logged reasoning. Drives the next-review cadence (1yr / 6mo / 3mo).
3. **Beneficial ownership + PSC capture** — manual entry plus CSV import. Companies House deep-link to the live PSC register.
4. **Document capture** — drag-drop, SHA-256 + IDB Blob, expiry tracking. Photo ID, proof of address, cert of incorporation, latest accounts, PSC register copy, proof of registered office, authority letter.
5. **Engagement letter** — inputs captured in the wizard; full markdown draft auto-generated on commit, hashed, queued for `fallbookspaper`.
6. **AML supervisor management** — firm-level (HMRC / ICAEW / ACCA / CIMA / AAT / CIOT / ATT / IFA), per-client override.
7. **PSC cache** — sovereign mirror of Companies House data (manual refresh, mesh-fed). Filter + export.
8. **Dashboard** — overdue AML reviews, due-within-30-days, PEP flags, sanctions matches, drafts in progress.
9. **AML Help (T0)** — 12 built-in rules answer the common questions (MLR 2017, supervisor identity, PSC definition, 25% threshold, EDD triggers, PEP, sanctions sources, Companies House deadlines, engagement letter contents, fee disclosure, conflict of interest, retention 5/6/7yr).
10. **Audit chain** — every state change appended with prevHash + docHash (SHA-256); 7-year retention by default; verify-chain and export-JSON in one click.

### First launch

1. Open `index.html` in Chrome 113+ (or any modern Chromium / Firefox).
2. Set up your **firm** (name, practice type, professional body, AML supervisor, HMRC agent ref, address).
3. Add the **first adviser** (you).
4. A demo client `DEMO · Marcus Osei Trading Ltd · overwrite me` is seeded so you can see a fully-onboarded record. Delete or overwrite.
5. Hit `+ client` to begin a real onboarding.

### The wizard, step by step

| # | Step | Captures |
|---|------|----------|
| 1 | Entity | sole-trader / partnership / LLP / Ltd / charity / trust / public-body / other |
| 2 | Identity | individuals: title/name/DOB/NINO/UTR. Entities: name, trading name, CRN, incorp date, CT UTR, VAT no, VAT scheme, PAYE ref, period end, year end, share capital |
| 3 | Contact | registered + trading address, email, phone, address history |
| 4 | BO / PSC | beneficial owners (manual) + PSC mirror (manual or CSV) |
| 5 | Services | accounts / CT / VAT / payroll / SA / tax planning / bookkeeping / CH filings |
| 6 | CDD core | cash-intensive flag, high-risk geo flag, auto-suggested risk grade, override + reasoning |
| 7 | Source of funds | source for fees, nature & purpose of business, expected transaction patterns |
| 8 | PEP / Sanctions | PEP flag + details, sanctions screen status (with OFSI / UK / UN deep-links) |
| 9 | Documents | drop-zone capture, SHA-256, IDB Blob, expiry days |
| 10 | Engagement | type, fee basis (fixed-monthly / hourly / fixed-annual / per-job), amount, frequency, scope, exclusions |
| 11 | Commit | review summary + gap list, then write to IDB + broadcast `client.created` + `engagement.signed` + queue letter draft for `fallbookspaper` |

### Outputs

- **CDD certificate** (markdown) — per client, evidences CDD measures held at issue.
- **Engagement letter** (markdown) — 10-section draft per ICAEW/ACCA conventions, signed-hash included.
- **Audit chain export** (JSON) — full chain with hashes.
- **Snapshot export / import** (JSON) — firm + advisers + clients + PSC cache.

### Cross-tool mesh

| Channel | Sent | Received |
|---------|------|----------|
| `fall-books` | `client.created`, `client.updated`, `adviser.updated`, `firm.updated`, `engagement.signed`, `engagement.draftReady`, `psc.fetched`, `sync.snapshot` | `sync.request`, `client.*`, `adviser.*`, `firm.*`, `psc.fetched` |
| `fall-signal` | `hello`, `pong` | `ping` |

On boot the tool emits `sync.request` and merges any returning `sync.snapshot` by `updatedAt`. All cross-tool writes ship the **full new record**, not a diff. Broadcasts are debounced 300ms.

### Risk → review cadence

| Grade | Next review |
|-------|-------------|
| low | 365 days |
| medium | 180 days |
| high | 90 days |

The dashboard surfaces overdue and within-30-days at a glance.

### Sovereignty

- One HTML file. No build step. No CDN dependencies (except Google Fonts).
- IndexedDB primary. localStorage not used.
- No telemetry. No analytics. No network calls except (optional) Claude BYOK from the AML Help tab and Google Fonts CSS.
- BYOK key lives in IDB only. Never broadcast on any channel.

### Disclaimer (verbatim)

> FallBooks is a tool for UK accountancy practices. It assists with multi-client bookkeeping, deadline tracking, SA/CT/VAT/PAYE summary preparation, engagement letters, and practice management. It is not an HMRC-approved filing system; submissions to HMRC/Companies House remain the practitioner's responsibility. Sovereign — client data never leaves the device unless exported.

---

## For developers

### File anatomy

- `index.html` — the entire app (HTML + CSS + vanilla JS, ~3000 LoC).
- `README.md` — this file.
- `LICENSE` — MIT.
- `.nojekyll` — required for GitHub Pages to serve files starting with `_` and to skip Jekyll processing.

### Constants (top of script block)

```js
const TOOLNAME='fallbooksonboard';
const VERSION='1.0.0';
const PRIME=773;
const SCHEMA_V=1;
const STORE='fallbooksonboard.v1';
```

### IDB stores

| Store | KeyPath | Purpose |
|-------|---------|---------|
| `firms` | `id` | single firm record (multi supported on import) |
| `advisers` | `id` | partners + staff |
| `clients` | `id` | `BookClient` records per shared schema |
| `cddDocuments` | `id` | uploaded files (Blob + meta) referenced from `client.kyc.documentsHeld[].blobRef` |
| `pscCache` | `id` (= CRN) | mirror of PSC info from Companies House |
| `audit` | `i` | append-only audit chain |
| `settings` | `k` | tier, BYOK key, active adviser |
| `engagementQueue` | `id` | engagement-letter drafts awaiting pickup by `fallbookspaper` |

### Audit chain

```
docHash = sha256({prevHash, ts, action, clientId, payload})
```

Genesis prevHash = `'GENESIS'`. Click *audit → verify chain* to recompute.

### Client schema

Conforms exactly to `BOOKS-BUNDLE-SHARED-SCHEMA.md` `BookClient`. Use `window.FALLBOOKSONBOARD.state().clients[0]` in DevTools to inspect.

### Mesh contract

Open both channels:

```js
new BroadcastChannel('fall-books');   // domain data
new BroadcastChannel('fall-signal');  // presence & ops
```

Sibling tools can fetch and broadcast PSC info:

```js
bc.postMessage({v:1,type:'psc.fetched',source:'mytool',payload:{
  id:'12345678',companyName:'Patel Wealth Ltd',
  pscs:[{name:'A Patel',natureOfControl:'ownership-of-shares-75-to-100-percent',dob:'1980-03',nationality:'GB'}],
  fetchedAt:Date.now()
}});
```

The cache updates and re-renders automatically.

### Remote trigger

```js
window.postMessage({target:'fallbooksonboard',action:'ping',id:1});
window.postMessage({target:'fallbooksonboard',action:'snapshot',id:2});
```

### Add a T0 rule

Append to the `T0_RULES` array near the top of the `<script>` block:

```js
{id:'newrule',q:['keyword1','keyword2'],
  title:'Rule title',
  body:'Full markdown body…',
  refs:['SI 2017/692 reg X','ICAEW guidance Y']}
```

### Bundle siblings

| Tool | Prime | Role |
|------|-------|------|
| `fallbooks` | 769 | anchor — multi-client bookkeeping, SA/CT/VAT/PAYE prep |
| **`fallbooksonboard`** | **773** | **this — AML CDD + PSC + engagement** |
| `fallbookspaper` | 797 | document generator (SA100/CT600/management accounts/letters) |
| `fallbookspractice` | 809 | deadlines + agent auths + PI + AML supervision |

### 14-pt sovereign gate

| Gate | Status |
|------|--------|
| Single HTML file | ✓ |
| <400 KB | ✓ |
| IDB primary | ✓ |
| KONOMI shim | ✓ (`window.KONOMI`) |
| fall-books mesh | ✓ |
| fall-signal mesh | ✓ |
| PWA manifest (data: URL) | ✓ |
| Mobile-first responsive | ✓ |
| MIT license | ✓ |
| README (two-audience) | ✓ |
| `.nojekyll` | ✓ |
| Disclaimer | ✓ |
| Audit chain | ✓ |
| Oxblood / brass / cream / void | ✓ |

### Browser support

Chrome 113+, Edge 113+, Firefox 115+, Safari 16.4+. `BroadcastChannel`, `crypto.subtle.digest`, IndexedDB Blob storage required.

### Deploy on GitHub Pages

```
git init && git add -A && git commit -m "ship fallbooksonboard v1.0.0"
git branch -M main
git remote add origin git@github.com:<you>/fallbooksonboard.git
git push -u origin main
# enable Pages → Build from branch: main / root
```

`.nojekyll` + serving `index.html` from root is all that's required.

---

**◊ 773 · sovereign**
