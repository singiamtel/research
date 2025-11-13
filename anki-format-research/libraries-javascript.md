# JavaScript/Node.js Libraries for Anki Decks

This document covers open-source JavaScript and Node.js libraries for reading, writing, and manipulating Anki decks without requiring the Anki application.

## Summary

| Library | Purpose | Status | Environment | Best For |
|---------|---------|--------|-------------|----------|
| **anki-apkg-export** | Create decks | Archived (2018) | Node.js + Browser | Simple deck generation |
| **anki-apkg-parser** | Read decks | Active | Node.js | Parsing existing decks |
| **anki-reader** | Read decks | Active | Node.js + Bun + Browser | Universal reading |
| **@seangenabe/apkg** | Read decks | Active | Node.js | Low-level APKG handling |
| **anki-apkg** | Create decks | Active | Node.js | Customizable deck creation |

## 1. anki-apkg-export

**Repository:** https://github.com/vvscode/js--anki-apkg-export
**NPM:** https://www.npmjs.com/package/anki-apkg-export
**License:** MIT
**Status:** Archived (read-only since August 2018)

### Overview

A universal JavaScript module for generating Anki decks. Originally ported from a Ruby gem, it works in both Node.js and browser environments.

**Key Features:**
- Create APKG files from JavaScript
- Works in Node.js and browsers
- Add media files (images, audio)
- Tag support
- Simple API

**Note:** This library is archived and no longer maintained, but still functional for basic use cases.

### Installation

```bash
npm install anki-apkg-export --save
```

### Basic Usage (Node.js)

```javascript
const AnkiExport = require('anki-apkg-export').default;

// Create deck
const apkg = new AnkiExport('My Deck Name');

// Add cards
apkg.addCard('Question 1', 'Answer 1');
apkg.addCard('Question 2', 'Answer 2');

// With tags
apkg.addCard('Tagged question', 'Tagged answer', { tags: ['spanish', 'vocabulary'] });

// Save to file
apkg
  .save()
  .then(zip => {
    require('fs').writeFileSync('./output.apkg', zip, 'binary');
    console.log('Deck created!');
  })
  .catch(err => console.log(err.stack || err));
```

### Browser Usage

```javascript
import AnkiExport from 'anki-apkg-export';

const apkg = new AnkiExport('My Deck Name');

apkg.addCard('Front', 'Back');

apkg
  .save()
  .then(blob => {
    // Use FileSaver.js or similar to download
    saveAs(blob, 'output.apkg');
  });
```

### Adding Media

```javascript
const fs = require('fs');

const apkg = new AnkiExport('Deck with Media');

// Add card with image
apkg.addCard(
  'What is this animal?',
  '<img src="dog.jpg">',
  { tags: ['animals'] }
);

// Add media file
const imageData = fs.readFileSync('./dog.jpg');
apkg.addMedia('dog.jpg', imageData);

apkg
  .save()
  .then(zip => {
    fs.writeFileSync('./deck-with-media.apkg', zip, 'binary');
  });
```

### Card Options

```javascript
apkg.addCard(front, back, {
  tags: ['tag1', 'tag2'],  // Array of tags
});
```

### Limitations

- Archived/unmaintained since 2018
- Basic features only
- No support for advanced card types (cloze, etc.)
- No custom models/templates
- Limited customization

### Use Cases

- Simple flashcard generation from web apps
- Quick APKG creation without complex setup
- Browser-based deck creation tools
- Legacy projects already using this library

## 2. anki-apkg-parser

**Repository:** https://github.com/74Genesis/anki-apkg-parser
**NPM:** https://www.npmjs.com/package/anki-apkg-parser
**License:** MIT
**Status:** Active
**Language:** TypeScript

### Overview

A Node.js library for parsing and exploring Anki APKG files. Allows extraction of notes, cards, media, and custom database queries.

**Key Features:**
- Unpack APKG archives
- Read notes and cards
- Access media files
- Custom SQL queries
- TypeScript support
- Multiple database format support (anki2, anki21, anki21b)

### Installation

```bash
npm install anki-apkg-parser
```

### Basic Usage

```javascript
const { Unpack, Deck } = require('anki-apkg-parser');

async function parseApkg() {
  // Unpack APKG file
  const unpack = new Unpack();
  const outPath = await unpack.unpack('path/to/deck.apkg', './output-dir');

  // Open deck
  const deck = new Deck(outPath);
  const db = await deck.dbOpen();

  // Get all notes
  const notes = await db.getNotes();
  console.log('Notes:', notes);

  // Get all cards
  const cards = await db.getCards();
  console.log('Cards:', cards);

  // Custom SQL query
  const result = await db.get('SELECT * FROM col');
  console.log('Collection info:', result);

  // Get specific data
  const specificNote = await db.get('SELECT * FROM notes WHERE id = ?', [noteId]);

  // Close database
  await db.close();
}

parseApkg();
```

