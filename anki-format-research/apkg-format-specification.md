# APKG Format Specification

This document provides detailed technical information about the Anki Package (APKG) format based on reverse-engineering efforts by the community.

## Overview

APKG files are **standard ZIP archives** that can be opened with any ZIP utility. They contain:
- SQLite database files with deck data
- Media files (images, audio, video)
- Media mapping files
- Metadata files (in newer versions)

**IMPORTANT:** There is no official specification from Anki developers. All documentation is based on community reverse-engineering efforts.

## File Structure

When extracted, an APKG archive typically contains:

```
deck.apkg (ZIP archive)
├── collection.anki2        # Legacy 1 or compatibility database
├── collection.anki21       # Legacy 2 database (if present)
├── collection.anki21b      # Latest format database (if present)
├── media                   # Media mapping file (JSON or Protobuf)
├── meta                    # Metadata file (Legacy 2 and Latest only)
├── 0                       # Media file (e.g., image.jpg)
├── 1                       # Media file (e.g., audio.mp3)
├── 2                       # Media file (e.g., video.mp4)
└── ...                     # Additional numbered media files
```

All files are stored in the archive root with no subdirectories.

## Format Versions

The APKG format has evolved through three major versions:

### Legacy 1 (2012-2018)

**Database File:** `collection.anki2`

**Characteristics:**
- Database schema version 11
- Deflate compression
- Configuration stored as JSON in TEXT columns
- Media mapping stored as JSON
- No metadata file
- Largest file sizes

**Detection:** Only `collection.anki2` file present

### Legacy 2 (2018-2019)

**Database Files:** `collection.anki21` (primary) + `collection.anki2` (compatibility)

**Characteristics:**
- Database schema version 11
- Deflate compression
- Configuration stored as JSON in TEXT columns
- Includes `meta` file for version information
- Most widely compatible format
- Best supported by third-party tools
- Larger file sizes

**Detection:** Both `collection.anki2` and `collection.anki21` present

### Latest (2020-Present)

**Database Files:** `collection.anki21b` (primary) + `collection.anki2` (compatibility)

**Characteristics:**
- Database schema version 18
- Zstd compression for individual database files
- 12 database tables (expanded from 5)
- Configuration stored as Protobuf messages in BLOB columns
- Includes `meta` file for version information
- Smallest file sizes
- Limited third-party tool support

**Detection:** Both `collection.anki2` and `collection.anki21b` present

## Version Comparison Table

| Feature | Legacy 1 | Legacy 2 | Latest |
|---------|----------|----------|--------|
| **Years** | 2012-2018 | 2018-2019 | 2020-Present |
| **Database File** | collection.anki2 | collection.anki21 | collection.anki21b |
| **Schema Version** | v11 | v11 | v18 |
| **Compression** | Deflate | Deflate | Zstd |
| **Config Format** | JSON/TEXT | JSON/TEXT | Protobuf/BLOB |
| **Tables Count** | 5 | 5 | 12 |
| **Metadata File** | No | Yes | Yes |
| **File Size** | Large | Large | Small |
| **Compatibility** | Older Anki | Wide | Modern only |
| **Library Support** | Limited | Best | Limited |

## Key Components

### 1. Database Files (collection.*)

SQLite database files containing all deck data:
- Cards and notes
- Deck configuration
- Review history
- Scheduling information
- Models (note types)
- Tags

See [database-structure.md](./database-structure.md) for detailed schema information.

### 2. Media Mapping File (media)

Maps user-friendly filenames to numbered media files in the archive.

**Format in Legacy 1 & 2:** JSON object
```json
{
  "0": "photo.jpg",
  "1": "audio.mp3",
  "2": "diagram.png"
}
```

**Format in Latest:** Protobuf message (binary format)

### 3. Media Files (0, 1, 2, ...)

Actual media assets referenced by cards:
- Images (JPG, PNG, GIF, SVG, etc.)
- Audio files (MP3, OGG, WAV, etc.)
- Video files (MP4, WebM, etc.)

Files are named with sequential numbers starting from 0.

### 4. Metadata File (meta)

Present in Legacy 2 and Latest versions only. Contains version and timestamp information.

Example:
```json
{
  "v": 18,
  "creation_timestamp": 1634567890
}
```

## APKG vs COLPKG

Both formats share identical underlying structure but serve different purposes:

| Aspect | APKG | COLPKG |
|--------|------|--------|
| **Purpose** | Share individual decks | Backup entire collections |
| **Import Behavior** | Appends to existing collection | Replaces entire collection |
| **Typical Use** | Distribution, sharing | Backup, migration |
| **Risk Level** | Low (additive) | High (destructive) |

## Compression Details

### Legacy 1 & 2
- Entire ZIP archive uses standard deflate compression
- Database files are not individually compressed

### Latest
- ZIP archive uses standard compression
- Each database file is individually compressed with zstd
- Must decompress database files before reading with SQLite
- Results in significantly smaller file sizes

## Format Detection Algorithm

To programmatically detect which format version you're dealing with:

```python
def detect_apkg_version(extracted_path):
    has_anki2 = os.path.exists(os.path.join(extracted_path, 'collection.anki2'))
    has_anki21 = os.path.exists(os.path.join(extracted_path, 'collection.anki21'))
    has_anki21b = os.path.exists(os.path.join(extracted_path, 'collection.anki21b'))

    if has_anki21b:
        return "Latest (v18)"
    elif has_anki21:
        return "Legacy 2 (v11)"
    elif has_anki2:
        return "Legacy 1 (v11)"
    else:
        return "Unknown or invalid"
```

## Working With APKG Files

### Reading APKG Files

1. Extract ZIP archive to temporary directory
2. Detect format version by checking which database files exist
3. If Latest format, decompress collection.anki21b with zstd
4. Open SQLite database
5. Query tables for cards, notes, and configuration
6. Map media references to numbered files using media mapping

### Creating APKG Files

1. Create SQLite database with appropriate schema
2. Populate cards, notes, decks, and models tables
3. Add media files with sequential numbering (0, 1, 2, ...)
4. Create media mapping file (JSON or Protobuf)
5. Optionally create metadata file
6. If Latest format, compress database with zstd
7. Create ZIP archive with all files at root level

## Common Pitfalls

1. **Wrong directory structure** - All files must be in archive root, not subdirectories
2. **Media numbering** - Must start at 0 and be sequential
3. **Schema version mismatch** - Ensure database schema matches declared version
4. **Missing compatibility database** - Legacy 2 and Latest should include collection.anki2 for older Anki versions
5. **Incorrect compression** - Latest format requires zstd compression for database files
6. **Character encoding** - Use UTF-8 for all text data
7. **Timestamps** - Many IDs are epoch milliseconds, not seconds

## Resources

- **Most comprehensive unofficial documentation:** https://eikowagenknecht.com/posts/understanding-the-anki-apkg-format/
- **AnkiDroid database documentation:** https://github.com/ankidroid/Anki-Android/wiki/Database-Structure
- **Official Anki manual (limited format info):** https://docs.ankiweb.net/

## License and Legal

The APKG format itself is not formally documented or licensed. The Anki application is licensed under AGPL v3. Third-party implementations of the format are typically released under permissive licenses (MIT, Apache, etc.).

**Note:** While the format can be worked with independently of Anki, compatibility with future Anki versions is not guaranteed without official specification.
