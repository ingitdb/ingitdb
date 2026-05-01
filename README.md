# inGitDB

<img src="https://github.com/ingitdb/.github/raw/main/inGitDB-full4.png" alt="inGitDB Logo" width="400" />

**The main repository** for [inGitDB](https://github.com/ingitdb) — an open-source versioned database that stores data in text files (JSON, YAML, columnar storage) with Git-based version control.

This repo is the central hub for **project planning**, **issue tracking**, **architecture documentation**, and **AI agent prompts** across the entire inGitDB ecosystem.

---

## 🔗 Quick Navigation

| Section                                                                | Description                                   |
| ---------------------------------------------------------------------- | --------------------------------------------- |
| [📋 Project Overview](#what-is-ingitdb)                                | What inGitDB is and why it exists             |
| [🏗️ Architecture](#architecture)                                       | System components and how they connect        |
| [📦 Repositories](#ecosystem-repositories)                             | All repos in the inGitDB organization         |
| [📄 Documentation](./docs/)                                            | Architecture specs, API reference, dev plan   |
| [🤖 AI Prompts](./ai/prompts/)                                         | 10 ready-to-use prompts for AI coding agents  |
| [📐 Development Plan](./docs/dev-plan/)                                | Execution plan, skills matrix, agent roles    |
| [🔌 OpenAPI Spec](./docs/open-api/)                                    | REST API contract (OpenAPI 3.0, 24 endpoints) |
| [🧩 TypeScript Client Architecture](./docs/ingitdb-ts-architecture.md) | Client design, fluent API, query builder      |
| [⚙️ GitHub Actions Integration](./docs/github-actions-integration.md)  | CI/CD workflows and custom validation actions |
| [📐 Schemas](https://github.com/ingitdb/ingitdb-schema)                | JSON Schema definitions for inGitDB files     |

---

## What is inGitDB?

inGitDB is a **Git-backed versioned database** designed for teams who want to manage structured data with the same workflows they use for code.

### Key Principles

- **🔀 Version Control** — full history, branching, and merging of data changes
- **👥 Collaboration** — multiple users work on data using Git-native workflows (PRs, reviews, branches)
- **📝 Text-Based Storage** — JSON, YAML, and columnar formats that are human-readable and Git-diffable
- **✅ Validation** — schema validation and referential integrity checks via CLI and GitHub Actions
- **📊 Reference Data** — ideal for configuration, reference tables, and slowly-changing data

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                   GitHub Actions                     │
│              (Validation & CI/CD)                    │
└─────────────────────────────────────────────────────┘
                         ▲
                         │ Uses
                         │
┌────────────────┐   ┌───▼──────────────┐   ┌─────────────────┐
│  ingitdb-ts    │◄──┤  ingitdb-server  │───┤   ingitdb-go    │
│  (TS Client)   │   │   (REST API)     │   │   (Go CLI/Lib)  │
└────────────────┘   └──────────────────┘   └─────────────────┘
                              │
                              ▼
                     ┌────────────────┐
                     │   Git Storage  │
                     │ (JSON/YAML/CSV)│
                     └────────────────┘
```

### Key Concepts

| Concept        | Description                                                                    |
| -------------- | ------------------------------------------------------------------------------ |
| **Database**   | A named set of collections stored as a Git repository or subdirectory          |
| **Collection** | A set of items validated against a JSON Schema                                 |
| **Item**       | An individual data entry with auto-generated `_id`, `_version`, and timestamps |
| **Branch**     | A Git branch for isolating data changes                                        |
| **Commit**     | A Git commit representing a point-in-time snapshot of database state           |

> 💡 **Collections vs Tables:** Unlike traditional database tables where records are flat rows, a collection **item** can itself contain **sub-collections** — enabling naturally hierarchical data structures without joins or foreign keys.

---

## Ecosystem Repositories

### Core

| Repository                                                      | Description                                                                   | Status                                                                                                                            |
| --------------------------------------------------------------- | ----------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| **[`ingitdb`](https://github.com/ingitdb/ingitdb)** (this repo) | Main hub — planning, issues, AI prompts, documentation                        | Active                                                                                                                            |
| [`ingitdb-go`](https://github.com/ingitdb/ingitdb-go)           | Go CLI & library — core validation, data management, server (`serve` command) | [![Build](https://github.com/ingitdb/ingitdb-go/actions/workflows/golangci.yml/badge.svg)](https://github.com/ingitdb/ingitdb-go) |
| [`ingitdb-ts`](https://github.com/ingitdb/ingitdb-ts)           | TypeScript client library for inGitDB                                         | Active                                                                                                                            |
| [`ingitdb-schema`](https://github.com/ingitdb/ingitdb-schema)   | JSON Schema definitions for inGitDB databases                                 | Active                                                                                                                            |

### GitHub Actions & CI/CD

| Repository                                                                  | Description                                                    |
| --------------------------------------------------------------------------- | -------------------------------------------------------------- |
| [`ingitdb-action`](https://github.com/ingitdb/ingitdb-action)               | Lightweight GitHub Action for running inGitDB validation       |
| [`ingitdb-github-action`](https://github.com/ingitdb/ingitdb-github-action) | Full-featured GitHub Action (TypeScript) for CI/CD integration |

### Demo & Test Databases

| Repository                                                      | Description                                                 |
| --------------------------------------------------------------- | ----------------------------------------------------------- |
| [`demo-ingitdb`](https://github.com/ingitdb/demo-ingitdb)       | Demo database with countries, tasks, and sample collections |
| [`ingitdb-demo-db`](https://github.com/ingitdb/ingitdb-demo-db) | Additional demo database                                    |
| [`ingitdb-test-db`](https://github.com/ingitdb/ingitdb-test-db) | Test database used for automated testing                    |

---

## What's in This Repo

This repository doesn't contain application source code. Instead, it serves as the **project hub**:

```
ingitdb/
├── README.md                              ← You are here
├── AGENTS.md                              ← AI agent guidelines (all agents)
├── CLAUDE.md                              ← Claude Code configuration
├── LICENSE                                ← CC0 1.0 Universal
├── docs/                                  ← Planning & architecture documents
│   ├── dev-plan/                          ←   Skills matrix, agents, execution plan
│   ├── open-api/                          ←   OpenAPI spec & REST API reference
│   ├── ingitdb-ts-architecture.md         ←   TypeScript client architecture
│   ├── github-actions-integration.md      ←   CI/CD workflows & actions
│   └── summary.md                         ←   Deliverable status & next steps
└── ai/
    └── prompts/                           ← 10 AI development task prompts
```

---

## Schemas

JSON Schema definitions for inGitDB configuration and collection files are available in the [`ingitdb-schema`](https://github.com/ingitdb/ingitdb-schema) repository:

| Schema File                                                                                                              | Validates                        | Description                                                                        |
| ------------------------------------------------------------------------------------------------------------------------ | -------------------------------- | ---------------------------------------------------------------------------------- |
| [`ingitdb-collection.schema.json`](https://github.com/ingitdb/ingitdb-schema/blob/main/ingitdb-collection.schema.json)   | `.ingitdb-collection.yaml` files | Collection structure — columns, record file format, ordering, and localized titles |
| [`ingitdb-root-config.schema.json`](https://github.com/ingitdb/ingitdb-schema/blob/main/ingitdb-root-config.schema.json) | `.ingitdb.yaml` files            | Root configuration mapping collection IDs to directory paths                       |

These schemas are derived from the Go struct definitions in [`ingitdb-go`](https://github.com/ingitdb/ingitdb-go). See the [`ingitdb-schema` README](https://github.com/ingitdb/ingitdb-schema) for full property documentation.

---

## Quick Start

### For Project Managers

1. Review the [Execution Plan](./docs/dev-plan/#execution-plan) — timeline and milestones
2. Check the [Skills Matrix](./docs/dev-plan/#skills-matrix) — team requirements
3. Skim [Risk Management](./docs/dev-plan/#risk-management) — known risks and mitigations
4. Track progress via [Summary](./docs/summary.md)

### For Developers

1. Browse the [Architecture Documentation](./docs/)
2. Reference the [OpenAPI Spec](./docs/open-api/) for the REST API contract
3. Follow [Technology Stack](./docs/dev-plan/#technology-stack-recommendations) recommendations
4. Use [AI Prompts](./ai/prompts/) for guided development tasks

### For AI Agents

1. Start with [AGENTS.md](./AGENTS.md) for complete guidelines
2. Find your [role and deliverables](./docs/dev-plan/#ai-agents--responsibilities)
3. Pick your [prompt](./ai/prompts/) and follow acceptance criteria

### Generate Server from OpenAPI

```bash
npx @openapitools/openapi-generator-cli generate \
  -i docs/openapi-spec.yaml \
  -g nodejs-express-server \
  -o ./ingitdb-server
```

### Generate TypeScript Client

```bash
npx @openapitools/openapi-generator-cli generate \
  -i docs/openapi-spec.yaml \
  -g typescript-axios \
  -o ./ingitdb-ts/src/generated
```

---

## Development Phases

| Phase | Duration    | Focus                            | Status         |
| ----- | ----------- | -------------------------------- | -------------- |
| 1     | Weeks 1–2   | API Design & OpenAPI Spec        | ✅ Done        |
| 2     | Weeks 3–6   | Server Implementation            | 🔄 In Progress |
| 3     | Weeks 5–7   | TypeScript Client                | 🔄 In Progress |
| 4     | Weeks 7–8   | GitHub Actions Integration       | Planned        |
| 5     | Weeks 9–10  | Documentation & Polish           | Planned        |
| 6     | Weeks 11–12 | Testing & Release                | Planned        |
| 7     | Weeks 13–16 | Extended Features & Integrations | Planned        |

See the full [Development Plan](./docs/dev-plan/) for details.

---

## Contributing

Contributions are welcome! This repo is best for:

- **Planning proposals** — open an issue or PR to discuss architecture changes
- **Documentation improvements** — fix typos, add guides, improve clarity
- **Issue tracking** — report bugs or request features across the ecosystem

For code contributions to implementations:

| Component                   | Repository                                                                          |
| --------------------------- | ----------------------------------------------------------------------------------- |
| Go CLI & Server             | [`ingitdb/ingitdb-go`](https://github.com/ingitdb/ingitdb-go)                       |
| TypeScript Client           | [`ingitdb/ingitdb-ts`](https://github.com/ingitdb/ingitdb-ts)                       |
| GitHub Actions              | [`ingitdb/ingitdb-github-action`](https://github.com/ingitdb/ingitdb-github-action) |
| Schema Definitions          | [`ingitdb/ingitdb-schema`](https://github.com/ingitdb/ingitdb-schema)               |

---

## Links

- 🌐 [inGitDB Organization](https://github.com/ingitdb)
- 📖 [inGitDB Documentation](https://ingitdb.github.io/)
- 🛠️ [ingitdb-go CLI](https://github.com/ingitdb/ingitdb-go)

## License

CC0 1.0 Universal — see [LICENSE](./LICENSE) for details.
Implementation repositories are licensed under MIT.
