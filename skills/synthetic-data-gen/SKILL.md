---
name: synthetic-data-gen
description: Generate realistic, edge-case-rich synthetic datasets for any domain - perfect for prototyping, testing, and development. Use this skill whenever someone needs to generate fake/mock/dummy/synthetic data, seed a database, create test fixtures, populate a schema with sample records, or simulate realistic data for a prototype or feature. Trigger on phrases like "generate synthetic data", "create test data", "mock data for my API", "seed data for", "sample dataset", "dummy records", "fake data for testing", "realistic test fixtures", "populate this schema", or any request to fill a schema/table/model/form with plausible data. Also trigger when the user shows you a schema, TS interface, Pydantic model, DB migration, or OpenAPI contract and wants data that exercises it - including edge cases. If someone is building a prototype and needs data to show it working, this skill applies.
---

# Synthetic Data Generation

You're generating datasets that are realistic enough to expose real bugs and make prototypes feel production-ready. Two failure modes to avoid:

1. **Too vanilla** - every name is "John Doe", every price is $9.99, every date is today. Bugs hide in homogeneous data.
2. **Too random** - values bear no relationship to each other or their domain. Age -47 with a birthdate in 2089 breaks logic and looks fake.

The goal is **realistic diversity with deliberate anomalies**: data that looks like it came from a real system that's been running long enough to accumulate the weird stuff.

---

## Clarification-First Principle

**One correct output after clarification is always better than multiple wrong ones.**

Before generating anything, assess what you actually know vs. what you'd be guessing at. The rule is simple:

- **If you can infer it with high confidence** (standard field semantics, well-known domain patterns, obvious types) → infer it, state your assumption inline in a brief note, and proceed.
- **If you'd be making it up** (ambiguous business logic, non-obvious constraints, unclear relationships, custom enums you don't know the values of) → **stop and ask**.

### When to ask

Ask upfront - in a single message, never one question at a time - when any of these are true:

- The schema has fields whose **valid values or constraints are domain-specific** and not universally obvious (e.g., `status` with custom states, `type` with proprietary categories, `tier` with business-defined levels)
- There are **FK relationships** but no parent schema was provided - you need the parent IDs to maintain referential integrity
- A field name is **ambiguous** in a way that materially changes generation (e.g., `amount` - is that currency, quantity, or percentage? `date` - event date, created date, or expiry?)
- The user's own **business rules** would govern data validity (e.g., "only premium users can have more than 5 projects") - these cannot be inferred
- The **output format matters** for their workflow and wasn't specified (some formats require exact schema alignment)
- The user explicitly said something like "use my real schema" or "match our actual data" - don't guess what "real" means

### When NOT to ask

Don't ask about things you can reasonably default on:

- Volume (default: 50), output format (default: JSON), locale (default: mixed) - state the default and proceed
- Standard field types on clearly named fields (`email`, `created_at`, `is_active`, `price`) - infer these
- Well-known domain fields - use `references/domain-patterns.md` as the baseline
- Edge cases themselves - always include them; don't ask permission to be thorough

### How to ask

Group all unclear points into **one message**. Never ask one question, wait for an answer, then ask another. Format clearly:

