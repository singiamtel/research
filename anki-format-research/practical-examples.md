# Practical Examples for Working with Anki Decks

This document provides practical, ready-to-use examples for common tasks when working with Anki decks programmatically.

## Table of Contents

1. [Creating a Simple Deck (Python)](#creating-a-simple-deck-python)
2. [Creating a Simple Deck (JavaScript)](#creating-a-simple-deck-javascript)
3. [Reading and Analyzing an Existing Deck (Python)](#reading-and-analyzing-an-existing-deck-python)
4. [Reading an Existing Deck (JavaScript)](#reading-an-existing-deck-javascript)
5. [Converting CSV to Anki Deck](#converting-csv-to-anki-deck)
6. [Adding Images to Cards](#adding-images-to-cards)
7. [Creating Cloze Deletion Cards](#creating-cloze-deletion-cards)
8. [Extracting All Cards from a Deck](#extracting-all-cards-from-a-deck)
9. [Bulk Deck Generation](#bulk-deck-generation)
10. [Working with Tags](#working-with-tags)

---

## Creating a Simple Deck (Python)

Using **genanki** to create a basic flashcard deck:

```python
#!/usr/bin/env python3
"""
Create a simple Anki deck with basic front/back cards
"""
import genanki
import random

# Generate a unique model ID (do this once, then hardcode it)
MODEL_ID = random.randrange(1 << 30, 1 << 31)  # Example: 1607392319

# Define the card model (note type)
simple_model = genanki.Model(
    MODEL_ID,
    'Simple Model',
    fields=[
        {'name': 'Question'},
        {'name': 'Answer'},
    ],
    templates=[
        {
            'name': 'Card 1',
            'qfmt': '{{Question}}',
            'afmt': '{{FrontSide}}<hr id="answer">{{Answer}}',
        },
    ])

# Create a deck
DECK_ID = random.randrange(1 << 30, 1 << 31)  # Example: 2059400110
my_deck = genanki.Deck(DECK_ID, 'Spanish Vocabulary')

# Add cards
vocabulary = [
    ('Hola', 'Hello'),
    ('Adiós', 'Goodbye'),
    ('Por favor', 'Please'),
    ('Gracias', 'Thank you'),
    ('¿Cómo estás?', 'How are you?'),
]

for question, answer in vocabulary:
    note = genanki.Note(
        model=simple_model,
        fields=[question, answer]
    )
    my_deck.add_note(note)

# Save to file
genanki.Package(my_deck).write_to_file('spanish_vocab.apkg')
print('✓ Created spanish_vocab.apkg with {} cards'.format(len(vocabulary)))
```

---

## Creating a Simple Deck (JavaScript)

Using direct implementation with JSZip and sql.js:

```javascript
/**
 * Create a simple Anki deck using JSZip and sql.js
 * Run: npm install jszip sql.js
 */
const JSZip = require('jszip');
const fs = require('fs');
const initSqlJs = require('sql.js');

async function createSimpleDeck() {
    // Initialize SQL.js
    const SQL = await initSqlJs();
    const db = new SQL.Database();

    // Create database schema (simplified)
    const timestamp = Date.now();
    const deckId = 1;

    // Col table
    db.run(`
        CREATE TABLE col (
            id INTEGER PRIMARY KEY,
            crt INTEGER NOT NULL,
            mod INTEGER NOT NULL,
            scm INTEGER NOT NULL,
            ver INTEGER NOT NULL,
            dty INTEGER NOT NULL,
            usn INTEGER NOT NULL,
            ls INTEGER NOT NULL,
            conf TEXT NOT NULL,
            models TEXT NOT NULL,
            decks TEXT NOT NULL,
            dconf TEXT NOT NULL,
            tags TEXT NOT NULL
        )
    `);

    // Notes table
    db.run(`
        CREATE TABLE notes (
            id INTEGER PRIMARY KEY,
            guid TEXT NOT NULL,
            mid INTEGER NOT NULL,
            mod INTEGER NOT NULL,
            usn INTEGER NOT NULL,
            tags TEXT NOT NULL,
            flds TEXT NOT NULL,
            sfld TEXT NOT NULL,
            csum INTEGER NOT NULL,
            flags INTEGER NOT NULL,
            data TEXT NOT NULL
        )
    `);

    // Cards table
    db.run(`
        CREATE TABLE cards (
            id INTEGER PRIMARY KEY,
            nid INTEGER NOT NULL,
            did INTEGER NOT NULL,
            ord INTEGER NOT NULL,
            mod INTEGER NOT NULL,
            usn INTEGER NOT NULL,
            type INTEGER NOT NULL,
            queue INTEGER NOT NULL,
            due INTEGER NOT NULL,
            ivl INTEGER NOT NULL,
            factor INTEGER NOT NULL,
            reps INTEGER NOT NULL,
            lapses INTEGER NOT NULL,
            left INTEGER NOT NULL,
            odue INTEGER NOT NULL,
            odid INTEGER NOT NULL,
            flags INTEGER NOT NULL,
            data TEXT NOT NULL
        )
    `);

    // Define model
    const modelId = 1607392319;
    const models = {
        [modelId]: {
            id: modelId,
            name: 'Basic',
            type: 0,
            flds: [
                { name: 'Front', ord: 0, sticky: false, rtl: false, font: 'Arial', size: 20 },
                { name: 'Back', ord: 1, sticky: false, rtl: false, font: 'Arial', size: 20 }
            ],
            tmpls: [{
                name: 'Card 1',
                ord: 0,
                qfmt: '{{Front}}',
                afmt: '{{FrontSide}}<hr id="answer">{{Back}}',
                bqfmt: '', bafmt: '', did: null
            }],
            css: '.card { font-family: arial; font-size: 20px; text-align: center; color: black; background-color: white; }',
            latexPre: '', latexPost: '', sortf: 0, did: deckId, mod: Math.floor(timestamp / 1000), usn: -1
        }
    };

    // Define deck
    const decks = {
        [deckId]: {
            id: deckId,
            name: 'JavaScript Deck',
            desc: '',
            conf: 1,
            mod: Math.floor(timestamp / 1000),
            usn: -1,
            collapsed: false
        }
    };

    // Insert col record
    db.run(`INSERT INTO col VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)`, [
        1,
        Math.floor(timestamp / 1000),
        timestamp,
        timestamp,
        11, // schema version
        0, 0, 0,
        '{}', // conf
        JSON.stringify(models),
        JSON.stringify(decks),
        '{"1":{"id":1,"name":"Default"}}', // dconf
        '{}' // tags
    ]);

    // Add sample cards
    const cards = [
        ['Hello', 'Hola'],
        ['Goodbye', 'Adiós'],
        ['Please', 'Por favor']
    ];

    cards.forEach((card, index) => {
        const noteId = timestamp + index;
        const cardId = timestamp + 1000 + index;

        // Insert note
        const fields = card.join('\x1f');
        db.run(`INSERT INTO notes VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)`, [
            noteId,
            `${noteId}`, // guid
            modelId,
            Math.floor(timestamp / 1000),
            -1, // usn
            '', // tags
            fields,
            card[0], // sfld
            0, // csum
            0, // flags
            '' // data
        ]);

        // Insert card
        db.run(`INSERT INTO cards VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)`, [
            cardId,
            noteId,
            deckId,
            0, // ord
            Math.floor(timestamp / 1000),
            -1, // usn
            0, // type (new)
            0, // queue (new)
            index + 1, // due
            0, 0, 2500, 0, 0, 0, 0, 0, 0, ''
        ]);
    });

    // Export database
    const dbData = db.export();

    // Create ZIP
    const zip = new JSZip();
    zip.file('collection.anki2', dbData);
    zip.file('media', '{}');

    // Generate and save
    const content = await zip.generateAsync({
        type: 'nodebuffer',
        compression: 'DEFLATE'
    });

    fs.writeFileSync('javascript_deck.apkg', content);
    console.log('✓ Created javascript_deck.apkg with', cards.length, 'cards');
}

createSimpleDeck().catch(console.error);
```

---

## Reading and Analyzing an Existing Deck (Python)

Using **ankipandas** for data analysis:

```python
#!/usr/bin/env python3
"""
Analyze an existing Anki deck using ankipandas
"""
from ankipandas import Collection
import pandas as pd

def analyze_deck(collection_path=None):
    # Load collection (finds automatically if path not specified)
    col = Collection(collection_path)

    # Get cards DataFrame
    cards = col.cards
    notes = col.notes

    print("=" * 60)
    print("DECK ANALYSIS")
    print("=" * 60)

    # Basic statistics
    print(f"\nTotal cards: {len(cards)}")
    print(f"Total notes: {len(notes)}")

    # Cards by deck
    print("\n--- Cards by Deck ---")
    deck_counts = cards.groupby('cdeck').size().sort_values(ascending=False)
    for deck, count in deck_counts.items():
        print(f"  {deck}: {count} cards")

    # Cards by type
    print("\n--- Cards by Type ---")
    type_map = {0: 'New', 1: 'Learning', 2: 'Review', 3: 'Relearning'}
    type_counts = cards['ctype'].map(type_map).value_counts()
    for card_type, count in type_counts.items():
        print(f"  {card_type}: {count}")

    # Average ease factor
    avg_ease = cards['cfactor'].mean() / 10  # Convert from permille to percentage
    print(f"\n--- Average Ease Factor ---")
    print(f"  {avg_ease:.1f}%")

    # Cards due
    due_cards = len(cards[cards['cqueue'] == 2])  # Review queue
    print(f"\n--- Cards Due for Review ---")
    print(f"  {due_cards} cards")

    # Most common tags
    print("\n--- Top 10 Tags ---")
    all_tags = []
    for tags in notes['ntags']:
        if tags and isinstance(tags, str):
            all_tags.extend(tags.strip().split())

    if all_tags:
        tag_counts = pd.Series(all_tags).value_counts().head(10)
        for tag, count in tag_counts.items():
            print(f"  {tag}: {count}")
    else:
        print("  No tags found")

    # Review history
    if hasattr(col, 'revs'):
        revs = col.revs
        print(f"\n--- Review Statistics ---")
        print(f"  Total reviews: {len(revs)}")
        print(f"  Average review time: {revs['rtime'].mean() / 1000:.1f}s")

        # Success rate (ease >= 3 is considered success)
        if len(revs) > 0:
            success_rate = (revs['rease'] >= 3).mean() * 100
            print(f"  Success rate: {success_rate:.1f}%")

    print("=" * 60)

if __name__ == '__main__':
    # Optionally specify path: analyze_deck('/path/to/collection.anki2')
    analyze_deck()
```

---

## Reading an Existing Deck (JavaScript)

Using **anki-apkg-parser**:

```javascript
/**
 * Read and display contents of an Anki deck
 * Run: npm install anki-apkg-parser
 */
const { Unpack, Deck } = require('anki-apkg-parser');
const fs = require('fs');

async function readDeck(apkgPath) {
    console.log(`Reading deck: ${apkgPath}`);
    console.log('='.repeat(60));

    // Unpack APKG
    const unpack = new Unpack();
    const outPath = await unpack.unpack(apkgPath, './temp_extract');

    // Open deck
    const deck = new Deck(outPath);
    const db = await deck.dbOpen();

    // Get all notes
    const notes = await db.all('SELECT * FROM notes');
    console.log(`\nTotal notes: ${notes.length}`);

    // Get all cards
    const cards = await db.all('SELECT * FROM cards');
    console.log(`Total cards: ${cards.length}`);

    // Display first 5 notes
    console.log('\n--- First 5 Notes ---');
    notes.slice(0, 5).forEach((note, index) => {
        const fields = note.flds.split('\x1f');
        console.log(`\nNote ${index + 1}:`);
        console.log(`  Front: ${fields[0]}`);
        console.log(`  Back: ${fields[1]}`);
        if (note.tags.trim()) {
            console.log(`  Tags: ${note.tags.trim()}`);
        }
    });

    // Card type distribution
    const cardTypes = {};
    cards.forEach(card => {
        const type = card.type;
        const typeName = { 0: 'New', 1: 'Learning', 2: 'Review', 3: 'Relearning' }[type] || 'Unknown';
        cardTypes[typeName] = (cardTypes[typeName] || 0) + 1;
    });

    console.log('\n--- Card Types ---');
    for (const [type, count] of Object.entries(cardTypes)) {
        console.log(`  ${type}: ${count}`);
    }

    // Close database
    await db.close();

    // Cleanup
    fs.rmSync(outPath, { recursive: true, force: true });

    console.log('\n' + '='.repeat(60));
}

// Usage
const deckPath = process.argv[2] || 'deck.apkg';
readDeck(deckPath).catch(console.error);
```

---

## Converting CSV to Anki Deck

Python script to convert a CSV file to an Anki deck:

```python
#!/usr/bin/env python3
"""
Convert CSV to Anki deck
CSV format: question,answer,tags (tags optional)
"""
import csv
import genanki
import random
import sys

def csv_to_anki(csv_path, output_path, deck_name):
    # Create model
    model_id = random.randrange(1 << 30, 1 << 31)
    model = genanki.Model(
        model_id,
        'CSV Import Model',
        fields=[
            {'name': 'Question'},
            {'name': 'Answer'},
        ],
        templates=[{
            'name': 'Card 1',
            'qfmt': '{{Question}}',
            'afmt': '{{FrontSide}}<hr id="answer">{{Answer}}',
        }])

    # Create deck
    deck_id = random.randrange(1 << 30, 1 << 31)
    deck = genanki.Deck(deck_id, deck_name)

    # Read CSV and create notes
    with open(csv_path, 'r', encoding='utf-8') as f:
        reader = csv.reader(f)
        next(reader, None)  # Skip header

        for row in reader:
            if len(row) < 2:
                continue

            question = row[0].strip()
            answer = row[1].strip()
            tags = row[2].split(',') if len(row) > 2 else []
            tags = [t.strip() for t in tags if t.strip()]

            note = genanki.Note(
                model=model,
                fields=[question, answer],
                tags=tags
            )
            deck.add_note(note)

    # Save deck
    genanki.Package(deck).write_to_file(output_path)
    print(f'✓ Created {output_path} with {len(deck.notes)} cards')

if __name__ == '__main__':
    if len(sys.argv) < 2:
        print('Usage: python csv_to_anki.py input.csv [output.apkg] [deck_name]')
        sys.exit(1)

    csv_path = sys.argv[1]
    output_path = sys.argv[2] if len(sys.argv) > 2 else 'output.apkg'
    deck_name = sys.argv[3] if len(sys.argv) > 3 else 'Imported Deck'

    csv_to_anki(csv_path, output_path, deck_name)
```

Example CSV file (`vocab.csv`):
```csv
Question,Answer,Tags
Hello,Hola,spanish greetings
Goodbye,Adiós,spanish greetings
Please,Por favor,spanish polite
Thank you,Gracias,spanish polite
```

---

## Adding Images to Cards

Python example with images:

```python
#!/usr/bin/env python3
"""
Create Anki deck with images
"""
import genanki
import random
import os

# Create model with image support
model = genanki.Model(
    random.randrange(1 << 30, 1 << 31),
    'Image Model',
    fields=[
        {'name': 'Question'},
        {'name': 'Image'},
        {'name': 'Answer'},
    ],
    templates=[{
        'name': 'Card 1',
        'qfmt': '{{Question}}<br>{{Image}}',
        'afmt': '{{FrontSide}}<hr id="answer">{{Answer}}',
    }])

# Create deck
deck = genanki.Deck(
    random.randrange(1 << 30, 1 << 31),
    'Animals Deck')

# Add cards with images
cards_data = [
    ('What animal is this?', 'dog.jpg', 'Dog'),
    ('What animal is this?', 'cat.jpg', 'Cat'),
    ('What animal is this?', 'bird.jpg', 'Bird'),
]

media_files = []

for question, image_file, answer in cards_data:
    if os.path.exists(image_file):
        # Reference image in HTML
        image_html = f'<img src="{image_file}">'

        note = genanki.Note(
            model=model,
            fields=[question, image_html, answer]
        )
        deck.add_note(note)
        media_files.append(image_file)
    else:
        print(f'Warning: {image_file} not found, skipping')

# Create package with media
package = genanki.Package(deck)
package.media_files = media_files
package.write_to_file('animals.apkg')

print(f'✓ Created animals.apkg with {len(deck.notes)} cards and {len(media_files)} images')
```

---

## Creating Cloze Deletion Cards

```python
#!/usr/bin/env python3
"""
Create cloze deletion cards
"""
import genanki
import random

# Create cloze model
cloze_model = genanki.Model(
    random.randrange(1 << 30, 1 << 31),
    'Cloze Model',
    fields=[
        {'name': 'Text'},
        {'name': 'Extra'},
    ],
    templates=[{
        'name': 'Cloze',
        'qfmt': '{{cloze:Text}}',
        'afmt': '{{cloze:Text}}<br>{{Extra}}',
    }],
    model_type=genanki.Model.CLOZE)

# Create deck
deck = genanki.Deck(
    random.randrange(1 << 30, 1 << 31),
    'Geography Cloze')

# Add cloze deletion cards
cloze_cards = [
    ('The capital of {{c1::France}} is {{c2::Paris}}.', 'Western Europe'),
    ('The {{c1::Pacific}} Ocean is the largest ocean.', ''),
    ('Mount {{c1::Everest}} is the highest mountain at {{c2::8,848}} meters.', 'Himalayas'),
]

for text, extra in cloze_cards:
    note = genanki.Note(
        model=cloze_model,
        fields=[text, extra]
    )
    deck.add_note(note)

genanki.Package(deck).write_to_file('geography_cloze.apkg')
print(f'✓ Created geography_cloze.apkg with {len(deck.notes)} cloze cards')
```

---

## Extracting All Cards from a Deck

Python script using direct SQLite access:

```python
#!/usr/bin/env python3
"""
Extract all cards from an APKG file to JSON
"""
import sqlite3
import zipfile
import json
import sys
from pathlib import Path

def extract_cards_to_json(apkg_path, output_json):
    # Extract APKG
    temp_dir = Path('./temp_extract')
    temp_dir.mkdir(exist_ok=True)

    with zipfile.ZipFile(apkg_path, 'r') as zip_ref:
        zip_ref.extractall(temp_dir)

    # Find database
    db_path = temp_dir / 'collection.anki21'
    if not db_path.exists():
        db_path = temp_dir / 'collection.anki2'

    # Connect to database
    conn = sqlite3.connect(db_path)
    conn.row_factory = sqlite3.Row
    cursor = conn.cursor()

    # Get all notes with their cards
    cursor.execute('''
        SELECT
            n.id as note_id,
            n.flds as fields,
            n.tags as tags,
            c.id as card_id,
            c.type as card_type,
            c.queue as card_queue
        FROM notes n
        JOIN cards c ON c.nid = n.id
        ORDER BY n.id
    ''')

    cards_data = []
    for row in cursor.fetchall():
        fields = row['fields'].split('\x1f')
        cards_data.append({
            'note_id': row['note_id'],
            'front': fields[0] if len(fields) > 0 else '',
            'back': fields[1] if len(fields) > 1 else '',
            'tags': row['tags'].strip().split() if row['tags'].strip() else [],
            'card_id': row['card_id'],
            'type': {0: 'new', 1: 'learning', 2: 'review', 3: 'relearning'}.get(row['card_type'], 'unknown'),
            'queue': row['card_queue']
        })

    conn.close()

    # Save to JSON
    with open(output_json, 'w', encoding='utf-8') as f:
        json.dump(cards_data, f, indent=2, ensure_ascii=False)

    print(f'✓ Extracted {len(cards_data)} cards to {output_json}')

    # Cleanup
    import shutil
    shutil.rmtree(temp_dir)

if __name__ == '__main__':
    if len(sys.argv) < 2:
        print('Usage: python extract_cards.py input.apkg [output.json]')
        sys.exit(1)

    apkg_path = sys.argv[1]
    output_json = sys.argv[2] if len(sys.argv) > 2 else 'cards.json'

    extract_cards_to_json(apkg_path, output_json)
```

---

## Bulk Deck Generation

Generate multiple related decks:

```python
#!/usr/bin/env python3
"""
Generate multiple themed decks at once
"""
import genanki
import random

def create_model():
    return genanki.Model(
        random.randrange(1 << 30, 1 << 31),
        'Simple Model',
        fields=[
            {'name': 'Question'},
            {'name': 'Answer'},
        ],
        templates=[{
            'name': 'Card 1',
            'qfmt': '{{Question}}',
            'afmt': '{{FrontSide}}<hr id="answer">{{Answer}}',
        }])

def create_deck_from_data(deck_name, cards_data, model):
    deck = genanki.Deck(
        random.randrange(1 << 30, 1 << 31),
        deck_name)

    for question, answer in cards_data:
        note = genanki.Note(model=model, fields=[question, answer])
        deck.add_note(note)

    return deck

# Define multiple decks
decks_data = {
    'Spanish_Colors': [
        ('Red', 'Rojo'),
        ('Blue', 'Azul'),
        ('Green', 'Verde'),
        ('Yellow', 'Amarillo'),
    ],
    'Spanish_Numbers': [
        ('One', 'Uno'),
        ('Two', 'Dos'),
        ('Three', 'Tres'),
        ('Four', 'Cuatro'),
    ],
    'Spanish_Animals': [
        ('Dog', 'Perro'),
        ('Cat', 'Gato'),
        ('Bird', 'Pájaro'),
        ('Fish', 'Pez'),
    ],
}

# Generate all decks
model = create_model()

for deck_name, cards in decks_data.items():
    deck = create_deck_from_data(deck_name, cards, model)
    filename = f'{deck_name}.apkg'
    genanki.Package(deck).write_to_file(filename)
    print(f'✓ Created {filename} with {len(cards)} cards')

print('\n✓ All decks generated successfully!')
```

---

## Working with Tags

Managing tags in cards:

```python
#!/usr/bin/env python3
"""
Create deck with organized tags
"""
import genanki
import random

model = genanki.Model(
    random.randrange(1 << 30, 1 << 31),
    'Tagged Model',
    fields=[
        {'name': 'Question'},
        {'name': 'Answer'},
    ],
    templates=[{
        'name': 'Card 1',
        'qfmt': '{{Question}}',
        'afmt': '{{FrontSide}}<hr id="answer">{{Answer}}',
    }])

deck = genanki.Deck(
    random.randrange(1 << 30, 1 << 31),
    'Tagged Vocabulary')

# Cards with hierarchical tags
vocabulary = [
    ('Hello', 'Hola', ['spanish', 'spanish::greetings', 'basic']),
    ('Goodbye', 'Adiós', ['spanish', 'spanish::greetings', 'basic']),
    ('Dog', 'Perro', ['spanish', 'spanish::animals', 'nouns']),
    ('Cat', 'Gato', ['spanish', 'spanish::animals', 'nouns']),
    ('Red', 'Rojo', ['spanish', 'spanish::colors', 'adjectives']),
    ('Blue', 'Azul', ['spanish', 'spanish::colors', 'adjectives']),
]

for question, answer, tags in vocabulary:
    note = genanki.Note(
        model=model,
        fields=[question, answer],
        tags=tags
    )
    deck.add_note(note)

genanki.Package(deck).write_to_file('tagged_vocab.apkg')
print(f'✓ Created tagged_vocab.apkg with {len(vocabulary)} tagged cards')

# Tag hierarchy explanation
print('\nTag hierarchy created:')
print('  spanish')
print('    ├── greetings')
print('    ├── animals')
print('    └── colors')
```

---

## Tips and Best Practices

### ID Generation

Always generate IDs once and reuse them:

```python
# Generate IDs (do once)
import random
model_id = random.randrange(1 << 30, 1 << 31)
deck_id = random.randrange(1 << 30, 1 << 31)

# Then hardcode them in your script
MODEL_ID = 1607392319
DECK_ID = 2059400110
```

### HTML in Fields

Anki supports HTML in fields. Escape properly:

```python
import html

# Escape HTML
safe_text = html.escape('<script>alert("XSS")</script>')

# Allow specific HTML
front = '<b>Important:</b> What is 2+2?'
back = 'The answer is <span style="color: red;">4</span>'
```

### Error Handling

Always handle errors when reading decks:

```python
try:
    col = Collection(path)
    cards = col.cards
except Exception as e:
    print(f'Error reading collection: {e}')
    sys.exit(1)
```

### Testing Your Decks

Before using a generated deck:

1. Import into Anki
2. Check a few cards manually
3. Verify media displays correctly
4. Test on mobile if targeting mobile users
5. Validate special characters (emoji, non-Latin scripts)

---

## Resources

- More examples in library documentation:
  - [Python Libraries](./libraries-python.md)
  - [JavaScript Libraries](./libraries-javascript.md)
  - [Other Languages](./libraries-other.md)

- Format specifications:
  - [APKG Format](./apkg-format-specification.md)
  - [Database Structure](./database-structure.md)
