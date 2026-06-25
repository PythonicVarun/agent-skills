# Domain Patterns

Pre-built field sets and edge case considerations for common domains. Use these as a starting point and tailor to the user's specific schema.

---

## E-Commerce

### Users / Customers

```
id, email, first_name, last_name, phone, date_of_birth,
created_at, updated_at, deleted_at (soft delete),
status (ACTIVE|INACTIVE|BANNED), avatar_url,
shipping_address (object), billing_address (object),
locale, timezone, currency_preference
```

**Key edge cases:**

- Guest account (no password, checkout-only)
- User registered but never ordered
- User with 50+ orders (VIP)
- Banned user still has historical orders
- International address (non-US postal code, non-ASCII name)
- User with same shipping + billing address
- User with no address (digital-only buyer)
- Account age > 10 years (old user)
- Multiple payment methods

### Products / Catalog

```
id, sku, name, description, category_id, brand,
price, compare_at_price (was-price), cost_price,
currency, tax_class,
inventory_count, low_stock_threshold, allow_backorder,
status (DRAFT|ACTIVE|ARCHIVED|OUT_OF_STOCK),
weight, dimensions (object), images (array),
tags (array), attributes (object: color/size/etc.),
created_at, published_at, updated_at
```

**Key edge cases:**

- Price = `0.00` (free product, digital giveaway)
- `price < cost_price` (loss-leader or pricing error)
- `inventory_count = 0` with `allow_backorder = true`
- `compare_at_price = null` (no sale)
- Product with 0 images (just-created draft)
- Very long description (2000+ chars)
- Product with 20+ tags
- Archived product that still has pending orders
- SKU with special chars: `PROD-X/2024_v2`

### Orders

```
id, order_number (human-readable), user_id, guest_email,
status (PENDING|PROCESSING|SHIPPED|DELIVERED|CANCELLED|REFUNDED),
line_items (array of: product_id, variant_id, quantity, unit_price, total_price),
subtotal, discount_amount, tax_amount, shipping_amount, total_amount,
payment_method, payment_status (PENDING|CAPTURED|FAILED|REFUNDED),
shipping_address, billing_address,
placed_at, confirmed_at, shipped_at, delivered_at, cancelled_at,
tracking_number, notes
```

**Key edge cases:**

- Order total = `0.00` (100% discount coupon)
- Negative total (refund/credit order)
- Order with 1 item vs. order with 30+ items
- Cancelled order (`cancelled_at` set, other timestamps null)
- Refunded order with original + refund records
- Guest checkout (`user_id = null`, `guest_email` set)
- Order with no shipping (digital product)
- `total != subtotal + tax + shipping - discount` (intentional bug-finder)
- Very old order (3+ years ago)
- Order placed at midnight (timezone edge)

### Reviews

```
id, product_id, user_id, rating (1-5), title, body,
verified_purchase, helpful_count, status (PENDING|APPROVED|REJECTED|SPAM),
created_at, updated_at
```

**Key edge cases:**

- Rating = 1 (most negative) and rating = 5 (most positive)
- Empty body (title only)
- Very long review (2000+ chars)
- Non-verified purchase review
- Review from a now-banned user
- `helpful_count = 0` and `helpful_count = 500`

---

## Finance / Banking

### Accounts

```
id, account_number, user_id, type (CHECKING|SAVINGS|CREDIT|LOAN),
status (ACTIVE|FROZEN|CLOSED|SUSPENDED),
currency, balance, available_balance, credit_limit (credit only),
opened_at, closed_at, last_activity_at
```

**Key edge cases:**

- `balance = 0.00` (empty account)
- `balance < 0` (overdraft)
- `balance = credit_limit` (maxed out)
- Frozen account with non-zero balance
- Closed account (non-null `closed_at`)
- `available_balance < balance` (pending holds)
- Very large balance (> $1,000,000)
- Old account (opened 20+ years ago)

### Transactions

```
id, account_id, type (DEBIT|CREDIT|TRANSFER|FEE|INTEREST|REFUND),
amount, currency, balance_after,
description, merchant_name, merchant_category,
reference_id, external_id,
status (PENDING|SETTLED|FAILED|REVERSED),
initiated_at, settled_at, reversed_at
```

**Key edge cases:**

- `amount = 0.01` (1 cent - smallest possible)
- `amount` with 4 decimal places (sub-cent during conversion)
- `amount` very large (wire transfer > $100,000)
- Negative amount (reversal or fee)
- Pending transaction (`settled_at = null`)
- Reversed transaction (has `reversed_at`, paired reversal record)
- Failed transaction (never settled)
- Currency different from account currency (FX transaction)
- Multiple transactions in exact same second
- Description with special characters (`ATM W/D "MAIN ST" & FEES`)

---

## Healthcare

### Patients

```
id, mrn (medical record number), first_name, last_name, dob,
sex (M|F|O|U), gender_identity,
blood_type, allergies (array), medications (array),
primary_physician_id,
insurance_provider, insurance_id, insurance_group,
emergency_contact (object),
created_at, updated_at
```

**Key edge cases:**

- Newborn (dob = recent date)
- Elderly patient (dob 90+ years ago)
- No insurance (uninsured)
- Multiple allergies (10+)
- Unknown blood type
- Gender identity different from sex
- Same name as another patient (deduplication test)
- Patient with no physician assigned

### Appointments

```
id, patient_id, provider_id, facility_id,
type (INITIAL|FOLLOWUP|URGENT|TELEMEDICINE|PROCEDURE),
status (SCHEDULED|CONFIRMED|CHECKED_IN|IN_PROGRESS|COMPLETED|NO_SHOW|CANCELLED),
scheduled_at, started_at, ended_at, duration_minutes,
chief_complaint, notes, diagnosis_codes (array),
created_at
```