> Before I generate, a few things I need to confirm:
>
> 1. What are the valid values for `status`? (I see it's an enum but your system may have custom states beyond the typical ones.)
> 2. You mentioned `project_id` as a FK - should I also generate the `projects` table, or will you provide those IDs?
> 3. Does `score` have a defined range (e.g., 0-100), or is it unbounded?
>
> Everything else (50 records, JSON output, mixed locale) I'll default to unless you say otherwise.

Once you have answers, generate. Don't ask follow-up questions mid-generation.

### Exception: "just figure it out" mode

If the user says something like "just make something up", "use your best judgment", "surprise me", or gives a very loose spec without caring about fidelity - skip the questions, make reasonable choices, and document every assumption clearly in the output metadata. The user has explicitly opted into inference mode.

---

## Phase 1: Understand the Request

Extract these before generating anything:

1. **Domain** - What is this data for? (e-commerce, healthcare, IoT, HR, social, finance, etc.)
2. **Schema** - Field names, types, constraints. If not provided and not inferrable, ask. If partially provided, infer safe defaults and state them.
3. **Volume** - How many records? Default: 50. State it.
4. **Output format** - JSON, CSV, SQL INSERTs, Python list-of-dicts, Markdown table, TypeScript array, etc. Default: JSON. State it.
5. **Relationships** - Multiple related tables/entities? If FK relationships are implied but parent schema is missing, ask for it.
6. **Locale** - Default: mixed (US + international). State it.
7. **Edge case intensity** - Default: normal (10-20% edge cases). State it.

After this phase, you should have either: (a) enough to generate confidently, or (b) a single consolidated question to ask the user.

---

## Phase 2: Schema Expansion

For each field, resolve:

| Attribute                     | What to determine                                                                                 |
| ----------------------------- | ------------------------------------------------------------------------------------------------- |
| **Type**                      | string / int / float / bool / date / datetime / enum / array / object                             |
| **Cardinality**               | unique (IDs), high-cardinality (names), low-cardinality (status enums), binary (bool)             |
| **Constraints**               | min/max, nullable, regex pattern, FK reference, unique index                                      |
| **Semantic meaning**          | What does this field _represent_? Age ≠ arbitrary int. Email ≠ arbitrary string.                  |
| **Distribution**              | uniform, normal, right-skewed (most orders are small), Zipfian (few popular items)                |
| **Cross-field relationships** | birthdate must precede account creation; `end_date > start_date`; `total = unit_price × quantity` |

For related tables/objects: build the FK dependency graph and generate parents before children.

---

## Phase 3: Edge Case Coverage

Read `references/edge-cases.md` for the full taxonomy. Every dataset must include deliberate coverage of the applicable categories below.

### Mandatory inclusions by type

**String fields**

- Empty string `""` (and `null` if nullable)
- Near-maximum length (long bio, full address block, 255-char description)
- Unicode: names with diacritics (José, 张伟, Müller, Nguyễn), emoji in a text field where plausible
- Parser-breaking characters: `O'Brien`, `"quoted value"`, `<tag>`, `100%`, `user+tag@domain.com`, backslash paths
- Whitespace edge cases: leading/trailing spaces, tab-separated content inside a field, embedded newline

**Numeric fields**

- Zero (`0`) and negative values where domain-valid (refund amount, temperature below zero)
- Boundary values: minimum, maximum, min+1, max-1
- Float precision: values like `0.1`, `19.99`, `1234567.89` - avoid clean round numbers everywhere
- `null` / missing if nullable

**Date/Time fields**

- Historical dates (years past)
- Future dates (scheduled, expiry)
- Epoch: `1970-01-01`, Unix timestamp `0`
- Leap day: `2000-02-29` or `2024-02-29`
- End-of-month: `2023-01-31`, `2023-02-28`
- Timezone-aware if timestamps: include UTC, UTC+5:30, UTC-8 entries
- Far future: `2099-12-31`

**Enum / status fields**

- Every valid enum value must appear at least once
- Deliberately include the "rare" states: `REFUNDED`, `BANNED`, `ARCHIVED`, `PENDING_VERIFICATION`

**Relationships / FK fields**

- One "popular" parent record (many children pointing to it - tests N+1 query paths)
- One "lonely" parent (no children - tests LEFT JOIN correctness)
- If soft-delete exists: children pointing to a deleted parent (tests cascade logic)

**ID fields**

- Mix of small IDs (1, 2, 3) and large IDs (10000+) - catches OFFSET-based pagination bugs
- If UUID: include both UUID v4 format and test that case-insensitive lookups work (one uppercase variant)

**Array / JSON fields**

- Empty array `[]`
- Single-element array
- Large array (10+ items)
- Nested objects if the schema allows

---

## Phase 4: Generation Strategy

### Volume Guidelines

| Use case                       | Count                                                 |
| ------------------------------ | ----------------------------------------------------- |
| Quick UI mockup / demo         | 10-25 rows                                            |
| API / endpoint testing         | 50-100 rows                                           |
| Pagination testing             | 150+ rows (span multiple pages at typical page sizes) |
| Full prototype with edge cases | 50 rows minimum; one row guaranteed per edge case     |
| Load / stress testing          | Provide a **generator script** instead (see Phase 5)  |

### Realism Techniques

**Correlated fields** - If `age` is 8, `occupation` should not be "Senior Director". If `country` is `"JP"`, `phone_prefix` should be `"+81"`. If `plan` is `"free"`, `monthly_spend` should be `0`.

**Temporal consistency** - `created_at < updated_at < deleted_at`. `order_date < shipped_date < delivered_date`. `start_date < end_date`. Never violate these silently.

**Realistic distributions** - Most orders are "completed"; a few are "failed" or "refunded". Don't generate equal counts of every enum. A product catalogue should have a few bestsellers with many sales and many long-tail products with few.

**Name/locale diversity** - Mix genders, cultures, and scripts in name fields. Include some non-ASCII names (José García, Priya Patel, 张伟, Aïcha Diallo). Avoid the "John Smith" monoculture.

**Referential integrity** - FK values must reference actual IDs in the parent collection within this dataset.

**Computed/derived fields** - If `total_price` exists alongside `quantity` and `unit_price`, make them consistent: `total_price = quantity × unit_price` (possibly plus tax).

---

## Phase 5: Output Formats

### JSON (default)

```json
{
  "metadata": {
    "domain": "e-commerce / orders",
    "record_count": 50,
    "generated": "2025-...",
    "edge_cases_included": [
      "empty shipping_address (digital product)",
      "negative total (refund record)",
      "unicode customer name (张伟)",
      "leap-day created_at (2000-02-29)",
      "order with deleted user (soft-delete test)",
      "very long notes field (498 chars)"
    ]
  },
  "data": [...]
}
```

Always include `metadata.edge_cases_included` - this is how the user knows what to test against without hunting through every row.

### CSV

- Header row always included
- Quote fields containing commas, quotes, or newlines
- Represent null as empty cell (or `NULL` if user prefers)
- Escape internal double-quotes as `""`

### SQL INSERTs

- Wrap in a transaction (`BEGIN; ... COMMIT;`)
- Respect FK order: INSERT parents before children
- Escape single quotes in values (`O''Brien`)
- Include `-- edge case: ...` comments on notable rows

### TypeScript / Python literal

- Typed array with inline objects
- Respect the interface/model from the user's codebase if provided

### Generator Script (for >500 rows or load testing)

Read `references/generation-scripts.md` for ready-to-use templates.

A generator script is more useful than a static blob when volume is high - it's reproducible, seedable, and the user can tune distributions. Provide one proactively for load-test requests.

---

## Phase 6: Self-Validation Checklist

Before presenting output, verify:

- [ ] Every enum value appears at least once
- [ ] At least one null/empty per nullable field
- [ ] No temporal inconsistencies (start > end, created > updated)
- [ ] FK references resolve to actual IDs in this dataset
- [ ] At least one record per major applicable edge case category
- [ ] Unicode/special-character records present
- [ ] Distribution is realistic (not uniform across all enums)
- [ ] Computed fields are internally consistent

Fix silently before outputting - don't narrate each fix.

---

## Phase 7: User Communication

After the data, add a short summary:

> **Edge cases included:** empty `bio` (2 records), unicode names (José García, 张伟), leap-day `created_at` (2000-02-29), `price = 0.00` (free tier), 498-char `description`, an order referencing a soft-deleted user (#1047), negative `balance` (overdraft), and an `orders` array with 0 items (new account).

This tells the user exactly what to test without them needing to grep through the data.

If the user asked for "normal" data but the domain has well-known hazards they didn't mention (e.g., SQL injection strings in a user-input field, or currency precision in a fintech app), note these and offer to include them.

---

## Reference Files

Load these when relevant - don't load all of them for every request:

- **`references/edge-cases.md`** - Full edge case taxonomy by field type. Load when you need a comprehensive checklist for a specific field type (phone numbers, URLs, addresses, etc.) or want to audit coverage.
- **`references/domain-patterns.md`** - Pre-built field sets + edge cases for common domains (e-commerce, HR, IoT, healthcare, finance, social, geospatial). Load when the user names a well-known domain - it'll save you from re-deriving the obvious fields.
- **`references/generation-scripts.md`** - Python and JS script templates using `faker`, `factory_boy`, and plain random. Load when volume > 500 or the user wants a reusable generator.
