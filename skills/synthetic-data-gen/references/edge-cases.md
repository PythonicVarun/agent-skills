# Edge Case Taxonomy

Complete reference for edge cases by field type. Use this to audit coverage before finalizing a dataset.

---

## Strings

### General string edge cases

| Case                    | Example                             | Why it matters                                     |
| ----------------------- | ----------------------------------- | -------------------------------------------------- |
| Empty string            | `""`                                | Many validators treat `""` differently from `null` |
| Null / undefined        | `null`                              | Missing vs. empty is a common bug source           |
| Whitespace only         | `"   "`                             | `.trim()` assumptions; DB constraints may allow it |
| Leading/trailing spaces | `" Alice "`                         | Display inconsistency, comparison bugs             |
| Embedded newline        | `"line1\nline2"`                    | CSV/TSV export breakage, textarea rendering        |
| Embedded tab            | `"col1\tcol2"`                      | TSV export, log parsing                            |
| Max-length value        | 255 `a`s                            | Column overflow, truncation bugs, UI overflow      |
| Single character        | `"x"`                               | Off-by-one in length checks                        |
| All uppercase           | `"JOHN DOE"`                        | Case-sensitivity bugs in comparisons               |
| All lowercase           | `"john doe"`                        | Same                                               |
| Mixed case              | `"jOhN dOe"`                        | Normalization issues                               |
| Numeric-looking string  | `"007"`                             | Type coercion bugs (`007 != 7`)                    |
| SQL injection attempt   | `"'; DROP TABLE users;--"`          | Sanitization testing                               |
| HTML/XSS                | `"<script>alert(1)</script>"`       | Output escaping                                    |
| Template literal        | `"Hello ${name}"`                   | Template injection                                 |
| JSON string             | `"{\"key\": \"val\"}"`              | Double-serialization bugs                          |
| Backslash path          | `"C:\\Users\\varun"`                | Escape handling                                    |
| URL encoded             | `"hello%20world"`                   | Decoding consistency                               |
| Emoji                   | `"Great job 🎉"`                    | UTF-8 4-byte handling, DB collation                |
| Arabic/Hebrew (RTL)     | `"مرحبا"`                           | Bidirectional text rendering                       |
| Chinese/Japanese        | `"张伟"`, `"東京"`                  | Multi-byte, collation, length-vs-bytes             |
| Diacritics              | `"José"`, `"Ångström"`, `"Müller"`  | Normalization, sorting                             |
| Quote variants          | `"O'Brien"`, `'single'`, `"double"` | SQL escaping                                       |
| Percent sign            | `"100%"`                            | printf-style format string injection               |
| Null byte               | `"hello\x00world"`                  | C-string termination in some DBs                   |
| Very long word          | 300-char word (no spaces)           | Word-wrap, indexing                                |

### Name fields specifically

- `"Dr. Mary-Jane O'Brien-Walsh III"` - titles, hyphens, apostrophes, suffix
- `"张伟"` - Chinese name (2 chars, no space)
- `"محمد بن سلمان"` - Arabic with spaces
- `"Ólafur Arnalds"` - Icelandic characters
- `"X Æ A-12 Musk"` - unusual but real
- Single-name person: `"Cher"` (no surname)
- Surname only: `""` first name (some cultures)

### Email fields

- Valid: `user@domain.com`
- Subaddressing: `user+tag@domain.com`
- Dots: `first.last@sub.domain.co.uk`
- Hyphenated domain: `user@my-company.io`
- Long TLD: `user@domain.technology`
- Invalid (no @): `notanemail`
- Invalid (no domain): `user@`
- Invalid (spaces): `user @domain.com`
- Internationalized: `用户@例子.广告`

### Phone number fields

- US: `+1-555-867-5309`, `(555) 867-5309`, `5558675309`
- International: `+44 20 7946 0958` (UK), `+81-3-1234-5678` (JP)
- With extension: `+1-555-000-0000 ext. 42`
- Too short: `123`
- Too long: `+999999999999999`
- All zeros: `+1-000-000-0000`

### URL fields

