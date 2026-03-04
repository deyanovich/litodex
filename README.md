# Litodex — Version Control for Humanity's Texts

Litodex is a platform for version-controlled, verified, and collaborative management of literary and sacred texts. It provides permanent identifiers, scholarly workflows, and a foundation for applications like the Litogram typing practice app.

## System

The Litodex system is a combination of:
- Git: standard source control
- Litogramma: markup language achieving two things:
  - adapting Git's line-based atomicity to humanities for easy diffing
  - providing a canonical form suitable for deterministic hashing
- A workflow system with a tiered, federated distributed nodes called "bibliothecae" designed to produce authoritative, community consensus-based, versions of texts, substantiated by sourcing evidence

The Litodex system consists of two applications:
1. `lit`, ("humanities git") a standalone Rust binary that uses Git internally as its storage engine. Git is an **implementation detail hidden from the user** — users never run `git` commands directly. The repository format is opaque: `lit` uses a non-standard storage directory (not `.git`) to prevent accidental or intentional use of raw `git` commands that would bypass workflow restrictions. This hiding is intentional and necessary: it enforces branch-naming conventions, source metadata requirements, signing requirements, and convergence rules that raw Git would not enforce.
2. `litodex`, ("humanities GitHub", but open source and federated), a server with user management, GPG signature management, and workflows

### What Lives Where

**In Git (managed by `lit` CLI):**
- Text content (the actual literary/sacred texts)
- Source metadata (in commit trailers and cumulative `sources.toml`)
- Convergence history (the commit graph)
- Versiones (annotated tags)

**In the bibliotheca server (managed by `litodex` server):**
- User accounts and authentication
- Certificate/key management (including Tier 2-issued custos keys)
- Role and permission assignments (curator, custos)
- Voting records and consensus state
- Federation peering and sync configuration

The `lit` CLI talks to Git for content operations and to the bibliotheca server API for user, role, and voting operations.

## Core Philosophy

- **One work = one codex** — not per edition, not per user
- **Stemmata = traditions** — multiple authoritative versions coexist as branches
- **No single master** — scholarship has no single source of truth
- **Manuscripts are first-class** — `ms/` stemmata alongside editions
- **Sources are sacred** — every change must be traceable to a verifiable source
- **Consensus-driven** — public editions emerge from community agreement, not maintainer fiat
- **Permanent identifiers** — every snapshot gets a citable LID
- **Lightweight markup** — Litogramma annotations make parsing trivial
- **Federated by design** — no central control, multiple sovereign nodes

## Core Terminology

| Git | Litodex (Formal) | Litodex Alias | When to Use |
|-----|------------------|---------------|-------------|
| **repository** | codex | (none) | `lit codex init`, `lit codex list` |
| **root branch** | radix | (none) | Special stemma with `meta.toml` |
| **branch** | stemma | `sm` | Any textual tradition |
| **tag** | versio | `ver` | Frozen snapshot with date |
| **commit** | actum | `act` | Recorded change |
| **log** | historia | `hist` | History of acts |
| **diff** | delta | `delta` | Difference (Δ) |
| **status** | status | `st` | Current state |
| **merge** | convergere | `con` | Combine proposals into a stemma |

## The Bibliotheca Federation

Litodex is not a single service but a **federation of sovereign nodes** called **bibliothecae** (singular: bibliotheca). Each bibliotheca is an independent server running the Litodex software, configured for its role in the scholarly ecosystem.

### The Three-Tier Hierarchy

```mermaid
graph TB
    subgraph "Tier 3: International"
        I[litodex.org<br/>Global Registry]
        I -->|only syncs metadata| I2[mirror.litodex.org]
    end

    subgraph "Tier 2: National"
        N1[bibliotheca.fr<br/>Academy of Letters]
        N2[bibliotheca.de<br/>National Library]
        N3[bibliotheca.br<br/>Academy of Sciences]
        N4[bibliotheca.in<br/>Scholarly Council]
    end

    subgraph "Tier 1: Operational"
        N1 --> U1[Sorbonne<br/>bibliotheca.sorbonne.fr]
        N1 --> U2[CNRS<br/>bibliotheca.cnrs.fr]
        N2 --> U3[Heidelberg<br/>bibliotheca.uni-hd.de]
        N2 --> U4[LMU<br/>bibliotheca.lmu.de]
        N3 --> U5[USP<br/>bibliotheca.usp.br]
        N3 --> U6[UFRJ<br/>bibliotheca.ufrj.br]
        
        U7[Private Scholar<br/>bibliotheca.smith.org]
        U8[Monastery<br/>bibliotheca.montecassino.it]
        U9[Digital Library<br/>bibliotheca.perseus.org]
    end
    
    style I fill:#f5f5f5,stroke:#333,stroke-width:3px
    style N1 fill:#e1f5fe,stroke:#01579b
    style N2 fill:#fff3e0,stroke:#e65100
    style N3 fill:#e8f5e8,stroke:#1b5e20
    style N4 fill:#f3e5f5,stroke:#4a148c
```

### Tier Definitions

| Tier | Type | Acts Enabled? | Can Issue LIDs? | Primary Function |
|------|------|---------------|-----------------|------------------|
| 1 | Operational | ✅ YES | ❌ NO | Create content, build consensus |
| 2 | National | ❌ NO | ✅ YES | Validate, preserve, issue national LIDs |
| 3 | International | ❌ NO | ✅ YES | Sync, detect conflicts, confirm global LIDs |

#### Key Insight
**Tier 2 and Tier 3 run the exact same software** — just with different configurations and peer relationships. The only difference is what they peer with.

---

## Tier 1: Operational Bibliothecae

**Who:** Universities, research institutions, private scholars, monasteries, museums, digital libraries

**What they do:**
- Create and modify content (`act`, `prop`, `priv`)
- Host their own stemmata with full version control
- Build consensus within their community
- Push converged works to their national bibliotheca
- Can peer directly with other Tier 1 for collaboration

### Technical Capabilities
- Full read/write access to their own stemmata
- Can host both public and private works
- Responsible for their own authentication
- Must peer with their national bibliotheca (but can also peer with others)
- **Cannot issue LIDs** — must request them from Tier 2

### Example Workflow

```bash
# A scholar at Sorbonne creates work
$ lit codex init grc/homer-iliad \
  --bibliotheca=bibliotheca.sorbonne.fr

# Work lives locally at:
# https://bibliotheca.sorbonne.fr/grc/homer-iliad

# Build consensus, create edition
$ lit sm create prop/iliad-sorbonne-xkm
$ lit act -m "Initial text" --source="..."
# ... discussion, voting ...
$ lit converge prop/iliad-sorbonne-xkm --into=ed/iliad-sorbonne

# When ready, push to national level
$ lit push national --to=bibliotheca.fr
Pushing ed/iliad-sorbonne/20250304 to national bibliotheca...
Requesting LID from bibliotheca.fr...
```

