# Agent Skills

> A curated archive of custom agent skills that I use across AI coding assistants.

## 📦 Available Skills

| Skill | Description | Install Command |
| :--- | :--- | :--- |
| [**`bulk-commit-clustering`**](./bulk-commit-clustering) | Groups and batches uncommitted working tree changes into clean, atomic commits with user approval. | `npx skills add assignment-sets/agent-skills --skill bulk-commit-clustering` |
| [**`openapi-contract-fetcher`**](./openapi-contract-fetcher) | Fetches clean REST endpoint contracts and compact DSL schemas directly from OpenAPI/Swagger docs. | `npx skills add assignment-sets/agent-skills --skill openapi-contract-fetcher` |
| [**`plain-talk`**](./plain-talk) | Delivers direct, candid, informal explanations with grounded real-world analogies and zero corporate fluff. | `npx skills add assignment-sets/agent-skills --skill plain-talk` |

---

## 🚀 Quick Start

### Install a specific skill
```bash
npx skills add assignment-sets/agent-skills --skill <skill-name>
```

### Install all skills from this repository
```bash
npx skills add assignment-sets/agent-skills
```

### Install globally (user-level)
```bash
npx skills add assignment-sets/agent-skills --skill <skill-name> -g -y
```
