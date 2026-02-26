# ORS Design Document — v0.1

Author: Santosh Kumar Dubey
Date: February 2026
Status: Draft

---

## Problem Statement

Software engineer resumes are unstructured documents optimized for human reading but hostile to machine processing. This creates three failures:

1. **ATS parsing failure** — Applicant Tracking Systems use heuristics to extract fields from PDFs. They frequently misparse job titles, dates, and skills. Engineers lose opportunities because a parser couldn't find their experience.

2. **Signal loss** — "Built scalable backend" tells a recruiter nothing. Was it 100 users or 10M? Was it a solo project or a team of 20? Unstructured prose hides the signal that matters.

3. **Redundant reformatting** — Engineers maintain 3-5 versions of the same information: LinkedIn, PDF resume, portfolio site, job application forms. Each reformatting introduces drift and errors.

## Design Goals

| Goal | Constraint |
|------|-----------|
| Machine-readable | Must validate against a schema |
| Human-writable | YAML, not XML or JSON |
| ATS-compatible | Output must map to standard ATS fields |
| Impact-first | Every role must have quantified impact, not just responsibilities |
| Versionable | Schema version in every file, backward compatibility guaranteed |
| Engineer-scoped (v0.1) | Only software engineers. No generic "professional" resume |

## Key Design Decisions

### 1. YAML over JSON

**Decision:** YAML as the authoring format, JSON Schema for validation.

**Why YAML:**
- Human-writable without tooling (no bracket matching, no trailing commas)
- Supports multi-line strings naturally (for summaries)
- Comments allowed (engineers can annotate their own resume)
- Every engineer already knows YAML from CI/CD configs

**Why not JSON:**
- No comments — engineers can't annotate decisions
- Verbose for nested structures
- Trailing comma errors are annoying

**Why not TOML:**
- Poor support for deeply nested structures
- Less familiar to most engineers
- Arrays of tables syntax is confusing

**Trade-off accepted:** YAML has gotchas (Norway problem, implicit type coercion). We mitigate with strict JSON Schema validation and explicit string quoting rules in the spec.

### 2. Impact as Structured Data, Not Prose

**Decision:** Every experience entry has an `impact` array with structured fields.

```yaml
impact:
  - metric: "70% reduction in pipeline latency"
    context: "Migrated batch to streaming with Flink"
    before: "6 hour batch refresh"
    after: "< 2 minute streaming"
```

**Why:**
- "Built scalable systems" is noise. "$1,000/day → $50/day" is signal.
- Structured impact enables machine comparison (sort candidates by cost savings, scale handled, team size)
- Forces the author to quantify — if you can't put a number on it, was it impactful?

**Why not free-text bullet points:**
- Every ATS and recruiter tool would need NLP to extract metrics from prose
- Authors default to vague language when unconstrained
- Structured fields are searchable, sortable, comparable

**Trade-off accepted:** Not all impact is quantifiable. We allow `context` as a free-text field for qualitative impact. But `metric` is required — even "Led team of 6" is a metric.

### 3. Skill Depth Is Explicit

**Decision:** Skills have `depth` (beginner/intermediate/advanced/expert) and `years`.

```yaml
skills:
  primary:
    - name: Distributed Systems
      depth: expert
      years: 9
      tools: [Kafka, Redis, gRPC]
```

**Why:**
- "Python" on a resume means nothing. 1 year of scripting ≠ 10 years of production systems.
- Depth + years gives recruiters a 2-second signal without reading the whole resume
- `tools` array maps to ATS keyword matching

**Why not just a flat list:**
- Flat lists ("Python, AWS, Docker, Kubernetes") are the status quo and they're useless
- No way to distinguish "I used Docker once" from "I designed the container orchestration"

**Depth scale:**
- `beginner` — used it in tutorials or side projects
- `intermediate` — used it in production, can work independently
- `advanced` — deep knowledge, can design systems with it, can mentor others
- `expert` — can make architectural decisions, knows the internals, has production war stories

**Trade-off accepted:** Self-reported depth is subjective. But it's still more signal than a flat keyword list. Future versions could link to evidence (GitHub repos, blog posts).

### 4. Primary vs Secondary Skills

**Decision:** Skills split into `primary` (what you lead with) and `secondary` (supporting skills).

**Why:**
- Engineers with 10+ years have 20+ skills. Listing all equally dilutes the signal.
- Primary = "hire me for this." Secondary = "I can also do this."
- Recruiters scan primary skills in 5 seconds. Secondary is for the hiring manager deep-dive.

**Rule:** Primary should be 3-5 skills max. If everything is primary, nothing is.

### 5. Schema Version in Every File

**Decision:** Every resume file starts with `schema: ors/v0.1`.

**Why:**
- Parsers know which schema to validate against
- Old resumes remain valid even as the schema evolves
- Enables migration tooling: `ors migrate resume.yaml --to v0.2`

**Backward compatibility rule:** New versions can ADD fields but never REMOVE or RENAME existing required fields. Optional fields can be deprecated with a 2-version grace period.

### 6. Experience Type Field

**Decision:** Each experience entry has a `type` field: `full-time`, `contract`, `freelance`, `founding`.

**Why:**
- Contract work is legitimate experience but is often hidden or misrepresented
- "Founding engineer" carries different weight than "senior engineer" — the type field captures this
- ATS systems and recruiters can filter by employment type

### 7. Projects as Structured Entries, Not Free Text

**Decision:** Optional `projects` section with the same `impact` structure as experience.