Each bibliotheca chooses its own consensus model. The `[consensus]` block in the
local configuration file describes how convergence decisions are made:

```toml
# Example: supermajority vote (the Sorbonne's chosen model)
[consensus]
model = "supermajority"
threshold = 0.70          # 70% approval required

# Alternative: a single designated authority (BDFL)
# [consensus]
# model = "bdfl"
# authority = "@schmidt"

# Alternative: unanimity — every participant must approve
# [consensus]
# model = "unanimity"
```

These are illustrative examples; bibliothecae are free to define the model that
fits their scholarly community.

---

## Tier 2: National Bibliothecae

**Who:** Academies of Letters/Sciences, National Libraries, equivalent scholarly bodies

**What they do:**
- **Receive** converged stemmata from Tier 1 institutions
- **Validate** that submissions meet national scholarly standards
- **Issue national LIDs** (e.g., `fr.2026.0042`)
- **Host** national editions for long-term preservation
- **Never create content directly** — only receive from Tier 1
- **Sync metadata** with the international bibliotheca

### Technical Capabilities
- **Acts are disabled** — no direct commits
- **LID issuance is enabled** — can create permanent identifiers
- **Sync with Tier 1** (institutional bibliothecae)
- **Sync with Tier 3** (international bibliotheca)
- Maintains provenance of all received works

### Configuration Example

```toml
# /etc/litodex/bibliotheca.fr.toml
[server]
name = "Bibliotheca Nationalis Franciae"
domain = "bibliotheca.fr"
tier = 2

[capabilities]
acts_enabled = false  # Cannot create content
lid_issuance = true    # Can issue national LIDs
sync_enabled = true    # Can sync with peers

[peers]
# Tier 1 institutions that feed into this national bibliotheca
tier1 = [
  "https://bibliotheca.sorbonne.fr",
  "https://bibliotheca.cnrs.fr",
  "https://bibliotheca.college-de-france.fr"
]

# Tier 3 peer (only one)
tier3 = "https://litodex.org"

[lid]
# National LID namespace
namespace = "fr"
pattern = "{namespace}.{year}.{sequential}"  # e.g., fr.2026.0001

[sync]
# Push metadata to Tier 3 immediately
push_to_tier3 = true
# Check for conflicts daily
conflict_check_interval = "24h"
```

### What a National Bibliotheca Stores

```bash
$ ls -la /var/lib/bibliotheca.fr/grc/
drwxr-xr-x  homer-iliad/
-rw-r--r--  provenance.toml

$ cat homer-iliad/provenance.toml
[work]
id = "grc/homer-iliad"
national_id = "fr.2026.0042"
global_lid = "grc/homer-iliad"  # Confirmed by Tier 3

[instances]
sorbonne-20250304 = {
  source = "bibliotheca.sorbonne.fr/grc/homer-iliad/ed/sorbonne/20250304",
  validated = "2026-03-05",
  validator = "Académie des Inscriptions et Belles-Lettres",
  validation_notes = "Meets French scholarly standards for critical editions"
}

cnrs-20250301 = {
  source = "bibliotheca.cnrs.fr/grc/homer-iliad/ed/cnrs/20250301",
  validated = "2026-03-02",
  validator = "Académie des Sciences",
  validation_notes = "Diplomatic transcription, apparatus complete"
}

[national_edition]
# The academy may choose to highlight certain versions
current = "sorbonne-20250304"
archive = ["cnrs-20250301"]
```

### A Note on Countries Without a Strong Tier 2

The three-tier model is designed for countries with strong national scholarly institutions. Countries without a meaningful Tier 2 authority — the United States is the most prominent example, where no single body plays the role of a national academy in this sense — can have their Tier 1 institutions sync directly with Tier 3. This is not a failure of the model. It is the federation adapting to local scholarly culture. The configuration is simple: a Tier 1 bibliotheca that lists Tier 3 as its peer instead of a Tier 2.

```toml
# Example: US institution syncing directly with Tier 3
[peers]
# No tier2 entry — sync directly with the international registry
tier3 = "https://litodex.org"
```

---

## Tier 3: International Bibliotheca (litodex.org)

**Who:** A lightweight coordinating body — the global registry

**What they do:**
- **Maintain the global LID registry** (which works exist, their canonical names)
- **Sync metadata** across all national bibliothecae
- **Detect identifier conflicts** when two nations use the same LID for different works
- **Never store content** — only metadata and pointers
- **Never exercise power** — conflicts are returned to the nations involved
- **Confirm global LIDs** after checking for conflicts

### Technical Capabilities
- **Acts are disabled** — no content, ever
- **LID issuance is enabled** — confirms global identifiers
- **Sync with all Tier 2** bibliothecae
- **Conflict detection** (automatic)
- **Conflict resolution** (human-mediated, never automatic)

### Configuration Example

```toml
# /etc/litodex/litodex.org.toml
[server]
name = "Litodex Internationalis"
domain = "litodex.org"
tier = 3

[capabilities]
acts_enabled = false     # Cannot create content
lid_issuance = true      # Can confirm global LIDs
sync_enabled = true      # Sync with all Tier 2

[peers]
# All national bibliothecae
tier2 = [
  "https://bibliotheca.fr",
  "https://bibliotheca.de",
  "https://bibliotheca.it",
  "https://bibliotheca.gr",
  "https://bibliotheca.br",
  "https://bibliotheca.in",
  "https://bibliotheca.cn",
  "https://bibliotheca.eg",
  "https://bibliotheca.il",
  # ... all others
]

[lid]
# Global namespace (no prefix)
namespace = "global"
pattern = "{lang}/{author}-{work}"  # e.g., grc/homer-iliad

[sync]
# Pull from all Tier 2 every hour
pull_interval = "1h"
# Immediately flag conflicts
conflict_detection = "immediate"
# Never resolve conflicts automatically
auto_resolve = false
```

### The Global Registry Data Model

