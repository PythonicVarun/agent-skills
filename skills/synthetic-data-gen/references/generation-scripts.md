# Generation Script Templates

Use these when volume > 500 rows or the user wants a reusable, seedable generator they can run locally.

---

## Python - `faker` + pure stdlib

Best for: quick, zero-config generation. `faker` handles locale-aware names, addresses, dates, etc.

```python
# /// script
# requires-python = ">=3.12"
# dependencies = [
#     "faker",
# ]
# ///
"""
Synthetic data generator - customize SCHEMA and EDGE_CASES below.
Usage: uv run generate.py --count 1000 --seed 42 --format json > data.json
"""

import json
import random
import argparse
from datetime import datetime, timedelta, timezone
from faker import Faker

fake = Faker(["en_US", "en_GB", "ja_JP", "de_DE", "zh_CN", "ar_EG"])

# ─── Configure here ──────────────────────────────────────────────────────────

STATUSES = ["active", "inactive", "banned", "pending_verification"]
STATUS_WEIGHTS = [0.75, 0.15, 0.05, 0.05]  # realistic distribution

# ─────────────────────────────────────────────────────────────────────────────


def random_date(start_years_ago: int = 5, end_years_ago: int = 0) -> str:
    start = datetime.now(timezone.utc) - timedelta(days=start_years_ago * 365)
    end = datetime.now(timezone.utc) - timedelta(days=end_years_ago * 365)
    delta = end - start
    return (start + timedelta(seconds=random.randint(0, int(delta.total_seconds())))).isoformat()


def make_user(i: int, force_edge: str | None = None) -> dict:
    """Generate one user record. force_edge injects a specific edge case."""

    if force_edge == "empty_name":
        first, last = "", ""
    elif force_edge == "unicode_name":
        first, last = random.choice(["张", "محمد", "Ólafur", "Nguyễn"]), fake.last_name()
    elif force_edge == "long_bio":
        first, last = fake.first_name(), fake.last_name()
    elif force_edge == "null_phone":
        first, last = fake.first_name(), fake.last_name()
    else:
        first, last = fake.first_name(), fake.last_name()

    bio = fake.text(max_nb_chars=2000) if force_edge == "long_bio" else (
        None if random.random() < 0.15 else fake.sentence()
    )

    phone = None if (force_edge == "null_phone" or random.random() < 0.1) else fake.phone_number()

    created = random_date(start_years_ago=10, end_years_ago=0)
    updated = random_date(start_years_ago=1, end_years_ago=0)
    if updated < created:
        updated = created  # ensure consistency

    return {
        "id": i,
        "email": fake.email() if force_edge != "empty_name" else f"user{i}@example.com",
        "first_name": first,
        "last_name": last,
        "phone": phone,
        "bio": bio,
        "status": random.choices(STATUSES, STATUS_WEIGHTS)[0] if not force_edge else (
            "banned" if force_edge == "banned_user" else random.choices(STATUSES, STATUS_WEIGHTS)[0]
        ),
        "created_at": created,
        "updated_at": updated,
    }


def generate(count: int, seed: int) -> list[dict]:
    random.seed(seed)
    Faker.seed(seed)

    records = []

    # --- Guaranteed edge cases (injected first) ---
    edge_cases = [
        ("empty_name", "Guaranteed edge case: empty name fields"),
        ("unicode_name", "Guaranteed edge case: non-ASCII name"),
        ("long_bio", "Guaranteed edge case: max-length bio"),
        ("null_phone", "Guaranteed edge case: null phone"),
        ("banned_user", "Guaranteed edge case: banned status"),
    ]

    for i, (edge_type, _comment) in enumerate(edge_cases, start=1):
        rec = make_user(i, force_edge=edge_type)
        rec["_edge_case"] = edge_type  # helpful during debugging; strip before prod
        records.append(rec)

    # --- Random records to fill the rest ---
    for i in range(len(edge_cases) + 1, count + 1):
        records.append(make_user(i))

    return records


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--count", type=int, default=100)
    parser.add_argument("--seed", type=int, default=42)
    parser.add_argument("--format", choices=["json", "csv", "jsonl"], default="json")
    args = parser.parse_args()

    data = generate(args.count, args.seed)

    if args.format == "json":
        print(json.dumps({
            "metadata": {
                "record_count": len(data),
                "seed": args.seed,
                "edge_cases_included": ["empty_name", "unicode_name", "long_bio", "null_phone", "banned_user"],
            },
            "data": data,
        }, indent=2, ensure_ascii=False))

    elif args.format == "jsonl":
        for row in data:
            print(json.dumps(row, ensure_ascii=False))

    elif args.format == "csv":
        import csv, sys
        writer = csv.DictWriter(sys.stdout, fieldnames=data[0].keys())
        writer.writeheader()
        writer.writerows(data)
```

