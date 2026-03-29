# AI-GECO — Grounded Entity Context Object
### Specification Version 1.0 · Draft

*A portable, model-agnostic standard for grounding AI language models with authoritative entity context — reducing hallucinations and maximizing token efficiency.*

---

| Ground | Reduce hallucinations | Save tokens |
|--------|----------------------|-------------|
| Provide AI models with a factual, authoritative starting point for any entity. | Fill the gaps that AI models otherwise fill with plausible but fabricated detail. | Deliver maximum context in minimum tokens — preserving the context window for conversation. |

---

## Table of Contents

1. [Purpose](#1-purpose)
2. [Scope](#2-scope)
3. [Design Principles](#3-design-principles)
4. [Structure](#4-structure)
5. [The Instruction Key](#5-the-instruction-key)
6. [Entity Types](#6-entity-types)
7. [Multi-Entity Networks](#7-multi-entity-networks)
8. [Implementation Guide](#8-implementation-guide)
9. [Token Efficiency](#9-token-efficiency)
10. [Versioning and Extensibility](#10-versioning-and-extensibility)
- [Appendix A — Blank Schema](#appendix-a--blank-schema)
- [Appendix B — Example Instances](#appendix-b--example-instances)
- [Appendix C — Changelog](#appendix-c--changelog)

---

## 1. Purpose

> **An AI-GECO is the minimum authoritative context an AI model needs to reason accurately about an entity. It is not a comprehensive profile — it is a factual anchor.**

AI language models hallucinate when they lack authoritative information. Faced with a gap, a model will generate a plausible-sounding answer rather than admit uncertainty — inventing dates, misattributing relationships, fabricating identifiers, or confusing one entity with another. This is not a failure of intelligence; it is an artifact of how large language models generate text.

The Grounded Entity Context Object (AI-GECO) addresses this directly. It is a compact, structured JSON file that declares authoritative facts about a single entity — a person, an animal, a product, a company, a place, a creative work, or anything else — and instructs the AI model to treat those facts as ground truth for every response in the session.

When an AI-GECO file is uploaded to an AI platform as persistent context, three things happen:

- The model is **grounded** — it has a factual starting point it can reason from rather than generate toward.
- Hallucination risk is **reduced** — the gaps the model would otherwise fill with fabrication are filled with declared facts.
- Tokens are **conserved** — the compact format delivers maximum context at minimum cost to the context window.

AI-GECO is model-agnostic and platform-agnostic. A single AI-GECO file can be uploaded to Claude Projects, ChatGPT Custom GPTs, Gemini Gems, Microsoft Copilot Agents, or any AI platform that supports file-based context injection. The file describes the entity. The model provides the intelligence.

---

## 2. Scope

AI-GECO is designed for any entity about which an AI model may need to reason. The standard defines seven entity types:

| `entity_type` | Applies to | Examples |
|---------------|------------|---------|
| `person` | Individual humans | Executives, clients, patients, historical figures, public figures, yourself |
| `animal` | Any animal | Pets, livestock, therapy animals, working animals, wildlife subjects |
| `product` | Physical goods | Packaged food, electronics, pharmaceuticals, manufactured items, components |
| `organization` | Entities and institutions | Companies, nonprofits, government agencies, schools, teams, clubs |
| `place` | Locations | Properties, venues, cities, regions, landmarks, facilities |
| `creative_work` | Works of authorship | Books, films, albums, artworks, software, podcasts, articles |
| `other` | Anything else | Concepts, events, vehicles, legal instruments — any entity not covered above |

Each entity type has a suggested extended module — a set of fields most commonly relevant to that kind of entity. Suggested modules are non-binding. Any extended field may be used with any entity type, and unused modules should be omitted from the file entirely.

---

## 3. Design Principles

Every design decision in AI-GECO is governed by three principles, applied in priority order.

### 3.1 Ground the model

Every field in an AI-GECO must serve a grounding purpose. A field that does not give the model authoritative information it would otherwise lack — or risk fabricating — does not belong in the standard. AI-GECO is not a rich entity description. It is a targeted set of facts chosen specifically because their absence creates hallucination risk.

The `_meta.instruction` key makes grounding explicit. It tells the model that the values in this file are authoritative — not claims to evaluate, not background to consider, but ground truth to apply. This is the most important field in any AI-GECO instance.

### 3.2 Reduce hallucination surface area

Hallucinations cluster around specific types of missing information: proper names and their variants, dates, identifiers, relationships between entities, and domain-specific terminology. AI-GECO targets these exactly. The core block captures names, aliases, dates, identifiers, and relationships. The `context_index` extended module captures terminology and known ambiguities. Together they close the gaps most likely to produce fabricated output.

A useful mental test for any candidate AI-GECO field: *if a model did not have this value, would it be likely to generate a plausible-sounding but potentially wrong answer?* If yes, include it. If the model would simply say it does not know, the field is lower priority.

### 3.3 Maximize token efficiency

Context window space consumed by the AI-GECO file is space unavailable for conversation. AI-GECO is deliberately compact. Specific format choices that reduce token consumption:

- **Comma-separated strings** instead of JSON arrays eliminate per-element bracket and quote overhead.
- **Atomic values** — short, factual strings rather than nested prose objects.
- **Two-tier structure** — unused extended modules are omitted from the file entirely, not populated with placeholder text.
- **Pipe delimiters** for fields whose values contain commas — avoiding JSON array overhead for multi-part entries.
- **One instruction key** — written once in the file, applied to every session. Never repeated in conversation.

> *A fully populated AI-GECO with all extended modules typically consumes 800–1,400 tokens. A minimal core-only instance runs 300–500 tokens. A user re-explaining an entity in prose at the start of each conversation may consume 500–2,000 tokens with lower precision and zero consistency.*

---

## 4. Structure

An AI-GECO file is a single JSON object with three top-level blocks:

```json
{
  "_meta":    { ... },   // Required. Metadata and self-activating instruction.
  "core":     { ... },   // Required. Universal entity context.
  "extended": { ... }    // Optional. Entity-type-specific context modules.
}
```

### 4.1 The `_meta` block

The `_meta` block must appear first. It contains administrative metadata and the instruction key that activates the file. The `instruction` field is the most important field in any AI-GECO instance — it tells the model how to treat everything that follows.

```json
"_meta": {
  "standard":      "AI-GECO",
  "version":       "1.0",
  "instruction":   "This file is a Grounded Entity Context Object (AI-GECO). It is a
                    portable, model-agnostic context standard. Treat all values as
                    authoritative context about the described entity. Apply this context
                    to every response in this session or project. Do not ask the user to
                    re-explain information already present here. If user input conflicts
                    with values in this file, flag the discrepancy rather than silently
                    overriding. Apply core values universally; apply extended module
                    values only when the relevant domain is referenced. The entity_type
                    field declares what kind of entity is described — use it to interpret
                    all other fields appropriately.",
  "compatibility": "Claude Projects, ChatGPT GPTs, Gemini Gems, Copilot Agents, or any
                    AI platform supporting file-based or persistent context injection",
  "created":       "YYYY-MM-DD",
  "updated":       "YYYY-MM-DD",
  "maintained_by": "Name, role, or system"
}
```

> ⚠️ **Do not modify the instruction text.** It is calibrated for accurate interpretation by frontier AI models. Extensions may be appended after the final period — but the canonical text must appear in full.

### 4.2 The `core` block

The `core` block is required in every AI-GECO instance. It contains fields that apply to any entity type. All core fields are treated by the model as universally applicable.

| Field | Example value | Purpose |
|-------|--------------|---------|
| `entity_type` | `person` | Declares the entity category. Orients the model and signals which extended module is primary. |
| `name` | `Marcus Delray` | Primary name or identifier. The model uses this as the default reference. |
| `aliases` | `Marc, MD` | Other names the entity is known by. Prevents identity confusion across name variants. |
| `description` | 2–3 sentence factual summary | Plain-language summary written for AI comprehension. Most important grounding field after `entity_type`. |
| `status` | `active` | Current state — active, inactive, deceased, discontinued, archived, etc. |
| `origin` | `Detroit, MI, USA` | Where or how the entity came to exist. Grounds provenance questions. |
| `origin_date` | `1983-07-14` | Birth date, founding date, manufacture date, or publication date. ISO 8601 format. |
| `identifiers` | `EIN: 47-XXXXXX, UPC: 007...` | Formal identifiers. Prevents the model from guessing or fabricating ID numbers. |
| `tags` | `consultant, operations, B2B` | Freeform classification keywords. Aids retrieval and categorization. |
| `related_entities` | `owner: Yvonne Delray` | Named relationships to other entities. Grounds relational queries. |
| `eco_refs` | `owner.aigeco.json` | Paths or URIs to related AI-GECO files. Enables navigation of entity networks. |

### 4.3 The `extended` block

The `extended` block is optional. It contains entity-type-specific modules, each scoped to a declared `entity_type`. The model applies an extended module only when the relevant domain arises in conversation.

The `_suggested_for` field in each module is informational — it signals the intended entity type but does not prevent use with other types. Unused modules should be omitted from the file entirely rather than left empty.

| Module | Suggested for | Contains |
|--------|--------------|---------|
| `person` | `person` | Legal name, pronouns, DOB, nationality, languages, occupation, employer, education, skills, contact, location, AI guidance preferences |
| `animal` | `animal` | Species, breed, sex, DOB, markings, microchip ID, owner, veterinarian, vaccinations, medical conditions, medications, diet, temperament, training, registration |
| `product` | `product` | Manufacturer, brand, model, SKU, UPC, category, dimensions, weight, materials, origin, dates, certifications, regulatory status, intended use, warnings, storage, nutritional info, allergens, price, supplier |
| `organization` | `organization` | Legal name, entity type, industry, location, founding, headcount, structure, key roles, departments, business model, customers, regions, mission, values, priorities, systems, compliance, AI guidance |
| `place` | `place` | Place type, address, coordinates, jurisdiction, timezone, area, owner, operator, capacity, access, hours, established date, notable features, zoning, historical significance |
| `creative_work` | `creative_work` | Work type, creator, contributors, publisher, publication date, edition, language, genre, format, identifier, synopsis, audience, content rating, awards, copyright, license |
| `other` | `other` | No predefined fields. Use `x_[namespace]_[fieldname]` convention for custom fields. |
| `context_index` | all types | Glossary of entity-specific terms, known ambiguities (terms that mean something specific here vs. common usage), and custom identifiers |

---

## 5. The Instruction Key

The `_meta.instruction` field is what makes an AI-GECO file self-activating. Without it, an uploaded file is passive reference material — a model may or may not consult it depending on the query. With it, the file carries its own behavioral directive: treat these values as ground truth, apply them universally, and do not seek re-confirmation of facts already declared.

This is the mechanism by which AI-GECO reduces hallucination. The model is not asked to *consider* the file — it is instructed to *anchor* on it. The distinction matters. A model that considers context may still generate elaborations, embellishments, or contradictory inferences. A model that anchors on declared facts treats fabrication of those facts as an error to avoid.

### 5.1 What the instruction does

- **Declares the file type** — identifies this as an AI-GECO so the model applies appropriate interpretation.
- **Establishes authority** — instructs the model to treat values as authoritative, not as claims to verify.
- **Scopes application** — core values apply universally; extended values apply when their domain is relevant.
- **Prevents redundant questioning** — the model does not ask the user to re-explain information already declared.
- **Establishes conflict handling** — if user input contradicts a declared value, the model flags the discrepancy rather than silently overriding.
- **Orients via entity type** — instructs the model to use the `entity_type` field to frame interpretation of all other fields.

### 5.2 Canonical instruction text

The following is the canonical instruction text for AI-GECO v1.0. Use it verbatim. Extensions may be appended after the final period.

```
"instruction": "This file is a Grounded Entity Context Object (AI-GECO). It is a portable,
  model-agnostic context standard. Treat all values as authoritative context about the
  described entity. Apply this context to every response in this session or project. Do not
  ask the user to re-explain information already present here. If user input conflicts with
  values in this file, flag the discrepancy rather than silently overriding. Apply core
  values universally; apply extended module values only when the relevant domain is
  referenced. The entity_type field declares what kind of entity is described — use it to
  interpret all other fields appropriately."
```

---

## 6. Entity Types

The `entity_type` field is declared in the core block and must be one of seven values. It performs two functions: it orients the model to the nature of the entity, and it signals which extended module contains the most relevant domain-specific context.

### 6.1 `person`

Use for individual humans — living, historical, public, or private. The person extended module captures professional, biographical, and communication context. The `ai_guidance` sub-object within the person module allows the maintainer to specify preferred tone, topics to avoid, and known communication preferences — making AI-GECO useful as a personal context layer for AI assistants operating on behalf of or in support of a specific individual.

### 6.2 `animal`

Use for any animal — pets, livestock, therapy animals, working animals, or wildlife subjects. The animal extended module is designed to support veterinary, care, and registration contexts. Fields for vaccinations, medications, temperament, and training reflect the most common informational needs when an AI is asked to reason about an animal's care, history, or behavior.

### 6.3 `product`

Use for physical goods and manufactured items of any kind — packaged food, electronics, apparel, pharmaceuticals, industrial components. The product extended module covers the full range of product identity fields, from UPC and SKU to nutritional information, allergens, certifications, and regulatory status. An AI-GECO for a can of vegetables and an AI-GECO for a medical device use the same module with different fields populated.

### 6.4 `organization`

Use for any organization — corporation, nonprofit, government agency, school, team, or club. The organization extended module carries the most fields of any module, reflecting the complexity of organizational context. It includes structure, operations, mission, systems, compliance, and a full `ai_guidance` sub-object equivalent to the one formerly defined in AI-OCO.

### 6.5 `place`

Use for any location — a property, venue, city, region, landmark, or facility. The place module captures physical, jurisdictional, operational, and historical context. It is useful when an AI is asked to reason about a specific location's characteristics, ownership, access, or history.

### 6.6 `creative_work`

Use for works of authorship — books, films, albums, artworks, software, podcasts, articles. The `creative_work` module captures authorship, publication, format, content, and rights information. It is particularly useful for AI assistants operating in research, publishing, or media contexts.

### 6.7 `other`

Use for any entity that does not fit the above types — vehicles, legal instruments, events, concepts. The `other` module defines no predefined fields. Custom fields should use the `x_[namespace]_[fieldname]` convention to avoid conflicts with future specification versions.

---

## 7. Multi-Entity Networks

Each AI-GECO file describes exactly one entity. Related entities are referenced, not embedded. The `core.related_entities` field names relationships in plain language. The `core.eco_refs` field provides paths or URIs to related AI-GECO files.

```json
"related_entities": "owner: Yvonne Delray, veterinarian: Dr. Sandra Osei",
"eco_refs": "yvonne-delray.aigeco.json, lakeview-animal-hospital.aigeco.json"
```

This pattern allows a network of entities to be navigated by the model without any single file needing to contain everything. A query about the owner of a pet can be answered from the pet's AI-GECO; a follow-up about the owner's employer can be answered by loading the owner's AI-GECO. Each file remains focused, compact, and independently maintainable.

For organizations with subsidiaries or a parent company, each entity has its own AI-GECO file. The parent references children via `eco_refs`; children reference the parent. A model operating in the context of a subsidiary AI-GECO is aware of the parent relationship without needing the full parent file in the same session.

---

## 8. Implementation Guide

### 8.1 Creating an AI-GECO file

1. Start with the blank schema (`ai-geco_schema_v1.json`). Do not modify the canonical instruction text in `_meta.instruction`.
2. Set `entity_type` first. This decision shapes which extended module to populate.
3. Populate all `core` fields. The `description` field requires the most care — write it for AI comprehension, not for human readers. Be precise and factual.
4. Add extended fields only where they serve a grounding purpose. If a field would not reduce hallucination risk, omit it.
5. Leave unused extended modules out of the file entirely — do not include them with empty or placeholder values.
6. Validate the file as valid JSON before uploading.
7. Update `_meta.updated` every time any value changes.

### 8.2 Platform upload instructions

| Platform | How to attach an AI-GECO file |
|----------|-------------------------------|
| **Claude (Anthropic)** | Create a Project → open Project settings → upload the `.aigeco.json` file as a project file. Available in all conversations within that project. |
| **ChatGPT (OpenAI)** | Create a Custom GPT → Configure tab → upload the `.aigeco.json` file under Knowledge. Referenced in all GPT conversations. |
| **Gemini (Google)** | Create a Gem → in Gem configuration, add the `.aigeco.json` file as a reference document. |
| **Copilot (Microsoft)** | Copilot Studio → create an Agent → attach the `.aigeco.json` file as a knowledge source in the agent configuration. |
| **Other platforms** | Upload the `.aigeco.json` file to any context, knowledge, or system prompt file attachment feature. The `_meta.instruction` field is self-contained and does not require platform-specific configuration. |

### 8.3 Maintenance

An AI-GECO is only as useful as it is accurate. Stale context is worse than no context — a model anchored on outdated facts will produce confidently wrong answers. Recommended maintenance practices:

- `_meta.updated` — update every time any value changes. Models aware of the date may flag stale context.
- `_meta.maintained_by` — assign a named person, role, or system responsible for keeping the file current.
- **Review frequency** — for people and organizations, quarterly. For products, on each release or formulation change. For animals, after each veterinary visit.
- **High-volatility fields** — `status`, `key_roles`, `strategic_priorities`, `medications`, and `vaccinations` are most likely to require frequent updates.

---

## 9. Token Efficiency

Token efficiency is a design constraint, not an afterthought. The AI-GECO format is shaped by it at every level. The goal is to deliver maximum grounding value per token — so that the context window is dominated by conversation, not overhead.

| Approach | Instead of | Token saving |
|----------|-----------|-------------|
| Comma-separated strings: `"Value A, Value B, Value C"` | JSON arrays: `["Value A", "Value B", "Value C"]` | ~30–50% per multi-value field |
| Pipe delimiters for compound entries: `"A: x \| B: y \| C: z"` | Nested objects: `{"A": "x", "B": "y", "C": "z"}` | ~40% per compound field |
| Omit unused modules entirely | Include with null or placeholder values | Hundreds of tokens per omitted module |
| Atomic string values | Nested prose objects with descriptions | 60–80% per field |
| One instruction key per file | Re-explaining context in each conversation | 500–2,000 tokens per session |

---

## 10. Versioning and Extensibility

### 10.1 Version policy

The `_meta.version` field records the specification version the file conforms to. AI-GECO uses semantic versioning. Breaking changes — renamed required fields, structural changes to the core block — increment the major version. Additive changes — new optional fields or extended modules — increment the minor version. Files conforming to an older version remain valid and interpretable.

### 10.2 Custom fields

Custom fields may be added to any extended module or to the `other` module using the `x_[namespace]_[fieldname]` convention:

```json
"extended": {
  "x_acme_internal_id":     "EMP-2024-0047",
  "x_acme_clearance_level": "Level 3",
  "x_acme_assigned_region": "Central"
}
```

Custom fields are passed to the model as context. Use them sparingly — every custom field consumes tokens and dilutes the grounding signal of standardized fields. A custom field should only be added if its absence creates a meaningful hallucination risk that the standard fields do not address.

### 10.3 Relationship to prior standards

AI-GECO supersedes two earlier iterations developed during the evolution of this standard:

- **AI-OCO** (AI Organizational Context Object) — scoped to organizational entities only.
- **ECD** (Entity Context Dataset) — a universal entity scope iteration, renamed before final release.

AI-GECO is the universal successor covering all entity types. Existing AI-OCO files may be migrated by mapping AI-OCO fields to AI-GECO `core` and `extended.organization` fields.

---

## Appendix A — Blank Schema

The canonical blank AI-GECO schema is maintained in `ai-geco_schema_v1.json`, distributed alongside this specification. Populate this file to create an AI-GECO instance for any entity type.

**File naming convention:** `[entity-name].aigeco.json`

Examples: `marcus-delray.aigeco.json`, `biscuit.aigeco.json`, `halcyon-bridge.aigeco.json`

---

## Appendix B — Example Instances

Four fully populated example instances are provided in `ai-geco_examples_v1.json`, distributed alongside this specification. The examples demonstrate the full range of entity types:

- **Marcus Delray** — `person` (management consultant)
- **Biscuit** — `animal` (therapy dog)
- **Muir Glen Organic Diced Tomatoes** — `product` (canned food)
- **Halcyon Bridge** — `organization` (B2B SaaS company)

---

## Appendix C — Changelog

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-03-27 | Initial specification. Universal entity scope across seven entity types. Supersedes AI-OCO and ECD. |

---

*— End of Specification —*

---

**Standard:** AI-GECO — Grounded Entity Context Object  
**Version:** 1.0  
**Status:** Draft  
**File extension:** `.aigeco.json`
