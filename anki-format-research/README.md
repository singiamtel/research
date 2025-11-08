# Anki Deck Format Research

This folder contains comprehensive research on the Anki deck format (APKG) and available open-source tools for reading, writing, and creating Anki decks without requiring the Anki application.

## Contents

- **[apkg-format-specification.md](./apkg-format-specification.md)** - Technical details of the APKG format, file structure, and format versions
- **[database-structure.md](./database-structure.md)** - SQLite database schema, tables, and data organization
- **[libraries-python.md](./libraries-python.md)** - Python libraries for working with Anki decks
- **[libraries-javascript.md](./libraries-javascript.md)** - JavaScript/Node.js libraries for APKG manipulation
- **[libraries-other.md](./libraries-other.md)** - Libraries in other programming languages

## Quick Summary

### What is APKG?

APKG (Anki Package) files are ZIP archives containing flashcard decks for Anki, a popular open-source spaced repetition software. The format stores cards, notes, media files, and scheduling information in a SQLite database along with media assets.

### Key Findings

1. **Format is reverse-engineered** - There is no official specification from Anki developers
2. **Three major versions exist** - Legacy 1 (2012-2018), Legacy 2 (2018-2019), and Latest (2020-present)
3. **SQLite-based storage** - All deck data is stored in SQLite databases
4. **Multiple open-source libraries available** - Viable options exist in Python, JavaScript, Go, and other languages
5. **No Anki installation required** - You can read and write APKG files using third-party libraries

### Can You Work With APKG Files Without Anki?

**YES!** Multiple open-source libraries enable reading, writing, and creating Anki decks without the Anki application:

**For Python:**
- **genanki** - Most popular for creating decks programmatically
- **ankipandas** - For data analysis and manipulation using pandas

**For JavaScript/Node.js:**
- **anki-apkg-export** - Generate APKG files (works in browser and Node.js)
- **anki-apkg-parser** - Parse and read existing APKG files
- **anki-reader** - Read APKG and collection files (supports Node, Bun, browser)

**For Go:**
- **flimzy/anki** - Read APKG files

These libraries work by directly manipulating the SQLite database and ZIP archive structure, bypassing the need for Anki itself.

## Limitations

- Most libraries focus on the Legacy 2 format (most widely compatible)
- Latest format (collection.anki21b with zstd compression) has limited library support
- Some advanced Anki features may not be fully supported by third-party libraries
- No official API or specification makes implementation challenging

## Use Cases

These libraries are useful for:
- Programmatically generating flashcard decks from data sources
- Batch converting content to Anki format
- Analyzing existing deck statistics
- Building custom flashcard applications
- Automating deck creation workflows
- Integrating Anki decks with other learning systems