- Valid HTTPS: `https://example.com/path?q=1&lang=en#section`
- HTTP: `http://legacy.example.com`
- With port: `https://api.example.com:8443/v2`
- Very long URL (500+ chars)
- No path: `https://example.com`
- With credentials (deprecated): `https://user:pass@example.com`
- IP address: `http://192.168.1.1/admin`
- Localhost: `http://localhost:3000`
- Invalid: `not-a-url`, `ftp://`, `javascript:void(0)`
- URL-encoded characters: `https://example.com/path%20with%20spaces`

### Address fields

- Full address: `"123 Main St, Apt 4B, Springfield, IL 62701, USA"`
- PO Box: `"PO Box 12345, New York, NY 10001"`
- International: `"10-2 Chiyoda, Tokyo 100-0001, Japan"`
- No apartment: `"42 Elm Street, Boston, MA 02101"`
- Very long street name
- Zip+4: `"12345-6789"`
- Non-numeric postal code: `"SW1A 1AA"` (UK), `"M5V 3A8"` (Canada)

---

## Numbers

### Integer fields

| Case         | Example                                       | Why it matters                       |
| ------------ | --------------------------------------------- | ------------------------------------ |
| Zero         | `0`                                           | Division, comparisons, "none" states |
| Negative     | `-1`, `-100`                                  | Refunds, temperature, debt           |
| One          | `1`                                           | Off-by-one                           |
| Max safe int | `9007199254740991` (JS), `2147483647` (int32) | Overflow                             |
| Min safe int | `-9007199254740991`                           | Underflow                            |
| Very large   | `1000000000`                                  | Display formatting, pagination       |
| null         | `null`                                        | Missing value handling               |

### Float / decimal fields

| Case                 | Example             | Why it matters                  |
| -------------------- | ------------------- | ------------------------------- |
| Zero                 | `0.0`               | Comparisons, division           |
| Negative             | `-0.01`             | Refunds, below-zero temperature |
| Fractional precision | `0.1`, `0.2`, `0.3` | Float precision (0.1+0.2 ≠ 0.3) |
| Currency: rounded    | `10.00`, `99.99`    | Rounding display                |
| Currency: precise    | `19.994`, `0.005`   | Banker's rounding edge case     |
| Very small           | `0.000001`          | Underflow, display precision    |
| Very large           | `999999999.99`      | Formatting, storage type        |
| null                 | `null`              | Missing value                   |

### Percentage / ratio fields

- `0` (zero percent)
- `100` (full / complete)
- `100.0001` (just over 100 - validation check)
- `-0.5` (negative - validation check)
- `50.555...` (repeating decimal)

---

## Booleans

- `true`
- `false`
- `null` (tri-state / unknown)
- String variants if coming from user input: `"true"`, `"1"`, `"yes"`, `"TRUE"` - only relevant if the field is being parsed from text

---

## Dates and Times

### Date fields (date only)

| Case                | Example                                  |
| ------------------- | ---------------------------------------- |
| Today               | `2025-06-25`                             |
| Past (ancient)      | `1899-12-31`                             |
| Unix epoch          | `1970-01-01`                             |
| Y2K                 | `2000-01-01`                             |
| Leap day (4-year)   | `2024-02-29`                             |
| Leap day (400-year) | `2000-02-29`                             |
| Non-leap Feb 28/29  | `2023-02-28` (2023 is not a leap year)   |
| End of month        | `2023-01-31`, `2023-03-31`, `2023-04-30` |
| Far future          | `2099-12-31`                             |
| Near future         | `2025-12-31`                             |
| null                | `null`                                   |

### Datetime / timestamp fields

All of the above plus:
| Case | Example |
|---|---|
| Midnight | `2025-06-25T00:00:00Z` |
| End of day | `2025-06-25T23:59:59Z` |
| UTC | `2025-06-25T12:00:00Z` |
| Positive offset | `2025-06-25T17:30:00+05:30` (India) |
| Negative offset | `2025-06-25T04:00:00-08:00` (US Pacific) |
| DST boundary (spring forward) | `2025-03-09T02:00:00-05:00` |
| Unix epoch | `1970-01-01T00:00:00Z` |
| Far future | `2099-12-31T23:59:59Z` |
| Millisecond precision | `2025-06-25T12:34:56.789Z` |
| Null | `null` |

---

## Enums / Status Fields

- Include **every** valid enum value at least once
- Ensure rare/terminal states appear: `DELETED`, `ARCHIVED`, `BANNED`, `FRAUD`, `EXPIRED`, `REFUNDED`, `PENDING_REVIEW`
- Don't generate equal counts - realistic distributions are skewed

