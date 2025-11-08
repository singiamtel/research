# Python Libraries for Anki Decks

This document covers open-source Python libraries for reading, writing, and manipulating Anki decks without requiring the Anki application.

## Summary

| Library | Purpose | Stars | Status | Best For |
|---------|---------|-------|--------|----------|
| **genanki** | Create decks | ~2k | Active | Generating new decks programmatically |
| **ankipandas** | Analyze decks | ~400 | Active | Data analysis with pandas |
| **AnkiTools** | CLI tools | ~100 | Active | Command-line automation |
| **py-anki** | Full library | Varies | Active | Comprehensive deck manipulation |

## 1. genanki

**Repository:** https://github.com/kerrickstaley/genanki
**License:** MIT
**Status:** Active, widely used

### Overview

genanki is the most popular Python library for programmatically generating Anki decks. It allows you to create flashcard decks without any Anki installation.

**Key Features:**
- Create notes and cards programmatically
- Define custom models (note types)
- Add media files (images, audio, video)
- Generate APKG files directly
- Stable GUID system for updating existing notes
- Pure Python implementation

### Installation

```bash
pip install genanki
```

### Basic Usage

```python
import genanki

# Create a model (note type)
my_model = genanki.Model(
    1607392319,  # Unique model ID (generate once and hardcode)
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
my_deck = genanki.Deck(
    2059400110,  # Unique deck ID
    'My Spanish Vocabulary')

# Create notes (cards)
note1 = genanki.Note(
    model=my_model,
    fields=['¿Cómo estás?', 'How are you?'])

note2 = genanki.Note(
    model=my_model,
    fields=['Buenos días', 'Good morning'])

# Add notes to deck
my_deck.add_note(note1)
my_deck.add_note(note2)

# Generate APKG file
genanki.Package(my_deck).write_to_file('output.apkg')
```

### Advanced Features

**Adding Media:**

```python
import genanki

# Create note with media reference
note = genanki.Note(
    model=my_model,
    fields=['What sound does a dog make?', '[sound:bark.mp3]'])

my_deck.add_note(note)

# Create package with media
my_package = genanki.Package(my_deck)
my_package.media_files = ['bark.mp3', 'dog.jpg']
my_package.write_to_file('output.apkg')
```

**Custom GUIDs:**

By default, GUIDs are generated from field content. For stable note updating:

```python
note = genanki.Note(
    model=my_model,
    fields=['Question', 'Answer'],
    guid='custom-unique-id-123')  # Custom GUID
```

**Cloze Deletions:**

```python
cloze_model = genanki.Model(
    1234567890,
    'Cloze Model',
    fields=[{'name': 'Text'}],
    templates=[{
        'name': 'Cloze',
        'qfmt': '{{cloze:Text}}',
        'afmt': '{{cloze:Text}}',
    }],
    model_type=genanki.Model.CLOZE)

cloze_note = genanki.Note(
    model=cloze_model,
    fields=['The capital of {{c1::France}} is {{c2::Paris}}'])
```

**Custom CSS:**

```python
my_model = genanki.Model(
    1607392319,
    'Styled Model',
    fields=[...],
    templates=[...],
    css='.card { font-family: arial; font-size: 20px; text-align: center; }')
```

### Generating Model IDs

Generate unique IDs for models and decks:

```bash
python3 -c "import random; print(random.randrange(1 << 30, 1 << 31))"
```

### Limitations

- Cannot read existing APKG files (write-only)
- Focuses on basic card creation
- Advanced Anki features may not be fully supported
- No built-in validation

### Use Cases

- Converting CSV/JSON data to Anki decks
- Automated flashcard generation from APIs
- Bulk deck creation
- Integration with content management systems
- Language learning apps that export to Anki

## 2. ankipandas

**Repository:** https://github.com/klieret/ankipandas
**PyPI:** https://pypi.org/project/ankipandas/
**License:** MIT
**Status:** Active

### Overview

ankipandas brings together Anki and pandas for data analysis and manipulation of flashcard collections. It allows you to load Anki collections as pandas DataFrames.

**Key Features:**
- Read and analyze Anki collections
- Manipulate data using pandas
- Statistical analysis of review history
- Export data to various formats
- Query cards, notes, and reviews
- Integration with Python data science ecosystem

**IMPORTANT:** Write functionality is currently disabled pending issue resolution. Use for reading/analysis only.

### Installation

```bash
pip install ankipandas
```

### Basic Usage

```python
from ankipandas import Collection

# Load collection (finds automatically or specify path)
col = Collection()

# Get cards as DataFrame
cards = col.cards
print(cards.head())

# Get notes as DataFrame
notes = col.notes
print(notes.columns)

# Get review history
revs = col.revs
```

### Analysis Examples

**Card Statistics:**