---

## Python - `factory_boy` (for Django / SQLAlchemy models)

Best for: projects that already use Django ORM or SQLAlchemy - generates records directly into the DB.

```python
# /// script
# requires-python = ">=3.12"
# dependencies = [
#     "factory_boy",
#     "faker",
# ]
# ///
"""
Usage: uv run generate.py
"""

import factory
from factory import fuzzy
from faker import Faker
from datetime import datetime, timezone
import random

fake = Faker()

# Replace `MyApp.models` with your actual import
# from myapp.models import User, Order


class UserFactory(factory.DjangoModelFactory):
    """
    Usage:
        UserFactory()                          # 1 user, random data
        UserFactory.create_batch(50)           # 50 users
        UserFactory(status="banned")           # override a field
        UserFactory(first_name="", last_name="")  # inject edge case
    """

    class Meta:
        model = "myapp.User"  # replace with your model

    id = factory.Sequence(lambda n: n + 1)
    email = factory.LazyAttribute(lambda obj: f"{obj.first_name.lower()}.{obj.last_name.lower()}@example.com")
    first_name = factory.Faker("first_name")
    last_name = factory.Faker("last_name")
    phone = factory.Maybe(
        factory.LazyFunction(lambda: random.random() > 0.1),
        yes_declaration=factory.Faker("phone_number"),
        no_declaration=None,
    )
    bio = factory.Maybe(
        factory.LazyFunction(lambda: random.random() > 0.15),
        yes_declaration=factory.Faker("sentence"),
        no_declaration=None,
    )
    status = fuzzy.FuzzyChoice(
        ["active", "inactive", "banned", "pending_verification"],
        # weights not natively supported; use LazyFunction for distributions
    )
    created_at = factory.Faker("date_time_between", start_date="-10y", end_date="-1d", tzinfo=timezone.utc)
    updated_at = factory.LazyAttribute(lambda obj: obj.created_at)


# --- Guaranteed edge case traits ---

class EmptyNameUserFactory(UserFactory):
    first_name = ""
    last_name = ""


class UnicodeUserFactory(UserFactory):
    first_name = factory.LazyFunction(lambda: random.choice(["张伟", "محمد", "Ólafur", "José"]))


class BannedUserFactory(UserFactory):
    status = "banned"


class LongBioUserFactory(UserFactory):
    bio = factory.Faker("text", max_nb_chars=2000)


# --- Seeding a database ---

def seed_db(count: int = 100):
    """Generate a realistic mix with guaranteed edge cases."""
    # Edge cases first
    EmptyNameUserFactory()
    UnicodeUserFactory()
    BannedUserFactory()
    LongBioUserFactory()
    UserFactory(phone=None)

    # Random remainder
    UserFactory.create_batch(count - 5)
    print(f"Seeded {count} users (5 edge cases + {count - 5} random)")
```

---

## JavaScript / TypeScript - pure stdlib (no dependencies)

Best for: Node.js projects, seeding a local dev DB, generating fixtures for Jest/Vitest.