The full registry data model — including `national_instances`, `archived_instances`, and copyright fields — is defined in the `meta.litodex.org` service. See [The Litodex Service Ecosystem](#the-litodex-service-ecosystem) for the canonical JSON schema.

---

## The LID Issuance Flow

```mermaid
sequenceDiagram
    participant Scholar
    participant Sorbonne as Sorbonne (Tier 1)
    participant FR as bibliotheca.fr (Tier 2)
    participant INT as litodex.org (Tier 3)
    
    Scholar->>Sorbonne: Create codex for new work
    Note over Sorbonne: Local identifier (temp)
    
    Scholar->>Sorbonne: Build consensus, create edition
    Sorbonne->>FR: Push for national registration
    
    FR->>FR: Validate work is new to France
    FR->>FR: Issue national LID: fr.2026.0042
    
    FR->>INT: Check if global LID exists
    INT->>INT: Query registry
    INT-->>FR: No conflict, grc/homer-iliad available
    
    FR->>INT: Register fr.2026.0042 as instance of grc/homer-iliad
    INT->>INT: Create/confirm global LID: grc/homer-iliad
    INT->>INT: Link: grc/homer-iliad ↔ fr.2026.0042
    
    INT-->>FR: Global LID confirmed
    FR-->>Sorbonne: Registration complete
    Sorbonne-->>Scholar: Your work is now: grc/homer-iliad
    Note over Scholar: Can now cite permanently
```

---

## Conflict Detection and Resolution

The international bibliotheca's **only power** is to detect conflicts and notify the parties involved.

### The Conflict Detector

The conflict-detection logic is implemented in `meta.litodex.org`. See [The Litodex Service Ecosystem](#the-litodex-service-ecosystem) for the canonical `GlobalRegistry` class.

### Real Conflict Example

```bash
# Two nations try to register the same LID simultaneously
$ litodex.org/logs/2026-03-04.log

10:32:15 [CONFLICT] Detected simultaneous reservation
  LID: "san/buddha-charita"
  
  Reservation 1: bibliotheca.in (India)
    Work: "Life of Buddha by Ashvaghosha"
    Evidence: "Critical edition based on Sanskrit manuscripts"
  
  Reservation 2: bibliotheca.cn (China)
    Work: "Buddhacarita (Chinese canon version)"
    Evidence: "Taisho Tripitaka edition with Chinese commentary"
  
  These appear to be different recensions of related but distinct texts.

10:32:16 [ACTION] Notified both parties
  To: bibliotheca.in, bibliotheca.cn
  Subject: LID conflict: san/buddha-charita
  
  "Both of you have reserved this LID for what appear to be
  different works. Please communicate and decide among yourselves:
  
  - One of you keeps the LID, the other chooses a different one
  - You agree to share the LID with clear attribution
  - You request hierarchical LIDs (e.g., san/buddha-charita/indian)
  
  litodex.org has no opinion. We await your consensus."

10:48:03 [RESOLUTION] Received from both parties
  "We have agreed:
   - bibliotheca.in will use san/buddha-charita for the Sanskrit version
   - bibliotheca.cn will use san/buddha-charita-chinese for the Chinese canon version
   - Both LIDs will cross-reference each other in metadata"
  
  Registry updated.
```

---

## Roles (Per Bibliotheca)

Each bibliotheca maintains its own roles independently. A scholar may be a curator at their university, a custos at the national level, and have no role internationally.

| Role | Latin | Responsibility | Level |
|------|-------|----------------|-------|
| **Curator** | *curator* | Maintains radix stemma (metadata) | Any |
| **Custos** | *custos* | Facilitates consensus for a public stemma | Any |

### Curator (at any level)

The curator maintains the **radix** — the root stemma containing only `meta.toml`.

```bash
# At institutional level
$ lit cur list --bibliotheca=bibliotheca.sorbonne.fr
Curatores for grc/homer-iliad:
  @smith (since 2026-01-15)
  @jones (since 2026-02-20)

# At national level (Tier 2, though acts disabled)
$ lit cur list --bibliotheca=bibliotheca.fr
Curatores for national metadata:
  @dupont (Académie des Inscriptions)
  @martin (Bibliothèque Nationale)
```

### Custos (at any level)

The **custos** serves the consensus within their bibliotheca.

```bash
$ lit cus list --bibliotheca=bibliotheca.uni-hd.de
Custodes for grc/homer-iliad:
  @schmidt → ed/iliad-heidelberg
  @weber → ms/venetus-a-diplomatic
```

A custos:
- Manages convergence according to the bibliotheca's own configured consensus model
- Facilitates discussion and, where applicable, voting
- **Verifies source integrity** before convergence (a Litodex-level requirement regardless of consensus model)
- Executes convergences only when the bibliotheca's configured rules are satisfied

#### The Custos Dashboard

```bash
$ lit custos dashboard --stemma=ed/iliad-oxford
Custos dashboard for ed/iliad-oxford

Current version: ed/iliad-oxford/20250401

Open proposals:
  prop/iliad-oxford-tyr (92% approve, sources: 3/3 verified) → ready to converge
  prop/iliad-oxford-wlm (63% approve, sources: 5/5 verified) → needs discussion
  prop/iliad-oxford-zab (41% approve, sources: 2/4 verified) → weak support, missing sources

Recent convergences:
  2025-04-01: converged prop/iliad-oxford-jqr (apparatus)
  2025-03-15: converged prop/iliad-oxford-tyr (line 102)
  2025-03-04: created ed/iliad-oxford from 3 proposals

Consensus model: supermajority (70% approve, configured locally), all sources must be verified
```

## Signing and Trust

Every act in Litodex must be signed with a GPG or SSH key. Signatures provide accountability, but their role differs between local and federated trust.

### The Trust Chain

```
Tier 2 (national bibliotheca)
  └── issues keys to custodes
        └── custos signs convergence acts
              └── Tier 2 verifies its own issued key
```

- **Individual act signatures** provide local accountability within a Tier 1 bibliotheca — they track who contributed what and deter tampering.
- **`prop/` branches are always squash-merged** into the target stemma when converged. This collapses the individual act signatures into a single convergence act.
- **Only the custos signs the convergence act.** This is the signature that matters for upstream trust. When a Tier 1 bibliotheca pushes to its national Tier 2, only this single signature needs to be verified.

### Key Issuance and Revocation

Tier 2 issues keys to custodes at Tier 1 institutions. A custos at the Sorbonne, for example, holds a key issued by `bibliotheca.fr` — not by the Sorbonne itself — that certifies them as a recognized custos in the French scholarly network.

```
1. Custos requests a key from their national bibliotheca (Tier 2)
2. Tier 2 verifies the scholar's role at their institution
3. Tier 2 issues a signed certificate binding the scholar's public key
   to their custos role
4. The custos uses this key to sign all convergence acts
5. When Tier 1 pushes to Tier 2, the national bibliotheca verifies
   a signature it issued itself — no cross-institutional trust needed
```

If a custos leaves or is otherwise revoked, the Tier 2 bibliotheca revokes the key. All future convergence acts from that custos will fail verification, protecting the integrity of the upstream record.

### Why Squash-Merging Makes This Work

Because `prop/` branches are always squash-merged, the entire history of individual acts — with their many different author signatures — is collapsed into one signed convergence act. Tier 2 does not need to trust every scholar who contributed to a proposal; it only needs to trust the custos who certified the result. This keeps the verification model simple and the trust surface small.

Individual act signatures remain in the local history at the Tier 1 bibliotheca for local accountability and auditing — they are simply not required for federation trust.

## Repository Structure (Same at All Levels)

Every codex follows this pattern:

```
{lang}/{author}-{work}
```

Example: `grc/homer-iliad`

### Stemma Hierarchy

| Prefix | Latin | Purpose | Protection |
|--------|-------|---------|------------|
| `radix` | *radix* | Root stemma with `meta.toml` and `sources.toml` | 🔒 Curators only |
| `ed/` | *editio* | Published editions (consensus-based) | 🔒 Custos-facilitated |
| `ms/` | *manuscriptum* | Historical manuscript transcriptions | 🔒 Custos-facilitated |
| `prop/` | *propositum* | Proposals for changes (must include sources) | ❌ Anyone, but must cite sources |
| `priv/` | *privatus* | Personal workspace | ❌ Owner only |
| `collab/` | *collaboratio* | Group projects | 🔒 Team |
| `rev/` | *recensio* | Review stemmata | ⚠️ Temporary |
| `arch/` | *archivum* | Archived stemmata | 🔒 Read-only |

### The Radix Stemma

Every codex has a `radix` stemma containing `meta.toml` and a system-managed `sources.toml` file:

```toml
# meta.toml
[work]
id = "grc/homer-iliad"
title = "Iliad"
author = "Homer"
language = "grc"
type = "poetry"

# Optional
period = "8th century BCE"
description = "Ancient Greek epic poem"
license = "public-domain"
```

The `sources.toml` file is a cumulative record of all sources from all converged proposals. It is **system-managed** — `lit` updates it automatically whenever acts with source metadata are converged into a stemma. Users cannot edit it directly.

```toml
# sources.toml (system-managed -- do not edit directly)
[[sources]]
act = "a1b2c3d"
stemma = "ed/iliad-oxford"
converged = "2026-03-04T10:30:00Z"
type = "digital"
url = "https://archive.org/details/homeriilias00home"
hash = "sha256:def456..."
conversion = "litogramma-v1"

[[sources]]
act = "e4f5g6h"
stemma = "ed/iliad-oxford"
converged = "2026-03-15T14:22:00Z"
type = "print"
citation = "Monro (1897). Homer: Iliad I-XII. Oxford. p. 23"
mediator = "@smith"

[[sources]]
act = "i7j8k9l"
stemma = "ed/iliad-oxford"
converged = "2026-03-15T14:22:00Z"
type = "manuscript"
identifier = "Venetus A"
catalog = "Marc. Gr. Z. 454"
folio = "47r"
mediator = "@jones"
```

This gives the radix a complete provenance record for the work, accumulating sources from every converged proposal across all stemmata.

The radix is:
- Created at initialization, never deleted
- Only editable by curators (for `meta.toml`; `sources.toml` is managed by `lit`)
- Automatically converged into all other stemmata when changed
- The source of truth for work identity and cumulative source provenance

```bash
$ lit sm show radix
Stemma: radix (PROTECTED)
Type: root stemma
Curators: @smith, @jones
Contains: meta.toml, sources.toml
Acts: 3 (last: a1b2c3d "Updated description")
Auto-converges to: all stemmata
```

## Public Stemmata: `ed/` and `ms/`

### Definition

| Stemma | Purpose | Example |
|--------|---------|---------|
| `ed/` | Published editions representing scholarly consensus | `ed/iliad-oxford` |
| `ms/` | Diplomatic transcriptions of historical manuscripts | `ms/venetus-a` |

Both follow the **same consensus-based workflow**. Neither exists until the community creates them through proposals.

### The Proposal System

Proposals use random 3-letter IDs to avoid implying priority or order:

```
prop/{target-stemma-name}-{random-id}
```

Examples:
- `prop/iliad-oxford-xkm`
- `prop/iliad-oxford-jqr`
- `prop/venetus-a-plm`

The random ID (consonant-vowel-consonant) ensures no proposal appears "first" or "more important."

## The Source Requirement

**Every act in a `prop/` stemma must be traceable to a source.** This creates an auditable chain of evidence.

### Source Types

```toml
# Digital sources (automatically verifiable)
[source.type.digital]
url = "https://..."           # Source URL
hash = "sha256:abc123..."     # Content hash for verification
conversion_pipeline = "litogramma"  # The markup conversion used

# Print sources (require human mediation)
[source.type.print]
citation = "West, M.L. (1998). Homerus: Ilias. Vol. I. Stuttgart: Teubner. p. 47"
mediator = "@scholar"         # Who verified this source
verification_date = "2026-03-04"
note = "Personal examination of copy in Bodleian Library"

# Manuscript sources (physical or digital)
[source.type.manuscript]
identifier = "Venetus A"       # Common name
catalog = "Marc. Gr. Z. 454"   # Catalog number
library = "Biblioteca Nazionale Marciana, Venice"
folio = "47r"
line = "12"
image_url = "https://..."      # If digitized
```

---

## The Consensus Workflow (Tier 1)

### Phase 1: No Public Stemma Exists

Initially, only `radix`, manuscripts (`ms/`), and personal stemmata exist:

```bash
$ lit sm list
grc/homer-iliad:
  radix
  ms/venetus-a
  ms/townley
  priv/smith-notes
  priv/jones-collation
  prop/iliad-oxford-xkm   (proposed Oxford edition)
  prop/iliad-oxford-jqr   (another proposal)
  prop/iliad-oxford-plm   (yet another)
```

### Phase 2: Create Proposal with Sources

Scholars create proposal stemmata with embedded source metadata:

```bash
# Create a proposal with source metadata
$ lit prop create iliad-oxford-xkm \
  --target=ed/iliad-oxford \
  --source-type=digital \
  --source-url="https://archive.org/details/homeriilias00home" \
  --source-hash="sha256:def456..." \
  --pipeline="litogramma-v1" \
  --message="Base text from Archive.org scan, converted to Litogramma"

# Source metadata is stored as structured trailers in the commit message,
# for example:
#
#   Base text from Archive.org scan, converted to Litogramma
#
#   Source-type: digital
#   Source-url: https://archive.org/details/homeriilias00home
#   Source-hash: sha256:def456...
#   Source-pipeline: litogramma-v1
#   Source-verified-by: @smith
```

### Phase 3: Building on a Proposal

Subsequent acts must also cite sources:

```bash
# Make a change with source attribution
$ lit act -m "Corrected accent in line 102" \
  --source-type=print \
  --source-citation="Monro (1897). Homer: Iliad I-XII. Oxford. p. 23" \
  --source-mediator="@smith" \
  --source-note="Monro discusses this crux"
```

Each act can track multiple sources using structured commit trailers. When an act references more than one source, `lit` records each source as a numbered set of trailers:

```
Corrected accent in line 102

Source-type: print
Source-citation: Monro (1897). Homer: Iliad I-XII. Oxford. p. 23
Source-mediator: @smith
Source-note: Monro discusses this crux

Source-type: manuscript
Source-identifier: Venetus A
Source-note: Confirmed reading on folio 47r
Source-mediator: @jones
```

The diff and the commit message together are sufficient to judge a proposal — the trailers provide the source provenance without requiring any separate metadata file.

### Viewing Proposal Sources

```bash
# Show all sources used in a proposal
$ lit prop sources iliad-oxford-xkm
Proposal: iliad-oxford-xkm
Target: ed/iliad-oxford

Initial source:
  📄 Archive.org scan (digital)
  URL: https://archive.org/details/homeriilias00home
  Hash: sha256:def456... ✓ verified
  Conversion: litogramma-v1

Act a1b2c3d: "Corrected accent in line 102"
  📚 Monro (1897), p. 23 (print)
  Mediator: @smith
  Note: "Monro discusses this reading"

Act e4f5g6h: "Added apparatus note"
  📜 Venetus A, fol. 47r (manuscript)
  Mediator: @jones
  Image: https://.../venetus-a/47r.jpg
```

### The Radix Auto-Convergence

When curators update the radix:

```bash
$ lit sm checkout radix
$ vim meta.toml
$ lit act -m "Updated license to CC-BY"

# Automatically converges to ALL stemmata
$ lit act show a1b2c3d
Actum: a1b2c3d
Stemma: radix
Message: "Updated license to CC-BY"

Auto-converged to:
  ✓ ed/iliad-oxford (convergence act e4f5g6h)
  ✓ ed/iliad-teubner (convergence act i7j8k9l)
  ✓ ms/venetus-a (convergence act m0n1o2p)
  ✓ priv/smith-experimental (convergence act q3r4s5t)
  ✓ ...
```

Metadata flows to all traditions automatically.

### Phase 4: Source Verification

Digital sources can be automatically verified:

```bash
# Verify a digital source
$ lit source verify https://archive.org/details/homeriilias00home \
  --hash="sha256:def456..." \
  --pipeline="litogramma-v1"

Verifying source...
Downloading... done
Computing hash... matches (def456...)
Converting to Litogramma... done
Validation: 0 errors, 2 warnings
  Warning: Line 47 missing verse number marker
  Warning: Line 103 has ambiguous line break

Source verified with warnings.
```

### Phase 5: Community Discussion and Voting

Scholars discuss, provide evidence, and vote:

```bash
$ lit prop vote prop/iliad-oxford-xkm --approve --reason="Matches manuscript evidence"
$ lit prop comment prop/iliad-oxford-xkm -m "See attached image of Venetus A folio 47r"

$ lit prop vote prop/iliad-oxford-jqr --reject --reason="Needs stronger evidence"
```

### Phase 6: Custos Verifies and Converges

When consensus is reached, the custos must verify all sources before converging:

```bash
$ lit consensus check prop/iliad-oxford-xkm --verify-sources
Checking consensus... 78% approve (local threshold met: supermajority ≥70%)
Checking sources...

Initial source: ✓ verified (hash matches)
Act a1b2c3d: ✓ source verified (print citation accepted)
Act e4f5g6h: ⚠️ manuscript image URL 404
  → Requires verification from mediator

Consensus met but source verification incomplete.
Cannot converge until all sources are verified.
```

After verification:

```bash
$ lit converge prop/iliad-oxford-xkm --into=ed/iliad-oxford
Converging prop/iliad-oxford-xkm into ed/iliad-oxford
Consensus confirmed: 78% approve (exceeds local threshold: supermajority ≥70%)
All sources verified: 12 digital, 8 print, 3 manuscript
Creating ed/iliad-oxford...
Convergence complete.

# A versio is automatically created with date suffix
$ lit ver list
ed/iliad-oxford/20250304   (first edition, includes xkm changes)
```

### Phase 7: Subsequent Corrections

Later, another scholar proposes a correction with proper sourcing:

```bash
# Create from an existing versio
$ lit prop create iliad-oxford-tyr \
  --from=ed/iliad-oxford/20250304 \
  --target=ed/iliad-oxford \
  --message="Correct line 102 based on manuscript evidence"

$ vim iliad.txt  # fix line 102
$ lit act -m "Corrected accent in line 102" \
  --source-type=manuscript \
  --source-identifier="Venetus A" \
  --source-folio="47r" \
  --source-library="Marciana" \
  --source-mediator="@smith"

# Discussion, voting, verification, convergence...
$ lit converge prop/iliad-oxford-tyr --into=ed/iliad-oxford
Converged. New versio: ed/iliad-oxford/20250315
```

### The Versio Timeline

```bash
$ lit ver list --stemma=ed/iliad-oxford
ed/iliad-oxford/20250304   (initial consensus edition)
ed/iliad-oxford/20250315   (correction to line 102)
ed/iliad-oxford/20250401   (added apparatus from jqr proposal)
ed/iliad-oxford/20250420   (further corrections)
```

Each versio is a frozen snapshot of community consensus at that point in time, with complete provenance tracking back to original sources.

---

## The `lit` CLI (Extended for Federation)

### Bibliotheca Management

```bash
# Configure your bibliotheca
$ lit config set bibliotheca https://bibliotheca.sorbonne.fr
$ lit config set national https://bibliotheca.fr

# Show federation status
$ lit federation status
Your bibliotheca: bibliotheca.sorbonne.fr (Tier 1)
National peer: bibliotheca.fr (Tier 2) - connected
International peer: litodex.org (Tier 3) - connected via national

# List all known bibliothecae
$ lit federation list
Tier 1 (operational):
  - bibliotheca.sorbonne.fr
  - bibliotheca.cnrs.fr
  - bibliotheca.uni-hd.de
  
Tier 2 (national):
  - bibliotheca.fr (peer)
  - bibliotheca.de
  - bibliotheca.it
  
Tier 3 (international):
  - litodex.org (connected)
```

### Pushing to National Level

```bash
# Push an edition for national registration
$ lit push national --stemma=ed/iliad-oxford --versio=20250304
Pushing to bibliotheca.fr...
Validation in progress...
✓ Meets French scholarly standards
✓ Sources verified (12/12)
✓ Consensus documented

Issued national LID: fr.2026.0042
Checking global registry...
Global LID confirmed: grc/homer-iliad

Your edition is now permanently citable as:
  https://bibliotheca.fr/grc/homer-iliad/ed/oxford/20250304
  Global LID: grc/homer-iliad
```

### Resolving LIDs

```bash
# Resolve a global LID
$ lit resolve grc/homer-iliad
Found 3 national instances:

1. France (fr.2026.0042)
   URL: https://bibliotheca.fr/grc/homer-iliad
   Editions: oxford-20250304, cnrs-20250301
   
2. Germany (de.2026.0087)
   URL: https://bibliotheca.de/grc/homer-iliad
   Editions: heidelberg-20250215, leipzig-20250120
   
3. Greece (gr.2026.0012)
   URL: https://bibliotheca.gr/grc/homer-iliad
   Editions: athens-academy-20250301

# Resolve a national LID  
$ lit resolve fr.2026.0042
National LID: fr.2026.0042
Issued by: bibliotheca.fr
Global LID: grc/homer-iliad
Editions:
  - https://bibliotheca.sorbonne.fr/grc/homer-iliad/ed/sorbonne/20250304
  - https://bibliotheca.cnrs.fr/grc/homer-iliad/ed/cnrs/20250301
```

---

## Integration with Litogram

Litodex provides the verified texts; Litogram provides the practice:

```typescript
// litogram.org backend
async function getText(lid: string) {
    // Resolve LID through federation
    const instances = await resolveLID(lid);
    
    // Prefer national instance or let user choose
    const selected = await selectInstance(instances);
    
    const { content, metadata, sources } = await fetch(selected.url);
    
    return {
        typing: strip_markup(content),      // 🌕 Full text
        memorizing: first_letters(content), // 🌗 First letters only
        reciting: blank_page(),              // 🌑 Blank page
        metadata,
        sources: formatCitation(sources),
        citation: `${lid} (via ${selected.nation})`
    };
}
```

---

## Why This Architecture?

### For Scholars
- Work at your institution with local authentication
- National validation ensures quality
- Global discovery through LIDs
- Complete provenance tracking

### For Institutions
- Full control over your scholarship
- No vendor lock-in — it's open source
- Brand recognition (your own bibliotheca)
- Teaching sandboxes without global pollution

### For Nations
- Cultural sovereignty respected
- Set your own validation standards
- Preserve national scholarly traditions
- Control what enters your bibliotheca

### For Humanity
- No single point of failure or control
- Multiple perspectives preserved
- Resilient network of scholarship
- Permanent, citable identifiers for all texts

## The Litodex Service Ecosystem

Litodex is not a single website or service — it's a family of distinct services, each with a clear and limited responsibility. This separation ensures the system remains decentralized, copyright-respecting, and resilient.

### The Three International Services

| Service | Domain | Purpose | Content | Acts | LIDs |
|---------|--------|---------|---------|------|------|
| **Documentation** | `www.litodex.org` | Explain the system, host specs, link to resources | No | No | No |
| **Metadata Registry** | `meta.litodex.org` | Global LID registry, discovery, conflict detection | No | No | Yes |
| **Content Archive** | `arch.litodex.org` | Optional mirror of public domain texts | Yes (PD only) | No | No |

#### 1. www.litodex.org — The Documentation Hub

The human-facing presence. Pure information, no data.

```html
<!-- www.litodex.org/index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Litodex — Version Control for Humanity's Texts</title>
</head>
<body>
    <h1>Litodex: A Federation of Scholarly Texts</h1>
    
    <section>
        <h2>What is Litodex?</h2>
        <p>A protocol and federation for version-controlled, 
        verified, and collaborative management of literary and sacred texts.</p>
    </section>
    
    <section>
        <h2>Quick Links</h2>
        <ul>
            <li><a href="https://meta.litodex.org">Global Metadata Registry</a></li>
            <li><a href="https://arch.litodex.org">Public Domain Text Archive</a></li>
            <li><a href="/spec/v1">Protocol Specification</a></li>
            <li><a href="/download">Download Litodex Server</a></li>
        </ul>
    </section>
    
    <section>
        <h2>Participating National Bibliothecae</h2>
        <ul>
            <li><a href="https://bibliotheca.fr">France</a></li>
            <li><a href="https://bibliotheca.de">Germany</a></li>
            <li><a href="https://bibliotheca.gr">Greece</a></li>
            <!-- dynamically populated from meta.litodex.org -->
        </ul>
    </section>
</body>
</html>
```

**What it does:**
- Explains the Litodex philosophy and architecture
- Hosts the official protocol specification
- Provides download links for the Litodex server software
- Links to all participating national bibliothecae
- Directs users to the metadata registry for discovery

**What it NEVER does:**
- Store or serve any text content
- Issue or resolve LIDs
- Get involved in disputes

---

#### 2. meta.litodex.org — The Global Metadata Registry

The thin sync layer — a directory of what exists and where to find it.

```bash
# What meta.litodex.org stores
$ curl https://meta.litodex.org/v1/registry/grc/homer-iliad

{
  "global_lid": "grc/homer-iliad",
  "canonical_name": "Homer, Iliad",
  "language": "grc",
  "registered": "2026-01-15T10:00:00Z",
  "last_sync": "2026-03-04T15:30:00Z",
  
  "national_instances": [
    {
      "nation": "fr",
      "bibliotheca": "https://bibliotheca.fr",
      "national_lid": "fr.2026.0042",
      "content_url": "https://bibliotheca.fr/grc/homer-iliad",
      "status": "active",
      "copyright": "public-domain"  # Metadata includes copyright status
    },
    {
      "nation": "de",
      "bibliotheca": "https://bibliotheca.de",
      "national_lid": "de.2026.0087",
      "content_url": "https://bibliotheca.de/grc/homer-iliad",
      "status": "active",
      "copyright": "cc-by-nc-4.0"
    },
    {
      "nation": "gr",
      "bibliotheca": "https://bibliotheca.gr",
      "national_lid": "gr.2026.0012",
      "content_url": "https://bibliotheca.gr/grc/homer-iliad",
      "status": "active", 
      "copyright": "in-copyright"
    }
  ],
  
  "archived_instances": [  # Optional pointers to content archive
    {
      "archive": "https://arch.litodex.org",
      "url": "https://arch.litodex.org/grc/homer-iliad/fr.2026.0042",
      "since": "2026-03-05"
    }
  ]
}
```

**What it does:**
- Maintains the global registry of LIDs
- Syncs metadata from all national bibliothecae
- Detects identifier conflicts (and only detects — never resolves)
- Provides a discovery API for users and tools
- Tracks copyright status of each national instance

**What it NEVER does:**
- Store any actual text content
- Resolve conflicts between nations (only notifies)
- Prefer one national instance over another
- Make editorial decisions

**The entire codebase (simplified):**

```python
# meta.litodex.org's core logic
class GlobalRegistry:
    def __init__(self):
        self.lids = {}  # global LID → list of national instances
    
    def check_conflict(self, global_lid, requesting_nation):
        """Check if a global LID is available"""
        if global_lid in self.lids:
            return {
                "status": "exists",
                "instances": self.lids[global_lid],
                "message": "This LID is already registered. See existing instances."
            }
        return {"status": "available"}
    
    def register(self, global_lid, national_instance):
        """Register a national instance under a global LID"""
        if global_lid not in self.lids:
            self.lids[global_lid] = []
        self.lids[global_lid].append(national_instance)
        self.broadcast_update(global_lid)
    
    def resolve(self, global_lid):
        """Return all known instances of a work"""
        return self.lids.get(global_lid, [])
```

---

#### 3. arch.litodex.org — Optional Public Domain Archive

A convenience service — fast global access to public domain texts, completely optional for national bibliothecae.

```bash
# How a national bibliotheca opts in
$ lit archive push fr.2026.0042 --to=arch.litodex.org
Checking copyright status...
✓ fr.2026.0042 is public domain
Pushing content...
Content archived at: https://arch.litodex.org/grc/homer-iliad/fr.2026.0042
Metadata updated at meta.litodex.org

# How a user accesses it
$ lit get grc/homer-iliad --prefer=archive
Found via meta.litodex.org:
  National instances: FR, DE, GR
  Archived copy: arch.litodex.org (public domain)
  
Fetching from archive for speed...
```

**What it does:**
- Mirrors public domain texts from participating national bibliothecae
- Provides fast global access to these texts
- Acts as a secondary preservation layer
- Respects copyright boundaries completely

**What it NEVER does:**
- Store any in-copyright or restricted-license texts
- Replace national bibliothecae as the authoritative source
- Accept direct uploads from individuals (only from national bibliothecae)
- Make claims about textual accuracy (delegates to national sources)

**National bibliotheca configuration:**

```toml
# /etc/litodex/bibliotheca.fr.toml
[archive]
# Optional: sync public domain works to central archive
enabled = true
archive_url = "https://arch.litodex.org"
sync_public_domain_only = true
sync_frequency = "24h"

[copyright]
# France's copyright determination
public_domain_years = "author_death + 70"
# Complex cases handled but never store without certainty
```

---

### How They Work Together

```mermaid
graph TB
    subgraph "User Facing"
        WEB[www.litodex.org<br/>Documentation]
        USER[Scholars & Students]
    end
    
    subgraph "Global Services"
        META[meta.litodex.org<br/>Metadata Registry]
        ARCH[arch.litodex.org<br/>Public Domain Archive]
    end
    
    subgraph "National Bibliothecae"
        FR[bibliotheca.fr]
        DE[bibliotheca.de]
        GR[bibliotheca.gr]
    end
    
    USER --> WEB
    USER --> META
    USER --> ARCH
    USER --> FR
    
    FR -->|metadata| META
    DE -->|metadata| META
    GR -->|metadata| META
    
    FR -.->|public domain| ARCH
    DE -.->|public domain| ARCH
    
    style WEB fill:#e1f5fe,stroke:#01579b
    style META fill:#f5f5f5,stroke:#333,stroke-width:3px
    style ARCH fill:#fff3e0,stroke:#e65100,stroke-dasharray: 5 5
```

### A User's Journey Through All Three

```bash
# 1. A student hears about Litodex and visits the documentation
$ open https://www.litodex.org
"Learn about the system, find the spec, get the software"

# 2. They want to find the Iliad and check the metadata registry
$ curl https://meta.litodex.org/v1/resolve/grc/homer-iliad
{
  "global_lid": "grc/homer-iliad",
  "instances": [
    {
      "nation": "fr",
      "url": "https://bibliotheca.fr/grc/homer-iliad",
      "copyright": "public-domain"
    },
    {
      "nation": "de", 
      "url": "https://bibliotheca.de/grc/homer-iliad",
      "copyright": "cc-by-nc-4.0"
    }
  ],
  "archived": "https://arch.litodex.org/grc/homer-iliad/fr.2026.0042"
}

# 3. They want the fastest access to a public domain version
$ lit get grc/homer-iliad --from=archive
Fetching from arch.litodex.org...
Download complete. (Source: bibliotheca.fr, public domain)

# 4. For scholarly work, they might go directly to the source
$ open https://bibliotheca.fr/grc/homer-iliad
"Welcome to the French National Bibliotheca's edition — 
 complete apparatus, source attribution, and scholarly context"
```

### Why This Separation Matters

| Concern | Handled By | Why It's Right |
|---------|------------|----------------|
| **Discovery** | `meta.litodex.org` | Central registry makes finding texts easy |
| **Content Authority** | National bibliothecae | Nations set standards, bear responsibility |
| **Copyright** | National bibliothecae + `arch.litodex.org` | Only public domain flows to archive |
| **Performance** | `arch.litodex.org` | Fast global access for public domain |
| **Documentation** | `www.litodex.org` | Clear separation from operational services |
| **Preservation** | National + optional archive | Multiple layers, no single point of failure |

### The Golden Rule

**No single service does too much.** Each has one job, does it well, and stays out of the others' way. This is how a global federation remains resilient, trustworthy, and scalable.

- `www.litodex.org` **informs** but never stores
- `meta.litodex.org` **points** but never hosts
- `arch.litodex.org` **mirrors** but only what's public domain
- National bibliothecae **author** but never control globally
- Institutions **create** but never issue permanent IDs

This is the architecture of a system designed to last centuries.

---

## Identifiers: LITID, ACTID, and CID

Litodex uses three distinct types of identifiers, each serving a specific purpose. They are designed to be complementary and impossible to confuse.

| Identifier | What It Identifies | Format | Example | Use Case |
|------------|-------------------|--------|---------|----------|
| **LITID** | Content (canonical text) | `sha256:...` | `sha256:8f2a3c...` | Integrity verification, deduplication |
| **ACTID** | Actum (commit) | Git SHA | `a1b2c3d7e8f9...` | Technical reference, Git operations |
| **CID** (Citation ID) | Human-readable path | `{lang}/{edition}/{versio}` | `grc/homer-iliad-oxford-1920/ed/20250304` | Scholarly citation, discovery |

### The Golden Rules

- **LITID proves the text hasn't changed.**
- **ACTID tracks the technical history.**
- **CID is what you put in your bibliography.**

All three are permanent. All three are mathematically verifiable. None can be forged without detection.

---

### LITID — Content Integrity

A LITID is simply the SHA256 hash of a file in its canonical Litogramma form. It answers only one question: **"Is this exactly the text that was published?"**

```bash
# Compute LITID of any file
$ lit litid iliad.txt
sha256:8f2a3c7d9e1f5a6b2c3d4e5f6a7b8c9d0e1f2a3b

# Verify a file against a known LITID
$ lit verify iliad.txt --litid=sha256:8f2a3c...
✓ SHA256 matches - content is authentic

# This works offline, without network, without Git
# Just math.
```

**Properties of LITID:**
- Pure content hash — no metadata, no context
- Same content always produces the same LITID anywhere in the world
- Verification requires nothing but the file itself
- Ideal for deduplication, caching, and integrity checks

---

### ACTID — Technical Identifier

An ACTID is the Git commit hash of an **actum** — a specific snapshot of a stemma. It answers: **"What exact state of the repository contains this change?"**

```bash
# View an actum by its ACTID
$ lit show a1b2c3d7e8f9g0h1i2j3k4l5m6n7o8p9q0r1s2t3

ACTID: a1b2c3d7e8f9g0h1i2j3k4l5m6n7o8p9q0r1s2t3
Date: 2026-03-04
Author: @smith
Message: "Corrected line 102 based on Venetus A"

Files:
  iliad.txt
    LITID: sha256:8f2a3c7d9e1f5a6b2c3d4e5f6a7b8c9d0e1f2a3b
  
  sources.toml
    Content: [[sources]] type = "manuscript"...

Parent: e4f5g6h7i8j9k0l1m2n3o4p5q6r7s8t9u0v1w2x
```

**Properties of ACTID:**
- Git commit hash — universally unique
- Identifies the exact state of the entire stemma
- Used for technical operations and cross-referencing
- Automatically generated by the version control system

---

### CID — Scholarly Citation

A CID is a human-readable path that identifies a specific versio (snapshot) of an edition. It answers: **"What should I cite in my paper so others can find exactly what I used?"**

```bash
# Resolve a CID to its technical identifiers
$ lit resolve grc/homer-iliad-oxford-1920/ed/20250304

CID: grc/homer-iliad-oxford-1920/ed/20250304
ACTID: a1b2c3d7e8f9g0h1i2j3k4l5m6n7o8p9q0r1s2t3
LITID: sha256:8f2a3c7d9e1f5a6b2c3d4e5f6a7b8c9d0e1f2a3b

Provenance:
  Bibliotheca: https://bibliotheca.fr
  Validated: 2026-03-04
  Sources: Venetus A, Townley manuscript
```

**Properties of CID:**
- Human-readable and memorable
- Follows consistent pattern: `{lang}/{edition-name}/{versio}`
- What you put in footnotes and bibliographies
- Resolves to ACTID and LITID through the metadata registry

---

### How They Work Together

```mermaid
graph TB
    subgraph "Human Layer"
        CID[CID: grc/homer-iliad-oxford-1920/ed/20250304]
    end
    
    subgraph "Technical Layer"
        CID -->|resolves via meta.litodex.org| ACTID[ACTID: a1b2c3d...]
        ACTID -->|contains at bibliotheca.fr| CONTENT[Content File]
        CONTENT -->|has hash| LITID[LITID: sha256:8f2a3c...]
    end
    
    subgraph "Verification"
        LITID -->|validates| FILE[Your downloaded copy]
    end
    
    style CID fill:#e1f5fe,stroke:#01579b
    style ACTID fill:#fff3e0,stroke:#e65100
    style LITID fill:#f5f5f5,stroke:#333
```

---

### In Scholarly Publication

#### Full Citation

```bibtex
@book{homer-iliad-oxford-1920,
  title = {Homeri Ilias (Oxford Classical Texts)},
  editor = {Monro, D.B. and Allen, T.W.},
  year = {1920},
  version = {20250304},
  
  # Litodex identifiers
  cid = {grc/homer-iliad-oxford-1920/ed/20250304},
  actid = {a1b2c3d7e8f9g0h1i2j3k4l5m6n7o8p9q0r1s2t3},
  litid = {sha256:8f2a3c7d9e1f5a6b2c3d4e5f6a7b8c9d0e1f2a3b},
  
  repository = {https://bibliotheca.fr}
}
```

#### Print Reference

```text
In a printed book:

The text follows the Oxford edition of 1920 as preserved in the 
Litodex federation (CID: grc/homer-iliad-oxford-1920/ed/20250304). 
The specific version used has ACTID a1b2c3d7e8f9... and content 
integrity verified by LITID sha256:8f2a3c7d9e1f...
```

#### Classroom Use

```text
Professor to students:

"For Thursday, read Iliad Book 1 using the Oxford edition.
CID: grc/homer-iliad-oxford-1920/ed/20250304

You can verify your copy matches by checking the LITID:
sha256:8f2a3c7d9e1f5a6b2c3d4e5f6a7b8c9d0e1f2a3b"
```

---

### When to Use Each

| Scenario | Use CID | Use ACTID | Use LITID |
|----------|---------|-----------|-----------|
| Citing in a paper | ✅ Primary | ⚠️ Supplementary | ⚠️ Supplementary |
| Finding an edition | ✅ Essential | ❌ Not needed | ❌ Not needed |
| Technical debugging | ❌ Not enough | ✅ Required | ❌ Not needed |
| Verifying a download | ❌ Not needed | ❌ Not needed | ✅ Required |
| Checking for duplicates | ❌ Not needed | ❌ Not needed | ✅ Perfect |
| Git operations | ❌ Not enough | ✅ Required | ❌ Not needed |
| Offline validation | ❌ Needs network | ❌ Needs network | ✅ Works anywhere |

---

### Mnemonic

> **C**ID = **C**ite it in your paper  
> **ACT**ID = The **ACT** of committing  
> **LIT**ID = The **LIT**eral content

---

### Resolution Chain

```bash
# Start with a CID (what you have in your paper)
$ lit resolve grc/homer-iliad-oxford-1920/ed/20250304
→ ACTID: a1b2c3d7e8f9g0h1i2j3k4l5m6n7o8p9q0r1s2t3
→ LITID: sha256:8f2a3c7d9e1f5a6b2c3d4e5f6a7b8c9d0e1f2a3b

# Fetch the content using the ACTID
$ lit fetch a1b2c3d7e8f9g0h1i2j3k4l5m6n7o8p9q0r1s2t3
Downloaded iliad.txt

# Verify using the LITID
$ lit verify iliad.txt --litid=sha256:8f2a3c...
✓ Content is authentic
```

The three identifiers work together seamlessly, each serving its purpose without overreach. This is the architecture of a system built to last.
---

## License

Litodex core is open source under the MIT License. Content licenses are determined by contributors at each bibliotheca, with source attribution preserved forever.

---

**One protocol. Sovereign nodes. Infinite texts. Every change traceable to its source.**

**[Get Started](#) | [Federation Protocol Spec](#) | [Run a Bibliotheca](#) | [Community](#)**