```python
import pandas as pd
from ankipandas import Collection

col = Collection()

# Cards by deck
cards_by_deck = col.cards.groupby('cdeck').size()
print(cards_by_deck)

# Average ease factor
avg_ease = col.cards['cfactor'].mean()
print(f"Average ease: {avg_ease}")

# Cards by type
card_types = col.cards['ctype'].value_counts()
print(card_types)
```

**Review Analysis:**

```python
# Review counts by date
reviews_by_date = col.revs.groupby(col.revs['rid'].dt.date).size()

# Average review time
avg_time = col.revs['rtime'].mean()

# Success rate
success_rate = (col.revs['rease'] >= 3).mean()
```

**Note Content Search:**

```python
# Find notes containing specific text
notes = col.notes
spanish_notes = notes[notes['nflds'].str.contains('hola', case=False)]
```

### DataFrames Structure

**Cards DataFrame:**
- `cid`: Card ID
- `cdeck`: Deck name
- `ctype`: Card type (new, learning, review)
- `cqueue`: Current queue
- `cdue`: Due date
- `civl`: Interval
- `cfactor`: Ease factor
- And more...

**Notes DataFrame:**
- `nid`: Note ID
- `nmodel`: Model name
- `nflds`: Fields (as string)
- `ntags`: Tags
- Additional metadata

**Reviews DataFrame:**
- `rid`: Review ID (timestamp)
- `rcid`: Card ID
- `rease`: Button pressed
- `rtime`: Time taken
- `rtype`: Review type

### Limitations

- **Writing disabled:** Cannot modify collections currently
- Read-only mode only
- Requires Anki's database schema knowledge for advanced queries
- May not support latest Anki format features

### Use Cases

- Analyzing study patterns
- Generating statistics and reports
- Finding problematic cards
- Data export for visualization
- Research on learning patterns
- Deck quality analysis

## 3. AnkiTools

**PyPI:** https://pypi.org/project/AnkiTools/
**License:** Open source

### Overview

Command-line tools for working with Anki collections.

### Installation

```bash
pip install AnkiTools
```

### Features

- CLI-based deck manipulation
- Batch operations
- Scripting support

**Note:** Limited documentation available. Explore package after installation.

## 4. Direct SQLite Access

For maximum control, you can directly manipulate the SQLite database:

```python
import sqlite3
import zipfile
import json
from pathlib import Path

def read_apkg(apkg_path):
    """Extract and read APKG file"""
    temp_dir = Path('temp_extract')
    temp_dir.mkdir(exist_ok=True)

    # Extract ZIP
    with zipfile.ZipFile(apkg_path, 'r') as zip_ref:
        zip_ref.extractall(temp_dir)

    # Open database
    db_path = temp_dir / 'collection.anki21'
    if not db_path.exists():
        db_path = temp_dir / 'collection.anki2'

    conn = sqlite3.connect(db_path)
    cursor = conn.cursor()

    # Read notes
    cursor.execute('SELECT id, flds, tags FROM notes')
    notes = cursor.fetchall()

    for note_id, fields, tags in notes:
        field_list = fields.split('\x1f')
        print(f"Note {note_id}: {field_list}")

    conn.close()

def create_basic_apkg(output_path):
    """Create basic APKG from scratch"""
    import time

    # Create database
    conn = sqlite3.connect('collection.anki2')
    cursor = conn.cursor()

    # Create tables (simplified)
    cursor.execute('''
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
    ''')

    cursor.execute('''
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
    ''')

    # ... (add more tables and data)

    conn.commit()
    conn.close()

    # Create ZIP
    with zipfile.ZipFile(output_path, 'w') as apkg:
        apkg.write('collection.anki2')
        # Add media files if needed
```

## Comparison

| Feature | genanki | ankipandas | Direct SQLite |
|---------|---------|------------|---------------|
| **Create decks** | ✅ Easy | ❌ No | ✅ Complex |
| **Read decks** | ❌ No | ✅ Easy | ✅ Complex |
| **Analysis** | ❌ No | ✅ Excellent | ⚠️ Manual |
| **Media support** | ✅ Yes | ⚠️ Limited | ✅ Yes |
| **Learning curve** | Low | Medium | High |
| **Flexibility** | Medium | Medium | Maximum |
| **Dependencies** | Minimal | pandas | None |

## Recommendations

**For creating decks:** Use **genanki** - it's the most straightforward and well-documented option.

**For analyzing existing decks:** Use **ankipandas** - pandas integration makes data analysis powerful and intuitive.

**For complex operations:** Use **direct SQLite access** - gives you complete control but requires understanding the database schema.

**For production systems:** Consider genanki with error handling and validation wrappers.

## Resources

- **genanki documentation:** https://github.com/kerrickstaley/genanki
- **ankipandas documentation:** https://ankipandas.readthedocs.io/
- **Anki database structure:** See [database-structure.md](./database-structure.md)
- **APKG format:** See [apkg-format-specification.md](./apkg-format-specification.md)
