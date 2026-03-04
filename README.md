# Litodex — Version Control for Humanity's Texts

Litodex is a platform for version-controlled, verified, and collaborative management of literary and sacred texts. It provides permanent identifiers, scholarly workflows, and a foundation for applications like the Litogram typing practice app.

## Core Philosophy

- **One work = one codex** — not per edition, not per user
- **Stemmata = traditions** — multiple authoritative versions coexist as branches
- **No single master** — scholarship has no single source of truth
- **Manuscripts are first-class** — `ms/` stemmata alongside editions
- **Sources are sacred** — every change must be traceable to a verifiable source
- **Consensus-driven** — public editions emerge from community agreement, not maintainer fiat
- **Permanent identifiers** — every snapshot gets a citable LID
- **Lightweight markup** — Litogramma annotations make parsing trivial

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

## Roles

| Role | Latin | Responsibility | Alias |
|------|-------|----------------|-------|
| **Curator** | *curator* | Maintains radix stemma (metadata) | `cur` |
| **Custos** | *custos* | Facilitates consensus for a public stemma | `cus` |

### Curator

The curator maintains the **radix** — the root stemma containing only `meta.toml`. This role is about preserving the work's identity, not controlling content.

```bash
$ lit cur list
Curatores for grc/homer-iliad:
  @smith (since 2026-01-15)
  @jones (since 2026-02-20)

$ lit cur add @patel
Added @patel as curator. They can now maintain radix.
```

### Custos

The **custos** (plural: **custodes**) does not decide — they **serve the consensus**. Their role is to facilitate discussion, monitor proposals, verify sources, and execute convergences when the community reaches agreement.

```bash
$ lit cus list
Custodes for grc/homer-iliad:
  @oxford_editor   → ed/iliad-oxford
  @teubner_editor  → ed/iliad-teubner
  @manuscript_scholar → ms/venetus-a

$ lit cus add @cambridge_editor --stemma=ed/iliad-cambridge
Added @cambridge_editor as custos of ed/iliad-cambridge.
```

A custos:
- Does not have unilateral convergence authority
- Cannot override community consensus
- Facilitates discussion and voting
- **Verifies source integrity** before convergence
- Executes convergences only when consensus thresholds are met

## Repository Structure

Every codex follows this pattern:

```
{lang}/{author}-{work}
```

Example: `grc/homer-iliad`

### Stemma Hierarchy

| Prefix | Latin | Purpose | Protection |
|--------|-------|---------|------------|
| `radix` | *radix* | Root stemma with `meta.toml` | 🔒 Curators only |
| `ed/` | *editio* | Published editions (consensus-based) | 🔒 Custos-facilitated |
| `ms/` | *manuscriptum* | Historical manuscript transcriptions | 🔒 Custos-facilitated |
| `prop/` | *propositum* | Proposals for changes (must include sources) | ❌ Anyone, but must cite sources |
| `priv/` | *privatus* | Personal workspace | ❌ Owner only |
| `collab/` | *collaboratio* | Group projects | 🔒 Team |
| `rev/` | *recensio* | Review stemmata | ⚠️ Temporary |
| `arch/` | *archivum* | Archived stemmata | 🔒 Read-only |

### The Radix Stemma

Every codex has a `radix` stemma containing a single `meta.toml` file:

```toml
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

The radix is:
- Created at initialization, never deleted
- Only editable by curators
- Automatically converged into all other stemmata when changed
- The source of truth for work identity

```bash
$ lit sm show radix
Stemma: radix (PROTECTED)
Type: root stemma
Curators: @smith, @jones
Contains: meta.toml only
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

**Every act in a `prop/` stemma must be traceable to a source.** This creates an auditable chain of evidence from manuscript/image to final text. Sources are first-class citizens in the proposal system.

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

## The Consensus Workflow

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

### Phase 2: Proposals Are Created with Sources

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

# This creates a special annotated act that records source metadata
# and branches from that act to create the proposal stemma
```

Each proposal contains a `proposal.toml` at its root:

```toml
[proposal]
id = "iliad-oxford-xkm"
target = "ed/iliad-oxford"
created = "2026-03-04T10:30:00Z"
creator = "@smith"

[proposal.initial_source]
type = "digital"
url = "https://archive.org/details/homeriilias00home"
hash = "sha256:def456..."
conversion = "litogramma-v1"
verification_status = "verified"
verified_by = "@smith"

[proposal.goal]
description = "Create a new Oxford edition based on public domain sources"
scope = "Full text of Iliad with minimal apparatus"
bases = ["ms/venetus-a", "ms/townley"]
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

Each act can track multiple sources:

```toml
# In the proposal metadata, acts track their sources
[[proposal.acts]]
hash = "abc123..."
message = "Corrected accent in line 102"
sources = [
    { type = "print", citation = "Monro (1897). Homer: Iliad I-XII. Oxford. p. 23", 
      mediator = "@smith", note = "Monro discusses this crux" },
    { type = "manuscript", identifier = "Venetus A", 
      note = "Confirmed reading on folio 47r" }
]
rationale = "Manuscript evidence supports Monro's correction"
```

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
Checking consensus... 78% approve (threshold met)
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
Consensus confirmed: 78% approve (exceeds 70% threshold)
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

## The Custos Dashboard

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

Consensus threshold: 70% approve, all sources must be verified
```

## Source Registry

Frequently used sources can be registered for reuse:

```bash
# Register a source
$ lit source register \
  --type=print \
  --citation="West, M.L. (1998). Homerus: Ilias. Vol. I." \
  --isbn="9783598714105" \
  --notes="Standard critical edition"

Registered as source: src/west-iliad-1998

