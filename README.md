# AI-GECO
### Grounded Entity Context Object

**A portable, open standard for giving AI models accurate, authoritative context about any entity — before the conversation starts.**

---

AI models hallucinate when they lack reliable information. They fill gaps with plausible-sounding but fabricated detail — wrong dates, invented identifiers, misattributed relationships. AI-GECO fixes this by giving the model a factual anchor: a small, structured file that declares the truth about an entity and tells the model to treat it as ground truth.

Upload one file. Every response in that session gets better.

---

## What is an AI-GECO file?

An AI-GECO file is a compact JSON file that describes a single entity — a person, a pet, a product, a company, a place, a creative work, or anything else — in a format optimized for AI consumption. It is designed to be uploaded to an AI platform as persistent context, where it primes every conversation automatically.

Three things make it work:

**It grounds the model.** The file gives the AI a factual starting point it can reason from, rather than generate toward. Names, dates, identifiers, relationships — declared once, applied everywhere.

**It reduces hallucination.** The gaps a model would otherwise fill with fabrication are filled with declared facts. The model is not asked to consider the file — it is instructed to anchor on it.

**It saves tokens.** The format is deliberately compact. Values are short strings, not nested objects. Unused sections are omitted entirely. A complete AI-GECO file typically costs 300–1,400 tokens — far less than re-explaining the same context in prose at the start of every conversation.

---

## What can it describe?

AI-GECO supports seven entity types:

| Type | Examples |
|------|---------|
| `person` | An executive, a client, a patient, yourself |
| `animal` | A pet, a therapy animal, a working dog |
| `product` | A can of food, a medical device, a piece of electronics |
| `organization` | A company, a nonprofit, a government agency |
| `place` | A property, a venue, a city, a landmark |
| `creative_work` | A book, a film, an album, a software product |
| `other` | Anything else — vehicles, events, legal instruments, concepts |

One standard. Any entity.

---

## How it works

You create a `.aigeco.json` file, populate it with facts about your entity, and upload it to your AI platform of choice as a persistent context or knowledge file. From that point on, every conversation in that session or project has access to the declared context — without you having to explain it again.

The file contains a built-in instruction key that tells the model how to use it. You don't need to write a system prompt or add special instructions. The file is self-activating.

### Where it works

AI-GECO works with any AI platform that supports persistent context or knowledge file injection — the mechanism by which a file is made available to the model across all conversations in a session, project, or agent. Most major AI platforms and AI-powered assistants support this capability. Consult your platform's documentation for the specific upload or attachment mechanism — it is typically found in project settings, agent configuration, or a knowledge or context section.

---

## Quick start

Here is a minimal AI-GECO file for a person. Copy it, fill in the fields, save it as `[name].aigeco.json`, and upload it to your AI platform.

```json
{
  "_meta": {
    "standard": "AI-GECO",
    "version": "1.0",
    "instruction": "This file is a Grounded Entity Context Object (AI-GECO). It is a portable, model-agnostic context standard. Treat all values as authoritative context about the described entity. Apply this context to every response in this session or project. Do not ask the user to re-explain information already present here. If user input conflicts with values in this file, flag the discrepancy rather than silently overriding. Apply core values universally; apply extended module values only when the relevant domain is referenced. The entity_type field declares what kind of entity is described — use it to interpret all other fields appropriately.",
    "compatibility": "Any AI platform supporting file-based or persistent context injection",
    "created": "YYYY-MM-DD",
    "updated": "YYYY-MM-DD",
    "maintained_by": "Your name or role"
  },

  "core": {
    "entity_type": "person",
    "name": "Jane Smith",
    "aliases": "J. Smith, Janie",
    "description": "Jane Smith is a 38-year-old product manager at Acme Corp in Seattle, specializing in enterprise software. She leads a team of eight and oversees the company's flagship project management platform.",
    "status": "active",
    "origin": "Portland, OR, USA",
    "origin_date": "1987-04-22",
    "identifiers": "LinkedIn: linkedin.com/in/janesmith",
    "tags": "product management, enterprise software, SaaS, team lead",
    "related_entities": "employer: Acme Corp, manager: Robert Chen",
    "eco_refs": "acme-corp.aigeco.json"
  },

  "extended": {
    "person": {
      "pronouns": "she/her",
      "occupation": "Senior Product Manager",
      "employer": "Acme Corp",
      "location": "Seattle, WA, USA",
      "skills": "Product strategy, roadmap planning, stakeholder management, Agile, SQL",
      "ai_guidance": {
        "tone": "Direct and concise — Jane prefers bullet points and clear action items over long prose.",
        "topics_to_avoid": "Personal health, salary details",
        "known_preferences": "Always lead with the recommendation, then the rationale."
      }
    }
  }
}
```