```yaml
projects:
  - name: "StreamBench"
    url: github.com/priya-sharma-example/streambench
    role: creator
    summary: "Benchmarking toolkit for stream processing frameworks"
    impact:
      - metric: "200+ GitHub stars"
        context: "Standardized benchmark suite with Docker-based test harness"
    technologies: [Java, Flink, Kafka Streams, Docker]
```

**Why:**
- Side projects, open source, and portfolio work don't belong under `experience` (no company, no title)
- But they're high-signal for Staff+ hiring — shows initiative, systems thinking, and craft
- Same `impact` structure keeps it consistent — no free-text bullet points

**Why not a generic "additional activities" field:**
- Free text is what we're trying to eliminate
- "Additional activities" becomes a dumping ground for hobbies, volunteering, and filler
- If it has impact, structure it. If it doesn't, ORS doesn't want it.

**`role` enum:** `creator` (built from scratch), `maintainer` (actively maintain), `contributor` (meaningful contributions). Tells the reader your relationship to the project in one word.

### 8. Learning Section for Self-Acquired Skills

**Decision:** Optional `learning` section for skills acquired outside of work experience.

```yaml
learning:
  - name: "Vector Search & RAG Pipelines"
    period: 2026-01
    depth: intermediate
    evidence: github.com/priya-sharma-example/vector-search-experiments
    topics: [vector embeddings, HNSW, hybrid search, RAG]
```

**Why:**
- Engineers transitioning into new domains (e.g., AI/ML) learn deeply through self-study, but this doesn't show up under any `experience.technologies`
- Listing "LangGraph" under skills without context is a keyword — linking to a repo that proves it is evidence
- `topics` array solves ATS keyword matching for skills that don't fit under any job title
- Separates "used at work" from "learned independently" — both are valid, but they're different signals

**Why not just add these to `skills`:**
- Skills says "I know this." Learning says "I learned this recently, here's proof."
- For someone with 12 years of experience adding AI/ML skills, the distinction matters
- Interviewers see a `learning` section and think "actively growing" — strong Staff+ signal

**`evidence` field:** Optional but powerful. Links to the repo, blog post, or certificate that proves the learning. Moves the resume from "claims" to "show me."

**Trade-off accepted:** Self-reported depth is still subjective. But `evidence` + `topics` gives the reader enough to verify independently.

### 9. No Education in v0.1

**Decision:** Education is deferred to v0.2.

**Why:**
- For engineers with 10+ years, education is the least important section
- Adding it now increases schema complexity without proportional value
- v0.1 is scoped to "what matters for Staff+ engineer hiring": skills, impact, experience

**v0.2 plan:** Add optional `education` and `certifications` sections.

---

## What We Explicitly Rejected

### Rejected: Skill endorsements / ratings (1-10 scale)

Numeric ratings are meaningless without calibration. One person's "8/10 in Python" is another person's "5/10." The depth enum (beginner/intermediate/advanced/expert) is coarser but more honest.

### Rejected: Project-level granularity

Some schemas break experience into projects within each role. This adds complexity and most engineers can't cleanly separate "projects" from ongoing responsibilities. Impact entries serve the same purpose with less structure overhead. Standalone projects (open source, side projects) belong in the dedicated `projects` section instead.

### Rejected: Cover letter field

Cover letters are per-application, not per-resume. They don't belong in a canonical resume schema.

### Rejected: Salary expectations

Sensitive, volatile, and context-dependent. Not a resume concern.

### Rejected: Photo / personal details

Age, gender, photo, marital status — none of these belong in a technical resume. Excluding them by design is a feature.

---

## Schema Evolution Plan

| Version | Scope | Breaking? |
|---------|-------|-----------|
| v0.1 | Engineer resumes: basics, skills, experience, impact, projects, learning | — |
| v0.2 | + education, certifications, publications, community | No |
| v0.3 | + engineering manager / architect variants | No |
| v0.4 | + multi-language support (i18n) | No |
| v1.0 | Stable release after community feedback | Possible cleanup |

**Rule:** No breaking changes until v1.0. All additions are optional fields.

---

## Comparison with Existing Formats

| | ORS | JSON Resume | LinkedIn Export | PDF |
|---|---|---|---|---|
| Machine-readable | Yes (YAML + JSON Schema) | Yes (JSON) | Partial (HTML) | No |
| Impact structured | Yes (metric + context + before/after) | No (free text) | No | No |
| Skill depth | Yes (depth + years + tools) | No (flat list) | Endorsements (noisy) | No |
| Self-study / learning | Yes (evidence-linked) | No | No | No |
| Projects with impact | Yes (structured) | Yes (free text) | No | No |
| Versioned schema | Yes | Yes | No | No |
| Human-writable | Yes (YAML) | Awkward (JSON) | No (UI only) | No (Word/LaTeX) |
| ATS-optimized | By design | Partial | N/A | Depends on formatting |

JSON Resume (jsonresume.org) is the closest existing standard. ORS differs in four ways:
1. YAML over JSON for authoring (human-writable)
2. Structured impact (not free-text bullet points)
3. Skill depth model (not flat keyword lists)
4. Evidence-based learning section for self-acquired skills

---

## Open Questions for Community Feedback

1. Should `impact.metric` be required or optional? (Current: required)
2. Is the 4-level depth scale (beginner/intermediate/advanced/expert) sufficient?
3. Should the schema support non-engineer roles in v0.1 or stay scoped?
4. Should `learning.evidence` be required or stay optional? (Current: optional)

---

## References

- JSON Resume: https://jsonresume.org
- OpenAPI Specification (design inspiration for versioning): https://spec.openapis.org
- YAML 1.2 Spec: https://yaml.org/spec/1.2.2/