```typescript
// generate.ts - run with: npx ts-node generate.ts
// or: node --experimental-strip-types generate.ts (Node 22+)

import { writeFileSync } from "fs";

// ── Helpers ──────────────────────────────────────────────────────────────────

let seed = 42;
function rand(): number {
    // Simple seeded LCG - replace with `crypto.randomUUID()` if you don't need reproducibility
    seed = (seed * 1664525 + 1013904223) & 0xffffffff;
    return (seed >>> 0) / 0xffffffff;
}

function pick<T>(arr: T[]): T {
    return arr[Math.floor(rand() * arr.length)];
}

function pickWeighted<T>(items: T[], weights: number[]): T {
    const total = weights.reduce((a, b) => a + b, 0);
    let r = rand() * total;
    for (let i = 0; i < items.length; i++) {
        r -= weights[i];
        if (r <= 0) return items[i];
    }
    return items[items.length - 1];
}

function randomDate(yearsAgo: number = 5): string {
    const ms = Date.now() - rand() * yearsAgo * 365 * 24 * 60 * 60 * 1000;
    return new Date(ms).toISOString();
}

function randomId(): string {
    return "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(/[xy]/g, (c) => {
        const r = Math.floor(rand() * 16);
        return (c === "x" ? r : (r & 0x3) | 0x8).toString(16);
    });
}

// ── Types ─────────────────────────────────────────────────────────────────────

type UserStatus = "active" | "inactive" | "banned" | "pending_verification";

interface User {
    id: number;
    uuid: string;
    email: string;
    first_name: string;
    last_name: string;
    phone: string | null;
    bio: string | null;
    status: UserStatus;
    created_at: string;
    updated_at: string;
    _edge_case?: string;
}

// ── Pools ─────────────────────────────────────────────────────────────────────

const FIRST_NAMES = [
    "Alice",
    "Bob",
    "Priya",
    "José",
    "张伟",
    "Nguyễn Văn",
    "Aïcha",
    "Ólafur",
    "María",
    "James",
];
const LAST_NAMES = [
    "Smith",
    "García",
    "Patel",
    "O'Brien",
    "Müller",
    "Nakamura",
    "Diallo",
    "Silva",
    "Kim",
    "Johnson",
];
const DOMAINS = [
    "gmail.com",
    "yahoo.com",
    "outlook.com",
    "company.io",
    "example.org",
];

function makeUser(id: number, edgeCase?: string): User {
    const firstName = edgeCase === "empty_name" ? "" : pick(FIRST_NAMES);
    const lastName = edgeCase === "empty_name" ? "" : pick(LAST_NAMES);
    const email = `${firstName.toLowerCase().replace(/\s+/g, ".") || "user"}.${id}@${pick(DOMAINS)}`;

    const status: UserStatus =
        edgeCase === "banned"
            ? "banned"
            : pickWeighted(
                  ["active", "inactive", "banned", "pending_verification"],
                  [75, 15, 5, 5],
              );

    const created = randomDate(10);
    const updated = new Date(
        new Date(created).getTime() + rand() * 365 * 24 * 60 * 60 * 1000,
    ).toISOString();

    return {
        id,
        uuid: randomId(),
        email,
        first_name: firstName,
        last_name: lastName,
        phone:
            edgeCase === "null_phone" || rand() < 0.1
                ? null
                : `+1-${Math.floor(rand() * 900 + 100)}-${Math.floor(rand() * 900 + 100)}-${Math.floor(rand() * 9000 + 1000)}`,
        bio:
            edgeCase === "long_bio"
                ? "Lorem ipsum ".repeat(200).trim()
                : rand() < 0.15
                  ? null
                  : "Some short bio text.",
        status,
        created_at: created,
        updated_at: updated,
        ...(edgeCase ? { _edge_case: edgeCase } : {}),
    };
}

// ── Generate ──────────────────────────────────────────────────────────────────

function generate(count: number): User[] {
    const edgeCases: Array<[string, string]> = [
        ["empty_name", "empty first/last name"],
        ["unicode_name_in_pool", "non-ASCII name (already in pool)"],
        ["long_bio", "2400-char bio"],
        ["null_phone", "null phone field"],
        ["banned", "banned status"],
    ];

    const users: User[] = edgeCases.map(([ec], i) => makeUser(i + 1, ec));

    for (let i = edgeCases.length + 1; i <= count; i++) {
        users.push(makeUser(i));
    }

    return users;
}

// ── Output ────────────────────────────────────────────────────────────────────

const COUNT = 100;
const output = {
    metadata: {
        record_count: COUNT,
        seed: 42,
        edge_cases_included: [
            "empty_name",
            "unicode_names",
            "long_bio",
            "null_phone",
            "banned_status",
        ],
    },
    data: generate(COUNT),
};

writeFileSync("synthetic_users.json", JSON.stringify(output, null, 2));
console.log(`Generated ${COUNT} users → synthetic_users.json`);
```

---

## SQL INSERT Generator (Python → SQL)

Best for: seeding a relational database directly.

