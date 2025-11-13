# Anki Database Structure

This document details the SQLite database schema used in Anki collections, based on the AnkiDroid wiki and community documentation.

## Overview

Anki stores all collection data in a single SQLite database file:
- **Legacy 1:** `collection.anki2`
- **Legacy 2:** `collection.anki21`
- **Latest:** `collection.anki21b` (with zstd compression)

The database uses a relational structure with multiple tables storing cards, notes, media, and metadata.

## Schema Versions

- **Schema v11:** Used by Legacy 1 and Legacy 2
- **Schema v18:** Used by Latest format (2020-present), expanded to 12 tables

This document primarily focuses on schema v11 as it's most widely supported by third-party tools.

## Main Tables

### 1. Cards Table

Stores individual flashcards for review.

**Purpose:** Each card represents a single item to be reviewed, generated from a note template.

**Columns:**

| Column | Type | Description |
|--------|------|-------------|
| `id` | INTEGER PRIMARY KEY | Creation timestamp (epoch milliseconds) |
| `nid` | INTEGER NOT NULL | Note ID (foreign key to notes table) |
| `did` | INTEGER NOT NULL | Deck ID where card belongs |
| `ord` | INTEGER NOT NULL | Ordinal position (template index or cloze number) |
| `mod` | INTEGER NOT NULL | Last modification timestamp (epoch seconds) |
| `usn` | INTEGER NOT NULL | Update sequence number (for syncing) |
| `type` | INTEGER NOT NULL | Card state: 0=new, 1=learning, 2=review, 3=relearning |
| `queue` | INTEGER NOT NULL | Current queue: -3=user buried, -2=sched buried, -1=suspended, 0=new, 1=learning, 2=review, 3=day learning |
| `due` | INTEGER NOT NULL | Due date/position (meaning varies by type) |
| `ivl` | INTEGER NOT NULL | Interval in days (negative for seconds) |
| `factor` | INTEGER NOT NULL | Ease factor (permille, default 2500 = 250%) |
| `reps` | INTEGER NOT NULL | Total number of reviews |
| `lapses` | INTEGER NOT NULL | Number of times card went from review to relearning |
| `left` | INTEGER NOT NULL | Reviews remaining until graduation |
| `odue` | INTEGER NOT NULL | Original due (when card was buried/suspended) |
| `odid` | INTEGER NOT NULL | Original deck ID (for filtered decks) |
| `flags` | INTEGER NOT NULL | Flag color: 0=none, 1=red, 2=orange, 3=green, 4=blue |
| `data` | TEXT NOT NULL | Unused (reserved for future use) |

**Key Relationships:**
- `nid` → `notes.id`
- `did` → Deck ID in col.decks JSON

**Indexes:**
- `ix_cards_nid` on `(nid)`
- `ix_cards_sched` on `(did, queue, due)`
- `ix_cards_usn` on `(usn)`

### 2. Notes Table

Stores the raw content that generates cards.

**Purpose:** Notes contain the actual information (facts) that are displayed on cards through templates.

**Columns:**

| Column | Type | Description |
|--------|------|-------------|
| `id` | INTEGER PRIMARY KEY | Creation timestamp (epoch milliseconds) |
| `guid` | TEXT NOT NULL | Globally unique identifier (for syncing) |
| `mid` | INTEGER NOT NULL | Model/note type ID |
| `mod` | INTEGER NOT NULL | Last modification timestamp (epoch seconds) |
| `usn` | INTEGER NOT NULL | Update sequence number (for syncing) |
| `tags` | TEXT NOT NULL | Space-separated tag list with leading/trailing spaces |
| `flds` | TEXT NOT NULL | Field values separated by `\x1f` (ASCII 31) |
| `sfld` | TEXT NOT NULL | Sort field (first field, used for searching) |
| `csum` | INTEGER NOT NULL | Checksum of first field (for duplicate detection) |
| `flags` | INTEGER NOT NULL | Unused (reserved) |
| `data` | TEXT NOT NULL | Unused (reserved) |

**Key Details:**
- `flds` separator: Character 0x1f (unit separator)
- `tags` format: " tag1 tag2 tag3 " (note spaces)
- `sfld` is typically the first field, stripped of HTML
- `csum` is CRC32 checksum of first field

**Indexes:**
- `ix_notes_usn` on `(usn)`
- `ix_notes_csum` on `(csum)`

### 3. Col Table (Collection)

Single-row table storing global settings.

**Purpose:** Contains collection-wide configuration and metadata.

**Columns:**

| Column | Type | Description |
|--------|------|-------------|
| `id` | INTEGER PRIMARY KEY | Always 1 (single row) |
| `crt` | INTEGER NOT NULL | Collection creation timestamp (epoch seconds) |
| `mod` | INTEGER NOT NULL | Last modification timestamp (milliseconds) |
| `scm` | INTEGER NOT NULL | Schema modification time (milliseconds) |
| `ver` | INTEGER NOT NULL | Version number (11 for legacy, 18 for latest) |
| `dty` | INTEGER NOT NULL | Dirty flag (0=clean, needs flush to disk) |
| `usn` | INTEGER NOT NULL | Update sequence number |
| `ls` | INTEGER NOT NULL | Last sync timestamp (milliseconds) |
| `conf` | TEXT NOT NULL | JSON configuration object |
| `models` | TEXT NOT NULL | JSON object defining note types |
| `decks` | TEXT NOT NULL | JSON object defining deck hierarchy |
| `dconf` | TEXT NOT NULL | JSON object defining deck option groups |
| `tags` | TEXT NOT NULL | JSON object with tag metadata |

**Key JSON Structures:**

