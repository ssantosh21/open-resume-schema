# Open Resume Schema (ORS) v0.1

A structured, machine-readable resume format for software engineers.

**Author:** Santosh Kumar Dubey
**Status:** Draft v0.1
**License:** Apache 2.0

---

## Why

Resumes are broken. The same information gets reformatted dozens of times — for ATS systems, recruiters, hiring managers, LinkedIn — and meaning gets lost every time.

The problem isn't formatting. It's that resumes have no schema.

- ATS systems guess which blob of text is your job title vs your company name
- Recruiters scan for keywords that may or may not reflect actual skill depth
- Hiring managers get a PDF that tells them nothing about impact magnitude
- Engineers spend hours tweaking margins instead of clarifying their story

ORS fixes this by defining a structured YAML schema for engineer resumes. Write once, render everywhere.

## What It Is

- A YAML schema spec for software engineer resumes
- Machine-readable, human-writable
- Designed for ATS compatibility, recruiter scanning, and technical depth
- Versioned with backward compatibility guarantees

## What It Is Not

- Not a resume builder or SaaS product
- Not a job portal
- Not trying to replace LinkedIn or PDF resumes — it generates them

## Quick Start

```yaml
# resume.yaml
schema: ors/v0.1

basics:
  name: Priya Sharma
  title: Staff Engineer
  location: Bangalore, India
  years_of_experience: 11
  available_from: 30
  summary: >
    Staff engineer with 11 years building distributed systems and data platforms.
    Led migration of monolith to event-driven microservices serving 5M+ users.
  contact:
    email: priya.sharma@example.com
    linkedin: linkedin.com/in/priya-sharma-example
    github: github.com/priya-sharma-example

skills:
  primary:
    - name: Distributed Systems
      depth: expert
      years: 9
      tools: [Kafka, Redis, gRPC, Consul, Envoy]
    - name: Search & Retrieval
      depth: expert
      years: 6
      tools: [Elasticsearch, OpenSearch, Vector Search]
  secondary:
    - name: Java / Kotlin
      depth: expert
      years: 11
    - name: Python
      depth: advanced
      years: 6

experience:
  - company: DataForge Inc.
    title: Staff Engineer
    start: 2021-03
    end: present
    location: Bangalore, India
    type: full-time
    summary: >
      Technical lead for real-time data platform team.
      Owns search infrastructure serving 5M+ daily active users.
    impact:
      - metric: "5M+ DAU served with p99 < 50ms"
        context: "Redesigned search to sharded multi-tenant architecture"
        services: [Elasticsearch, Kafka, Redis]
      - metric: "70% reduction in feature pipeline latency"
        context: "Migrated batch to streaming with Flink"
        before: "6 hour batch refresh"
        after: "< 2 minute streaming"
    technologies: [Java, Kafka, Elasticsearch, Redis, Flink, Kubernetes]

# Optional: side projects and open source
projects:
  - name: StreamBench
    url: github.com/priya-sharma-example/streambench
    role: creator
    summary: "Benchmarking toolkit for stream processing frameworks"
    impact:
      - metric: "200+ GitHub stars"
        context: "Standardized benchmark suite with Docker-based test harness"
    technologies: [Java, Flink, Kafka Streams, Docker]

# Optional: self-acquired skills with evidence
learning:
  - name: "Vector Search & RAG Pipelines"
    period: 2026-01
    depth: intermediate
    evidence: github.com/priya-sharma-example/vector-search-experiments
    topics: [vector embeddings, HNSW, hybrid search, RAG]
```

## Validate

```bash
ors validate resume.yaml
```

## Generate

```bash
ors render resume.yaml --format pdf    # ATS-friendly PDF
ors render resume.yaml --format json   # Machine-readable JSON
ors render resume.yaml --format brief  # 1-page recruiter summary
```

## Schema Design Principles

1. **Structured impact over prose** — every role has quantified `impact` entries with metric, context, and before/after
2. **Skill depth is explicit** — not just "Python" but depth (beginner/intermediate/advanced/expert) and years
3. **Machine-readable, human-writable** — YAML is readable; JSON Schema validates it
4. **Versioned** — `schema: ors/v0.1` in every file, backward compatibility guaranteed
5. **Engineer-first** — designed for software engineers, not generic "professional resumes"
6. **Projects are structured, not free text** — side projects get the same `impact` treatment as work experience
7. **Learning is evidence-based** — self-acquired skills link to proof (repos, posts), not just keyword claims

## Project Structure

```
open-resume-schema/
├── README.md                  # This file
├── DESIGN.md                  # Design rationale and trade-off decisions
├── LICENSE                    # Apache 2.0
├── spec/
│   ├── v0.1/
│   │   ├── schema.yaml        # The schema definition
│   │   └── schema.json        # JSON Schema for validation
│   └── CHANGELOG.md           # Version history
├── examples/
│   ├── staff-engineer.yaml    # Full example
│   └── minimal.yaml           # Minimum viable resume
└── cli/                       # Future: validation + rendering CLI
    └── (planned)
```

## Roadmap

- [x] v0.1 schema — engineer resumes (current)
- [ ] JSON Schema validation
- [ ] CLI: `ors validate`
- [ ] CLI: `ors render --format pdf`
- [ ] CLI: `ors render --format brief` (1-page recruiter summary)
- [ ] v0.2 — education, certifications, publications
- [ ] v0.3 — multi-role support (engineering manager, architect)
- [ ] Community feedback + RFC process

## Contributing

This is an early-stage spec. Feedback welcome via GitHub Issues.

## License

Apache 2.0 — use it, extend it, build on it. Attribution appreciated.