```python
# /// script
# requires-python = ">=3.12"
# dependencies = [
#     "faker",
# ]
# ///
"""
Generates SQL INSERT statements with proper escaping and FK ordering.
Outputs a .sql file ready to run against Postgres or MySQL.

Usage: uv run generate.py
"""

import random
from faker import Faker

fake = Faker()
random.seed(42)
Faker.seed(42)


def esc(val) -> str:
    """Escape a value for SQL."""
    if val is None:
        return "NULL"
    if isinstance(val, bool):
        return "TRUE" if val else "FALSE"
    if isinstance(val, (int, float)):
        return str(val)
    # Escape single quotes by doubling them
    return "'" + str(val).replace("'", "''") + "'"


def gen_users(n: int) -> list[dict]:
    rows = []
    statuses = ["active"] * 75 + ["inactive"] * 15 + ["banned"] * 5 + ["pending_verification"] * 5

    # Edge cases
    rows.append({"id": 1, "email": "empty@example.com", "first_name": "", "last_name": "", "status": "active", "bio": None})
    rows.append({"id": 2, "email": "unicode@example.com", "first_name": "张伟", "last_name": "García", "status": "active", "bio": "Hello 世界"})
    rows.append({"id": 3, "email": "banned@example.com", "first_name": "Bob", "last_name": "Smith", "status": "banned", "bio": None})

    for i in range(4, n + 1):
        rows.append({
            "id": i,
            "email": fake.email(),
            "first_name": fake.first_name(),
            "last_name": fake.last_name(),
            "status": random.choice(statuses),
            "bio": None if random.random() < 0.15 else fake.sentence(),
        })

    return rows


def gen_orders(users: list[dict], n: int) -> list[dict]:
    rows = []
    active_user_ids = [u["id"] for u in users if u["status"] == "active"]
    all_user_ids = [u["id"] for u in users]
    statuses = ["completed"] * 60 + ["pending"] * 20 + ["cancelled"] * 10 + ["refunded"] * 10

    # Edge cases
    rows.append({"id": 1, "user_id": active_user_ids[0], "total": 0.00, "status": "completed", "notes": "100% coupon applied"})
    rows.append({"id": 2, "user_id": 3, "total": -25.50, "status": "refunded", "notes": "Refund issued"})  # banned user

    for i in range(3, n + 1):
        uid = random.choice(active_user_ids) if random.random() > 0.05 else random.choice(all_user_ids)
        rows.append({
            "id": i,
            "user_id": uid,
            "total": round(random.uniform(0.99, 999.99), 2),
            "status": random.choice(statuses),
            "notes": None,
        })

    return rows


def to_sql(table: str, rows: list[dict]) -> str:
    if not rows:
        return ""
    cols = ", ".join(rows[0].keys())
    lines = [f"-- {table} ({len(rows)} rows)"]
    for row in rows:
        vals = ", ".join(esc(v) for v in row.values())
        lines.append(f"INSERT INTO {table} ({cols}) VALUES ({vals});")
    return "\n".join(lines)


if __name__ == "__main__":
    users = gen_users(50)
    orders = gen_orders(users, 100)

    sql = "\n".join([
        "BEGIN;",
        "",
        to_sql("users", users),
        "",
        to_sql("orders", orders),
        "",
        "COMMIT;",
    ])

    with open("seed.sql", "w", encoding="utf-8") as f:
        f.write(sql)

    print(f"Generated seed.sql ({len(users)} users, {len(orders)} orders)")
```

---

## Quick Tips

**Reproducibility**: Always accept a `--seed` parameter and seed both `random.seed()` and `Faker.seed()` at the top. This lets users reproduce the exact same dataset for debugging.

**Stripping debug fields**: Fields like `_edge_case` are useful during development. Add a `--strip-debug` flag to remove them before exporting to production fixtures.

**Multiple related tables**: Generate in FK dependency order. Put all generators in one file and call them sequentially. Pass parent IDs down to child generators explicitly rather than using global state.

**Locale mixing**: `Faker(["en_US", "ja_JP", "de_DE", "ar_EG"])` automatically picks a random locale per call, giving you diverse names and addresses without extra code.

**Large volumes without memory issues**: Use `yield` (Python generator) or streams (Node.js) for > 100k rows. Don't build the entire list in memory.