**models (Note Types):**
```json
{
  "1234567890": {
    "id": 1234567890,
    "name": "Basic",
    "type": 0,
    "flds": [
      {"name": "Front", "ord": 0, ...},
      {"name": "Back", "ord": 1, ...}
    ],
    "tmpls": [
      {"name": "Card 1", "qfmt": "{{Front}}", "afmt": "{{FrontSide}}<hr>{{Back}}", ...}
    ],
    "css": "card styling here"
  }
}
```

**decks:**
```json
{
  "1": {
    "id": 1,
    "name": "Default",
    "desc": "",
    "conf": 1,
    "extendRev": 50,
    "usn": 0,
    "collapsed": false,
    "newToday": [0, 0],
    "revToday": [0, 0],
    "lrnToday": [0, 0]
  }
}
```

### 4. Revlog Table (Review Log)

Historical record of every review session.

**Purpose:** Tracks all review events for statistics and scheduling.

**Columns:**

| Column | Type | Description |
|--------|------|-------------|
| `id` | INTEGER PRIMARY KEY | Review timestamp (epoch milliseconds) |
| `cid` | INTEGER NOT NULL | Card ID being reviewed |
| `usn` | INTEGER NOT NULL | Update sequence number |
| `ease` | INTEGER NOT NULL | Button pressed (1=again, 2=hard, 3=good, 4=easy) |
| `ivl` | INTEGER NOT NULL | Interval after review (days or seconds if negative) |
| `lastIvl` | INTEGER NOT NULL | Interval before review |
| `factor` | INTEGER NOT NULL | Ease factor after review (permille) |
| `time` | INTEGER NOT NULL | Time taken to review (milliseconds) |
| `type` | INTEGER NOT NULL | 0=learning, 1=review, 2=relearn, 3=cram |

**Indexes:**
- `ix_revlog_cid` on `(cid)`
- `ix_revlog_usn` on `(usn)`

### 5. Graves Table

Tracks deleted items for synchronization.

**Purpose:** Records deletions so they can be synced to other devices.

**Columns:**

| Column | Type | Description |
|--------|------|-------------|
| `usn` | INTEGER NOT NULL | Update sequence number when deleted |
| `oid` | INTEGER NOT NULL | Original object ID that was deleted |
| `type` | INTEGER NOT NULL | Object type: 0=card, 1=note, 2=deck |

**Note:** No primary key; records are cleaned up after successful sync.

## Schema v18 Additional Tables

The Latest format (schema v18) adds several new tables:

- `config` - Key-value configuration storage
- `tags` - Tag metadata
- `notetypes` - Note type definitions (replaces col.models)
- `templates` - Card templates
- `fields` - Field definitions
- Additional supporting tables

This represents a normalization of data previously stored as JSON in the col table.

## Field Separator

Notes use ASCII character 31 (0x1f, unit separator) to delimit fields:

```python
fields = note['flds'].split('\x1f')
# Example: "Hello\x1fWorld" → ["Hello", "World"]
```

## Timestamp Formats

Anki uses different timestamp formats in different contexts:

| Context | Format | Example |
|---------|--------|---------|
| Card/Note IDs | Epoch milliseconds | 1634567890123 |
| Modification times | Epoch seconds | 1634567890 |
| Review log IDs | Epoch milliseconds | 1634567890123 |

## Common Queries

### Get all cards in a deck
```sql
SELECT * FROM cards WHERE did = ?
```

### Get note content for a card
```sql
SELECT n.* FROM notes n
JOIN cards c ON c.nid = n.id
WHERE c.id = ?
```

### Get cards due for review today
```sql
SELECT * FROM cards
WHERE type = 2  -- review cards
  AND queue = 2  -- in review queue
  AND due <= ?  -- today's day number
```

### Get all notes with a specific tag
```sql
SELECT * FROM notes
WHERE tags LIKE '% tagname %'
```

### Review history for a card
```sql
SELECT * FROM revlog
WHERE cid = ?
ORDER BY id DESC
```

## Data Integrity

**Important considerations:**

1. **Foreign Key Consistency:** While SQLite supports foreign keys, Anki doesn't enforce them. Ensure manual consistency.

2. **ID Generation:** IDs are typically millisecond timestamps. Ensure uniqueness:
   ```python
   import time
   new_id = int(time.time() * 1000)
   ```

3. **Update Sequence Numbers (usn):** Used for syncing. Set to -1 for new items that need syncing.

4. **Checksum Calculation:** For notes.csum:
   ```python
   import zlib
   csum = zlib.crc32(first_field.encode('utf-8')) & 0xffffffff
   ```

## Model Types

The `models` JSON defines note types:

- **type = 0:** Standard notes (multiple card templates)
- **type = 1:** Cloze deletion notes (single template, multiple clozes)

## Card Queue Values

Understanding the queue system:

| Queue | Value | Description |
|-------|-------|-------------|
| User Buried | -3 | Manually buried by user |
| Sched Buried | -2 | Automatically buried by scheduler |
| Suspended | -1 | Card suspended |
| New | 0 | New card, never reviewed |
| Learning | 1 | In initial learning phase |
| Review | 2 | In review phase (graduated) |
| Day Learning | 3 | Learning card with interval ≥1 day |

## Best Practices

1. **Always use transactions** when modifying multiple related records
2. **Update modification timestamps** when changing cards or notes
3. **Maintain USN consistency** for proper syncing
4. **Preserve JSON structure** in col table
5. **Handle media references** properly (use `[sound:file.mp3]` or `<img src="file.jpg">`)
6. **Validate HTML** in field content
7. **Strip HTML for sfld** (sort field should be plain text)

## Resources

- **AnkiDroid Database Structure Wiki:** https://github.com/ankidroid/Anki-Android/wiki/Database-Structure
- **Anki Source Code:** https://github.com/ankitects/anki (definitive reference)
- **Schema Evolution:** Track changes across Anki versions in the source repository
