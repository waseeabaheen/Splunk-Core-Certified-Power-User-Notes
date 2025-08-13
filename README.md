# Splunk Core Certified Power User — Complete Technical Notes

---

## 1. Transforming Commands for Visualizations

### 1.1 `chart`
**Purpose:** Aggregate metrics into multi-dimensional tables.

**Syntax:**
```spl
chart <agg-function>(<field>) BY <field1>[, <field2>]
```
**Aggregate Functions:**
- `count` — Number of events.
- `sum(field)` — Sum of a numeric field.
- `avg(field)` — Average of a numeric field.
- `min(field)` / `max(field)` — Minimum/maximum values.

**Example:**
```spl
index=web sourcetype=access_combined
chart count BY status
```

---

### 1.2 `timechart`
**Purpose:** Time-series aggregation for trends over time.

**Syntax:**
```spl
timechart [span=<interval>] <agg-function>(<field>) [BY <field2>]
```
**Span Values:**
- `1m` = 1 minute
- `1h` = 1 hour
- `1d` = 1 day

**Example:**
```spl
index=web sourcetype=access_combined
timechart span=1h avg(response_time) BY host
```

---

## 2. Filtering and Formatting Results

### 2.1 `eval`
**Purpose:** Create or modify fields using expressions.

**Syntax:**
```spl
eval <new_field>=<expression>
```
**Operators:**
- Arithmetic: `+ - * / %`
- String concatenation: `"str1" . "str2"`
- Conditional: `if(condition, value_true, value_false)`

**Example:**
```spl
eval load_time=page_end - page_start
eval status_desc=if(status=200, "OK", "Error")
```

---

### 2.2 `search`
**Purpose:** Filter results based on criteria.

**Syntax:**
```spl
search <criteria>
```
**Example:**
```spl
index=security status=failed user=admin
```

---

### 2.3 `where`
**Purpose:** Apply post-pipeline filtering using Boolean conditions.

**Syntax:**
```spl
... | where <condition>
```
**Example:**
```spl
... | where duration > 5 AND bytes > 10000
```

---

### 2.4 `fillnull`
**Purpose:** Replace null or missing values.

**Syntax:**
```spl
fillnull value=<replacement> [field-list]
```
**Example:**
```spl
... | fillnull value=0 error_count
```

---

## 3. Correlating Events

### 3.1 `transaction`
**Purpose:** Group related events into a single transaction.

**Syntax:**
```spl
transaction <field-list> [maxspan=<time>] [maxpause=<time>] [startswith=<search>] [endswith=<search>]
```
**Options:**
- `maxspan` — Total allowed duration of the transaction.
- `maxpause` — Max gap between events.
- `startswith` / `endswith` — Define start and end conditions.

**Example:**
```spl
transaction session_id maxspan=30m
```

---

### 3.2 Using `stats` for Correlation
**Purpose:** Alternative to transactions for performance.

**Example:**
```spl
stats earliest(_time) AS start latest(_time) AS end BY session_id
```

---

## 4. Creating and Managing Fields

### 4.1 Field Extraction
**Methods:**
- Inline extraction using `rex`:
  ```spl
  rex field=<field> "<regex>"
  ```
- Field Extractor (FX) GUI for regex or delimiter-based extractions.

**Example:**
```spl
rex field=_raw "user=(?<username>\w+)"
```

---

## 5. Field Aliases & Calculated Fields

### 5.1 Field Aliases
**Purpose:** Map different field names to a single alias.

**Example:** Alias `src_ip` and `source_ip` to `src`.

---

### 5.2 Calculated Fields
**Purpose:** Create new fields based on expressions.

**Example:**
```
new_field = if(status=200, "OK", "Fail")
```

---

## 6. Tags & Event Types

### 6.1 Tags
**Purpose:** Label specific field-value pairs.

**Example:**
- Tag `host=fw01` as `firewall`.
- Search: `tag=firewall`

---

### 6.2 Event Types
**Purpose:** Save a search definition for reuse.

**Example:**
- Name: `web_errors`
- Search: `sourcetype=access_combined status>=500`

---

## 7. Macros

**Definition:** Reusable SPL snippets with optional arguments.

**Syntax:**
```spl
`macro_name(arguments)`
```
**Example:**
- Macro: `error_search($status$)`
- Definition: `search status=$status$`
- Use: `` `error_search(500)` ``

---

## 8. Workflow Actions

**Purpose:** Perform actions directly from search results.

**Types:**
1. GET — Open an external URL:
   ```
   https://whois.domaintools.com/$domain$
   ```
2. POST — Send data to an API.
3. Search — Trigger another Splunk search.

---

## 9. Data Models

**Definition:** Structured datasets for pivot reporting.

**Structure:**
- Objects: Events, searches, transactions.
- Fields: Attributes within objects.

**Acceleration:** Speeds up Pivot performance.

---

## 10. CIM (Common Information Model)

**Purpose:** Normalize field names for consistency across data sources.

**Example:**  
Normalize `src_ip`, `source_ip`, and `client_ip` → `src`.

---

## Appendix: SPL Functions Reference

### String Functions:
- `len(field)`
- `lower(field)`, `upper(field)`
- `substr(field, start, length)`
- `replace(field, "regex", "replacement")`

### Math Functions:
- `round(number, decimal)`
- `abs(number)`
- `ceil(number)`
- `floor(number)`

### Date/Time Functions:
- `strftime(_time, "%Y-%m-%d %H:%M:%S")`
- `strptime(date_string, "%Y-%m-%d")`
- `relative_time(now(), "-1d@d")`

---

## Quick Command Reference Table

| Command    | Purpose                          | Key Syntax Example |
|------------|----------------------------------|--------------------|
| chart      | Aggregate data into a chart      | `chart count BY status` |
| timechart  | Time-series aggregation          | `timechart span=1h avg(bytes)` |
| eval       | Create/modify fields             | `eval duration=end-start` |
| search     | Filter results                   | `search status=200` |
| where      | Conditional filtering            | `where bytes > 10000` |
| fillnull   | Replace nulls                    | `fillnull value=0` |
| transaction| Group related events             | `transaction session_id` |
| rex        | Extract fields with regex        | `rex "user=(?<u>\w+)"` |

---