Common enum sets:

```
Order status:     PENDING, PROCESSING, SHIPPED, DELIVERED, CANCELLED, REFUNDED
User status:      ACTIVE, INACTIVE, BANNED, PENDING_VERIFICATION, DELETED
Payment status:   PENDING, AUTHORIZED, CAPTURED, FAILED, REFUNDED, DISPUTED, CHARGEBACK
Subscription:     TRIAL, ACTIVE, PAST_DUE, CANCELLED, PAUSED
Priority:         LOW, MEDIUM, HIGH, CRITICAL
Visibility:       PUBLIC, PRIVATE, UNLISTED, DRAFT
```

---

## Arrays / Lists

| Case                     | Example                        |
| ------------------------ | ------------------------------ |
| Empty                    | `[]`                           |
| Single item              | `["item"]`                     |
| Typical                  | `["a", "b", "c"]`              |
| Large                    | 10-50 items                    |
| Duplicates               | `["a", "a", "b"]` (if allowed) |
| Mixed types (if untyped) | `[1, "two", null, true]`       |
| Null                     | `null` (vs. empty array)       |

---

## IDs

### Auto-increment integer IDs

- Start from 1 (not 0)
- Include gaps (deleted records): IDs jump from 5 to 9
- Include very high IDs (> 10,000) alongside low ones - this catches bugs in OFFSET-based pagination and assumptions about ID ordering

### UUID (v4)

- Include both uppercase and lowercase: `550E8400-E29B-41D4-A716-446655440000` vs. `550e8400-e29b-41d4-a716-446655440000`
- All zeros (nil UUID): `00000000-0000-0000-0000-000000000000`
- Version bits correct: third group starts with `4`, fourth with `8`, `9`, `a`, or `b`

### Composite / slug IDs

- Include hyphens, underscores, case variants
- Include very short (1 char) and very long slugs
- Include slugs with numbers: `product-123`, `user-v2`

---

## Currency / Money

- `0.00` (free)
- `0.01` (1 cent - minimum)
- `0.009` (sub-cent - tests rounding)
- `9.99`, `19.99`, `99.99` (common pricing)
- `1000.00` (round thousands)
- `1234567.89` (large value)
- `-50.00` (refund / credit)
- `null` (no price set / draft)
- Different currency codes if multi-currency: `USD`, `EUR`, `INR`, `JPY` (note: JPY has no cents)

---

## Geographic / Location

- `(0, 0)` - Null Island - catches "default coordinates" bugs
- `(90, 0)` - North Pole
- `(-90, 0)` - South Pole
- `(0, 180)` / `(0, -180)` - Antimeridian
- Negative latitude (southern hemisphere): `-33.8688, 151.2093` (Sydney)
- Negative longitude (western hemisphere): `40.7128, -74.0060` (New York)
- Very precise: 8+ decimal places
- Rounded: `40.7, -74.0`
- Invalid: `lat > 90`, `lng > 180`

---

## Text / Rich Content

- Plain ASCII
- Markdown: `**bold**, _italic_, [link](url), \`code\``
- HTML: `<b>bold</b>`, `<br>`, `<img src="x" onerror="alert(1)">`
- Very short: single word
- Very long: 10,000+ character essay (tests MEDIUMTEXT vs. TEXT vs. VARCHAR limits)
- Code snippet (contains backticks, angle brackets)
- Multiple paragraphs with blank lines
- Bullet list (`- item\n- item`)
- Only whitespace
- Only numbers
- All punctuation

---

## Relationships

| Case                                     | Tests                                               |
| ---------------------------------------- | --------------------------------------------------- |
| Parent with many children (>10)          | N+1 query performance, aggregation                  |
| Parent with exactly one child            | Boundary                                            |
| Parent with zero children                | LEFT JOIN correctness, aggregation returning NULL/0 |
| Child pointing to deleted parent         | Cascade/soft-delete handling                        |
| Circular reference (if trees)            | Infinite loop in recursive queries                  |
| Self-referencing ID (tree parent = self) | Root node detection                                 |
| Deep nesting (depth 5+)                  | Recursive query depth limits                        |
| Orphan record (FK missing from dataset)  | If the dataset is a partial export                  |