**Key edge cases:**

- Appointment in the past (historical)
- Appointment in the future (upcoming)
- `status = NO_SHOW` (patient never arrived)
- `duration_minutes` far exceeding scheduled slot
- Emergency/same-day appointment
- Telemedicine appointment (no physical location)
- Cancelled appointment (no started_at)
- Multiple appointments same day for same patient

---

## Human Resources / Employee

### Employees

```
id, employee_id (company-assigned), first_name, last_name, email,
personal_email, phone,
department_id, team_id, manager_id (self-ref FK),
job_title, level (IC1-IC6, M1-M5, etc.),
employment_type (FULL_TIME|PART_TIME|CONTRACT|INTERN),
status (ACTIVE|ON_LEAVE|TERMINATED|SUSPENDED),
hire_date, termination_date, rehire_date,
salary, pay_frequency (HOURLY|BIWEEKLY|MONTHLY|ANNUAL),
office_location, remote_work_eligible,
created_at, updated_at
```

**Key edge cases:**

- Employee who is their own manager (root of org chart)
- Terminated employee (non-null `termination_date`)
- Rehired employee (second hire_date later)
- Contract worker (no salary, has hourly rate)
- Intern (short tenure, low level)
- Employee on leave (status = ON_LEAVE, no termination_date)
- Employee with no direct reports
- Manager with 20+ direct reports
- Employee with missing phone (optional field)
- Two employees with same name in different departments

---

## IoT / Sensor Data

### Devices

```
id, device_id (hardware serial), name, type (THERMOSTAT|CAMERA|SENSOR|ACTUATOR),
firmware_version, hardware_version,
status (ONLINE|OFFLINE|ERROR|MAINTENANCE),
location (object: lat/lng/floor/room),
owner_id, site_id,
last_seen_at, registered_at, decommissioned_at
```

**Key edge cases:**

- Device never seen online (`last_seen_at = null`)
- Device in ERROR state
- Decommissioned device (has `decommissioned_at`)
- Device with very old firmware
- Multiple devices at same location
- Device with null/unknown location

### Sensor Readings

```
id, device_id, sensor_type (TEMPERATURE|HUMIDITY|PRESSURE|MOTION|CO2|LIGHT),
value, unit, quality (GOOD|UNCERTAIN|BAD),
timestamp, received_at
```

**Key edge cases:**

- `value` at sensor minimum (e.g., temperature = -40°C)
- `value` at sensor maximum (e.g., temperature = 85°C)
- `quality = BAD` (sensor malfunction)
- `quality = UNCERTAIN` (borderline reading)
- Duplicate readings (same device, same timestamp)
- Out-of-order timestamps (`received_at` much later than `timestamp`)
- Null value (sensor offline, quality = BAD)
- Very high frequency burst (many readings in 1 second)
- Gap in readings (device offline period)

---

## Social / Content Platform

### Users

```
id, username, display_name, bio, avatar_url, website_url,
email, email_verified,
follower_count, following_count, post_count,
status (ACTIVE|SUSPENDED|DEACTIVATED|PRIVATE),
verified (boolean), created_at, last_active_at
```

**Key edge cases:**

- New user (0 followers, 0 posts)
- Viral user (10M+ followers)
- Suspended user with historical posts
- Private account
- Verified badge
- User with no bio
- Username with allowed special chars: `user.name`, `user_123`
- Very long display name
- User who has never posted

### Posts / Content

```
id, author_id, content, media (array of URLs),
parent_id (for replies/reposts), repost_of_id,
like_count, repost_count, reply_count, view_count,
hashtags (array), mentions (array of user_ids),
status (PUBLISHED|DRAFT|ARCHIVED|REMOVED|UNDER_REVIEW),
published_at, edited_at, deleted_at
```

**Key edge cases:**

- Post with 0 likes (unpopular)
- Post with 1M+ likes (viral)
- Post with no text (image only)
- Post with maximum character count
- Reply to a deleted parent post
- Repost chain (A reposted B which reposted C)
- Post under moderation review
- Edited post (`edited_at != null`)
- Post with 50+ hashtags (spam test)
- Post with 20+ media attachments

---

## Geospatial / Location

### Points of Interest

```
id, name, type (RESTAURANT|HOTEL|HOSPITAL|PARK|TRANSIT|SHOP),
address, city, state, country, postal_code,
lat, lng, altitude,
phone, website, email,
hours (object: mon-sun open/close),
rating, review_count,
tags (array), accessibility_features (array),
created_at, verified_at
```

**Key edge cases:**

- `(0, 0)` coordinates (Null Island - "not geocoded yet")
- Location at antimeridian (lng ≈ ±180)
- No address (remote wilderness point)
- 24/7 operation hours
- Closed permanently (no future hours)
- Location with 1000+ reviews vs. 0 reviews
- Address in country without postal codes
- Multiple locations at same exact coordinates

---

## API / Web Request Logs

```
id, request_id (UUID), timestamp,
method (GET|POST|PUT|PATCH|DELETE|OPTIONS|HEAD),
path, query_string, headers (object), body_size_bytes,
user_id (nullable), ip_address, user_agent,
response_status (200|201|400|401|403|404|422|429|500|503),
response_time_ms, response_size_bytes,
error_message (nullable), trace_id
```

**Key edge cases:**

- `response_status = 500` (server error, has error_message)
- `response_status = 429` (rate limited)
- `response_time_ms > 10000` (timeout)
- Unauthenticated request (`user_id = null`)
- `body_size_bytes = 0` (GET request)
- Very large body (upload: > 10MB)
- Path with special characters and URL encoding
- Bot user agent
- Private IP address (`192.168.x.x`)
- Multiple requests in same millisecond (batch)
- HEAD request (response_size_bytes = 0)