# Use registered source in proposal
$ lit prop create iliad-oxford-xkm \
  --source=src/west-iliad-1998 \
  --pages="47-49"
```

## Viewing Proposal Sources

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

## The Radix Auto-Convergence

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

## Permanent Identifiers (LIDs)

Every versio gets a permanent, citable Litodex Identifier:

```
{lang}/{author}/{work}/{stemma}/{date}
```

Example: `grc/homer/iliad/ed/oxford-1920/20250304`

Resolution:
```
https://lid.litodex.org/grc/homer/iliad/ed/oxford-1920/20250304
```

LIDs are stored as Git tags in `refs/tags/lid/` and are immutable.

## The `lit` CLI

### Codex Operations

```bash
# Initialize a new codex
$ lit codex init grc/homer-iliad --author="Homer" --title="Iliad"
Created codex grc/homer-iliad
  radix stemma initialized with meta.toml

# List all codices
$ lit codex list

# Show codex info
$ lit codex show
```

### Daily Work

```bash
# List stemmata
$ lit sm list
stemmata in grc/homer-iliad:
  radix
  ms/venetus-a
  priv/smith-experimental
  prop/iliad-oxford-xkm

# Create private stemma
$ lit sm create priv/smith-experimental --from=ms/venetus-a

# Switch stemma
$ lit sm checkout priv/smith-experimental

# Check status
$ lit st
Stemma: priv/smith-experimental
Status: 1 unstaged change

# Commit changes (private stemmata don't require sources)
$ lit act -m "Collated lines 1-50"

# View history
$ lit hist
a1b2c3d 2026-03-04 "Collated lines 1-50"
e4f5g6h 2026-03-03 "Initial copy from Venetus A"
```

### Proposal Workflow

```bash
# Create a proposal with source
$ lit prop create iliad-oxford-xkm \
  --target=ed/iliad-oxford \
  --source-type=digital \
  --source-url="https://archive.org/details/homeriilias00home" \
  --source-hash="sha256:def456..." \
  --message="Base text from Archive.org"

# Make a sourced change
$ lit act -m "Corrected line 102" \
  --source-type=print \
  --source-citation="Monro (1897) p.23" \
  --source-mediator="@smith"

# Open for discussion
$ lit prop open iliad-oxford-xkm

# Vote on proposals
$ lit prop vote iliad-oxford-xkm --approve
$ lit prop comment iliad-oxford-xkm -m "Evidence attached"

# Check consensus with source verification
$ lit consensus check iliad-oxford-xkm --verify-sources

# Converge (custos only)
$ lit converge iliad-oxford-xkm --into=ed/iliad-oxford
```

### Source Management

```bash
# Register a source for reuse
$ lit source register \
  --type=print \
  --citation="West, M.L. (1998). Homerus: Ilias." \
  --isbn="9783598714105"

# Verify a digital source
$ lit source verify https://example.com/text.txt \
  --hash="sha256:abc123..."

# Show proposal sources
$ lit prop sources iliad-oxford-xkm

# Export sources as bibliography
$ lit prop sources iliad-oxford-xkm --format=bibtex > sources.bib
```

### Working with Versiones

```bash
# List versiones
$ lit ver list --stemma=ed/iliad-oxford
ed/iliad-oxford/20250304
ed/iliad-oxford/20250315

# Show specific versio
$ lit show grc/homer/iliad/ed/iliad-oxford/20250304 --verse=1.47

# Compare versiones
$ lit delta ed/iliad-oxford/20250304 ed/iliad-oxford/20250315 --verse=1.47

# Cite versio
$ lit cite grc/homer/iliad/ed/iliad-oxford/20250304 --format=bibtex
```

### Role Management

```bash
# List roles
$ lit role list
Codex: grc/homer-iliad

Curatores (radix):
  @smith
  @jones

Custodes (public stemmata):
  @oxford_editor   → ed/iliad-oxford
  @teubner_editor  → ed/iliad-teubner
  @manuscript_scholar → ms/venetus-a

# Add curator
$ lit cur add @patel

# Add custos
$ lit cus add @cambridge_editor --stemma=ed/iliad-cambridge
```

## Integration with Litogram

Litodex provides the verified texts; Litogram provides the practice:

```typescript
// litogram.org backend
async function getText(lid: string) {
    const { content, metadata, sources } = await fetch(`https://api.litodex.org/v1/resolve/${lid}`);
    return {
        typing: strip_markup(content),      // 🌕 Full text
        memorizing: first_letters(content), // 🌗 First letters only
        reciting: blank_page(),              // 🌑 Blank page
        metadata,
        sources: formatCitation(sources)     // "Source: West (1998), p. 47"
    };
}
```

## Why Litodex?

- **For scholars**: Permanently citable versiones, consensus-driven workflow, manuscript tracking, **complete provenance**
- **For students**: Verified texts with source attribution, Litogram integration, citation-ready
- **For institutions**: Hosted collections, private repositories, custom branding, **auditable chains of custody**
- **For humanity**: Preservation of cultural heritage with cryptographic provenance and scholarly rigor

## Source Verification Rules

The system enforces:

1. **Every act in a proposal must have at least one source** (purely metadata changes exempt)
2. **Digital sources must have valid hashes** matching the content at creation time
3. **Print sources must have a mediator** (someone taking responsibility)
4. **Manuscript sources must include library/shelfmark** for identification
5. **Sources cannot be changed** after an act is created (immutability)
6. **All sources must be verified** before convergence to a public stemma

## License

Litodex core is open source under the MIT License. Content licenses are determined by contributors, with source attribution preserved forever.

---

**One platform. One community. Infinite texts. Every change traceable to its source.**