### Advanced Queries

```javascript
// Get cards from a specific deck
const deckCards = await db.all(
  'SELECT c.* FROM cards c WHERE c.did = ?',
  [deckId]
);

// Get notes with tags
const taggedNotes = await db.all(
  "SELECT * FROM notes WHERE tags LIKE ?",
  ['%vocabulary%']
);

// Get review history
const reviews = await db.all(
  'SELECT * FROM revlog WHERE cid = ? ORDER BY id DESC',
  [cardId]
);

// Parse field data
const notes = await db.getNotes();
notes.forEach(note => {
  const fields = note.flds.split('\x1f');
  console.log('Front:', fields[0]);
  console.log('Back:', fields[1]);
});
```

### Working with Media

```javascript
const fs = require('fs');
const path = require('path');

async function extractMedia(apkgPath) {
  const unpack = new Unpack();
  const outPath = await unpack.unpack(apkgPath, './temp');

  // Media files are numbered (0, 1, 2, ...)
  const mediaPath = path.join(outPath, '0');
  if (fs.existsSync(mediaPath)) {
    const mediaData = fs.readFileSync(mediaPath);
    // Process media...
  }

  // Read media mapping
  const mediaMapPath = path.join(outPath, 'media');
  if (fs.existsSync(mediaMapPath)) {
    const mediaMap = JSON.parse(fs.readFileSync(mediaMapPath, 'utf8'));
    console.log('Media files:', mediaMap);
    // Example: { "0": "image.jpg", "1": "audio.mp3" }
  }
}
```

### TypeScript Support

```typescript
import { Unpack, Deck } from 'anki-apkg-parser';

interface Note {
  id: number;
  flds: string;
  tags: string;
  mid: number;
}

async function parseTyped(apkgPath: string): Promise<Note[]> {
  const unpack = new Unpack();
  const outPath = await unpack.unpack(apkgPath, './out');

  const deck = new Deck(outPath);
  const db = await deck.dbOpen();

  const notes: Note[] = await db.getNotes();

  await db.close();

  return notes;
}
```

### Limitations

- Primarily for reading, not creating
- Requires filesystem access (not browser-friendly)
- Documentation could be more comprehensive

### Use Cases

- Analyzing existing decks
- Extracting content from APKG files
- Deck conversion tools
- Statistics and reporting
- Content migration from Anki

## 3. anki-reader

**Repository:** https://github.com/ewei068/anki-reader
**NPM:** https://www.npmjs.com/package/anki-reader
**License:** MIT
**Status:** Active

### Overview

Universal Anki file reader compatible with Node.js, Bun, and browser environments. Reads both APKG files and collection files.

**Key Features:**
- Multi-runtime support (Node, Bun, Browser)
- Read APKG files
- Read .anki2/.anki21 collections
- TypeScript support
- Lightweight

### Installation

```bash
npm install anki-reader
```

### Basic Usage

```javascript
const { readApkg } = require('anki-reader');

async function readDeck() {
  const deck = await readApkg('./deck.apkg');

  console.log('Deck name:', deck.name);
  console.log('Cards:', deck.cards);
  console.log('Notes:', deck.notes);
  console.log('Media:', deck.media);
}
```

### Browser Usage

```javascript
import { readApkg } from 'anki-reader';

// From File input
document.getElementById('fileInput').addEventListener('change', async (e) => {
  const file = e.target.files[0];
  const deck = await readApkg(file);
  console.log(deck);
});
```

### Use Cases

- Browser-based deck viewers
- Cross-platform tools
- Web applications analyzing decks
- Mobile app integrations (via React Native, etc.)

## 4. @seangenabe/apkg

**NPM:** https://www.npmjs.com/package/@seangenabe/apkg
**License:** Open source
**Status:** Active

### Overview

Low-level APKG loading abstraction that's environment-agnostic. Designed to work with various zip-handling implementations.

**Key Features:**
- Environment-agnostic design
- Stream-based abstractions
- Flexible zip library integration
- TypeScript support

### Installation

```bash
npm install @seangenabe/apkg
```

### Basic Concept

This library provides abstractions, allowing you to choose your own zip library implementation:

```javascript
const { loadApkg } = require('@seangenabe/apkg');

// Use with your preferred zip library
// Implementation varies based on environment
```

### Use Cases

- Custom APKG processors
- Integration with specific zip libraries
- Advanced stream processing
- Custom decoding pipelines

## 5. anki-apkg

**NPM:** https://www.npmjs.com/package/anki-apkg
**License:** Open source
**Status:** Active

### Overview

Offers more flexibility than basic libraries, allowing customization of fields and other deck variables.