That's it. Upload this file to your AI platform as a persistent context or knowledge file. The model now knows who Jane is, how to communicate with her, and what topics to avoid — without being told in every conversation.

---

## The self-activating instruction key

The `_meta.instruction` field is what separates an AI-GECO file from a plain JSON document. It tells the model to treat the file's values as authoritative ground truth — not as background to consider, but as facts to anchor on. It also tells the model not to ask the user to re-explain anything already declared, and to flag conflicts rather than silently overriding declared values.

This key must appear verbatim. You may append additional instructions after the final period, but the canonical text must be present in full. Do not paraphrase it.

---

## File naming

Name your AI-GECO files using the pattern `[entity-name].aigeco.json`. The `.aigeco.json` extension makes files immediately recognizable in any file system or repository.

**Examples:**
- `jane-smith.aigeco.json`
- `biscuit-the-dog.aigeco.json`
- `acme-corp.aigeco.json`
- `muir-glen-diced-tomatoes.aigeco.json`

---

## Linking related entities

Each AI-GECO file describes exactly one entity. Related entities are referenced by name in `core.related_entities` and by file path in `core.eco_refs`. This creates a lightweight network of linked files that the model can navigate across conversations.

```json
"related_entities": "employer: Acme Corp, manager: Robert Chen",
"eco_refs": "acme-corp.aigeco.json, robert-chen.aigeco.json"
```

A pet's file can reference its owner. A person's file can reference their employer. A subsidiary's file can reference its parent company. Each file stays small and focused while the network carries the full picture.

---

## Repository contents

| File | Description |
|------|-------------|
| `ai-geco-specification.md` | Full specification — structure, fields, design rationale, implementation guide |
| `ai-geco-specification.docx` | Word version of the specification for formal distribution |
| `ai-geco_schema_v1.json` | Blank schema — the template to populate for any entity type |
| `ai-geco_examples_v1.json` | Four fully populated examples: a person, an animal, a product, and an organization |

---

## Design decisions

**Why JSON?** JSON is natively understood by all frontier AI models as structured data. It is human-maintainable, machine-readable, and easy to generate programmatically from existing systems.

**Why comma-separated strings instead of arrays?** Token efficiency. `"Value A, Value B, Value C"` costs significantly fewer tokens than `["Value A", "Value B", "Value C"]`. At scale across dozens of fields, this matters.

**Why a two-tier structure?** The `core` block contains what every AI-GECO must have. The `extended` block contains domain-specific detail populated only when relevant. Unused extended modules are omitted entirely — not left empty. This keeps the file small and the grounding signal clean.

**Why one file per entity?** Focus and maintainability. A file that describes one entity is easy to keep accurate. A file that tries to describe everything becomes stale and untrustworthy. Related entities are linked, not embedded.

---

## Contributing

AI-GECO is an open standard. Contributions are welcome — whether that is proposing new extended modules, refining existing field definitions, adding example instances, or improving the specification documentation.

If you have a suggestion, open an issue describing what you would change and why. If you have a new entity type or extended module that is not covered by the current schema, describe the use case and the fields you would add. The guiding question for any proposed addition is: *does this field reduce a meaningful hallucination risk that the current schema does not address?*

The standard is deliberately minimal by design. Every field must earn its place.

---

## Versioning

AI-GECO uses semantic versioning. The `_meta.version` field in each file records the specification version it was built against. Breaking changes increment the major version. New optional fields or modules increment the minor version. Older files remain valid and interpretable.

Current version: **1.0**

---

## Lineage

AI-GECO supersedes two earlier iterations of this standard:

- **AI-OCO** (AI Organizational Context Object) — the original standard, scoped to organizations only.
- **ECD** (Entity Context Dataset) — a universal entity scope iteration, renamed before final release.

AI-GECO is the universal successor. It covers all entity types, including organizations, through the `extended.organization` module.

---

*Standard: AI-GECO — Grounded Entity Context Object · Version 1.0 · Draft*