### Installation

```bash
npm install anki-apkg
```

### Features

- Customizable field structures
- Enhanced configuration options
- More control over deck properties

**Note:** Documentation is limited. Explore package after installation.

## Direct Implementation

For maximum control, you can implement APKG handling directly:

```javascript
const JSZip = require('jszip');
const sqlite3 = require('sql.js');
const fs = require('fs');

async function readApkg(apkgPath) {
  // Read and unzip
  const data = fs.readFileSync(apkgPath);
  const zip = await JSZip.loadAsync(data);

  // Extract database
  let dbFile = zip.file('collection.anki21') ||
               zip.file('collection.anki2');

  const dbBuffer = await dbFile.async('nodebuffer');

  // Open with sql.js
  const SQL = await require('sql.js')();
  const db = new SQL.Database(dbBuffer);

  // Query notes
  const notes = db.exec('SELECT * FROM notes');
  console.log(notes);

  // Extract media
  const mediaFile = zip.file('media');
  if (mediaFile) {
    const mediaJson = await mediaFile.async('string');
    const mediaMap = JSON.parse(mediaJson);
    console.log('Media:', mediaMap);
  }

  return { db, zip };
}

async function createApkg(outputPath) {
  const SQL = await require('sql.js')();
  const db = new SQL.Database();

  // Create schema
  db.run(`CREATE TABLE col (...)`);
  db.run(`CREATE TABLE notes (...)`);
  db.run(`CREATE TABLE cards (...)`);
  // ... populate tables

  // Export database
  const dbData = db.export();

  // Create ZIP
  const zip = new JSZip();
  zip.file('collection.anki2', dbData);
  zip.file('media', '{}');

  // Save
  const content = await zip.generateAsync({
    type: 'nodebuffer',
    compression: 'DEFLATE'
  });

  fs.writeFileSync(outputPath, content);
}
```

### Required Dependencies

```bash
npm install jszip sql.js
```

## Comparison

| Feature | Export | Parser | Reader | Direct |
|---------|--------|--------|--------|--------|
| **Create** | ✅ Easy | ❌ No | ❌ No | ✅ Complex |
| **Read** | ❌ No | ✅ Advanced | ✅ Easy | ✅ Complex |
| **Browser** | ✅ Yes | ❌ No | ✅ Yes | ✅ Yes |
| **Node.js** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |
| **Media** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |
| **TypeScript** | ⚠️ Partial | ✅ Yes | ✅ Yes | Manual |
| **Active** | ❌ Archived | ✅ Yes | ✅ Yes | N/A |
| **Learning Curve** | Low | Medium | Low | High |

## Recommendations

**For creating decks in 2024+:**
- Use **direct implementation with JSZip** (anki-apkg-export is archived)
- Or contribute to/fork anki-apkg-export

**For reading decks in Node.js:**
- Use **anki-apkg-parser** for full control and custom queries
- Use **anki-reader** for simpler cases

**For browser applications:**
- Use **anki-reader** (actively maintained, universal)
- Use **direct implementation** for custom needs

**For production systems:**
- Implement comprehensive error handling
- Validate APKG format version
- Handle both Legacy 2 and Latest formats
- Test with various deck types

## Browser Considerations

When working in browsers:

1. **Use FileReader API** for file uploads
2. **Use JSZip** for ZIP handling
3. **Use sql.js** for SQLite (WASM-based)
4. **Consider bundle size** (sql.js is ~1.5MB)
5. **Handle async operations** properly

Example:

```javascript
import JSZip from 'jszip';
import initSqlJs from 'sql.js';

async function browserReadApkg(file) {
  const arrayBuffer = await file.arrayBuffer();
  const zip = await JSZip.loadAsync(arrayBuffer);

  const dbFile = zip.file('collection.anki21') || zip.file('collection.anki2');
  const dbBuffer = await dbFile.async('uint8array');

  const SQL = await initSqlJs({
    locateFile: file => `https://sql.js.org/dist/${file}`
  });

  const db = new SQL.Database(dbBuffer);
  return db;
}
```

## Resources

- **JSZip documentation:** https://stuk.github.io/jszip/
- **sql.js documentation:** https://sql.js.org/
- **APKG format:** See [apkg-format-specification.md](./apkg-format-specification.md)
- **Database schema:** See [database-structure.md](./database-structure.md)
- **Example projects:** Search GitHub for "anki apkg" with JavaScript filter

## Future Considerations

Given that anki-apkg-export is archived, the JavaScript ecosystem would benefit from:

1. A modern, actively maintained creation library
2. Better TypeScript support across libraries
3. Comprehensive documentation
4. Support for Latest format (collection.anki21b with zstd)
5. Browser-optimized implementations

Consider contributing to existing projects or creating new ones to fill these gaps.
