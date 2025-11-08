# Libraries for Anki Decks in Other Languages

This document covers open-source libraries for working with Anki decks in programming languages other than Python and JavaScript.

## Go

### flimzy/anki

**Repository:** https://github.com/flimzy/anki
**License:** Apache 2.0
**Status:** Active

**Overview:**
A Go library for reading Anki APKG files.

**Features:**
- Read APKG archives
- Parse SQLite databases
- Extract notes and cards
- Access media files

**Installation:**
```bash
go get github.com/flimzy/anki
```

**Basic Usage:**
```go
package main

import (
    "fmt"
    "github.com/flimzy/anki"
)

func main() {
    // Open APKG file
    apkg, err := anki.OpenFile("deck.apkg")
    if err != nil {
        panic(err)
    }
    defer apkg.Close()

    // Read collection
    collection := apkg.Collection()

    // Iterate through decks
    decks := collection.Decks()
    for decks.Next() {
        deck := decks.Deck()
        fmt.Printf("Deck: %s\n", deck.Name())
    }

    // Read notes
    notes := collection.Notes()
    for notes.Next() {
        note := notes.Note()
        fmt.Printf("Note ID: %d\n", note.ID)
    }
}
```

**Use Cases:**
- CLI tools for Anki manipulation
- Backend services processing APKG files
- Deck conversion utilities
- Analysis tools

## Rust

### anki (Official Anki Source)

**Repository:** https://github.com/ankitects/anki
**License:** AGPL v3
**Status:** Active (official Anki implementation)

**Overview:**
The official Anki application is now written in Rust (with Python frontend). The Rust codebase provides comprehensive APKG handling.

**Features:**
- Complete APKG read/write support
- All Anki formats (Legacy 1, 2, Latest)
- Scheduling algorithms
- Sync protocol implementation
- Media handling
- Database migrations

**Note:** This is the reference implementation. While it's AGPL-licensed (which requires open-sourcing derivative works), you can study it to understand the format or create compatible implementations under different licenses.

**Location:**
- Main Rust code: `rslib/` directory in Anki repository
- Database handling: `rslib/src/storage/`
- APKG import/export: `rslib/src/import_export/`

**Building and Using:**
```bash
git clone https://github.com/ankitects/anki.git
cd anki
# Follow build instructions in docs/
```

**Use Cases:**
- Reference for implementing your own library
- Understanding the official format specification
- Building Anki extensions
- Creating compatible applications

## Ruby

### anki2 gem

**Repository:** https://github.com/albertzak/anki2
**RubyGems:** https://rubygems.org/gems/anki2
**License:** MIT
**Status:** Archived (last update 2015)

**Overview:**
Ruby gem for creating Anki decks. This is the original library that inspired the JavaScript `anki-apkg-export`.

**Installation:**
```bash
gem install anki2
```

**Basic Usage:**
```ruby
require 'anki2'

# Create deck
deck = Anki2::Deck.new('My Deck')

# Add cards
deck.add_card('Front 1', 'Back 1')
deck.add_card('Front 2', 'Back 2')

# Add card with tags
deck.add_card('Tagged front', 'Tagged back', tags: ['vocabulary', 'spanish'])

# Generate APKG file
deck.save_to_file('output.apkg')
```

**Note:** This library is quite old and may not support newer Anki features.

## C#

### AnkiSharp

**Repository:** Various community projects (search GitHub)
**License:** Varies by project
**Status:** Community maintained

**Overview:**
Several C# implementations exist for working with Anki decks, primarily for Windows applications and Unity games.

**Basic Concept:**
```csharp
using System.Data.SQLite;
using System.IO.Compression;

public class AnkiReader
{
    public void ReadApkg(string path)
    {
        // Extract ZIP
        ZipFile.ExtractToDirectory(path, "./temp");

        // Open SQLite database
        using (var conn = new SQLiteConnection("Data Source=./temp/collection.anki2"))
        {
            conn.Open();
            var cmd = conn.CreateCommand();
            cmd.CommandText = "SELECT * FROM notes";

            using (var reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    var id = reader.GetInt64(0);
                    var fields = reader.GetString(6);
                    // Process note...
                }
            }
        }
    }
}
```

**Dependencies:**
- System.IO.Compression (built-in .NET)
- System.Data.SQLite (NuGet package)

**Use Cases:**
- Windows desktop applications
- Unity educational games
- .NET backend services
- WPF/WinForms flashcard apps

## Java

### Anki-Android

**Repository:** https://github.com/ankidroid/Anki-Android
**License:** GPL v3
**Status:** Active (official Android app)

**Overview:**
The official Anki Android app includes a complete Java implementation of APKG handling.

**Features:**
- Full APKG support
- Comprehensive database operations
- Media handling
- Sync support
- Well-documented database structure

**Key Classes:**
- `Collection.java` - Main collection handling
- `Deck.java` - Deck operations
- `Note.java` - Note management
- `Card.java` - Card operations
- `Storage.java` - Database interface

**Use Cases:**
- Android applications
- Java backend services
- Reference implementation for Java developers
- Understanding APKG format from Java perspective

**Example (conceptual):**
```java
import com.ichi2.libanki.*;

// Open collection
Collection col = Storage.collection(context, "path/to/collection.anki2");

// Get decks
HashMap<String, Deck> decks = col.getDecks().all();

// Query notes
List<Note> notes = col.findNotes("deck:\"My Deck\"");

// Add new note
Note note = col.newNote();
note.setField(0, "Front");
note.setField(1, "Back");
col.addNote(note);
```

## PHP

### Community Implementations

**Status:** Limited, mostly custom implementations

**Basic Approach:**
```php
<?php
// Using ZipArchive and SQLite3 (built-in PHP extensions)

$zip = new ZipArchive();
$zip->open('deck.apkg');
$zip->extractTo('./temp');
$zip->close();

// Open database
$db = new SQLite3('./temp/collection.anki2');

// Query notes
$results = $db->query('SELECT * FROM notes');
while ($row = $results->fetchArray()) {
    $fields = explode("\x1F", $row['flds']);
    echo "Front: " . $fields[0] . "\n";
    echo "Back: " . $fields[1] . "\n";
}

$db->close();
?>
```

**Use Cases:**
- Web applications
- WordPress plugins
- Content management systems
- API endpoints for Anki integration

## Swift/Objective-C (iOS)

### AnkiMobile (Closed Source)

**Overview:**
The official iOS app is closed source but uses similar structure to Anki-Android.

**Community Implementations:**
Several open-source iOS projects exist that parse APKG files using Swift.

**Basic Approach:**
```swift
import SQLite3
import ZIPFoundation

func readApkg(path: String) {
    // Unzip
    try FileManager.default.unzipItem(at: URL(fileURLWithPath: path),
                                      to: URL(fileURLWithPath: "./temp"))

    // Open SQLite
    var db: OpaquePointer?
    sqlite3_open("./temp/collection.anki2", &db)

    // Query
    var statement: OpaquePointer?
    sqlite3_prepare_v2(db, "SELECT * FROM notes", -1, &statement, nil)

    while sqlite3_step(statement) == SQLITE_ROW {
        let id = sqlite3_column_int64(statement, 0)
        let fields = String(cString: sqlite3_column_text(statement, 6))
        // Process...
    }

    sqlite3_finalize(statement)
    sqlite3_close(db)
}
```

**Use Cases:**
- iOS applications
- macOS applications
- Educational apps
- Flashcard viewers

## Kotlin

### AnkiDroid (Kotlin migration)

**Overview:**
AnkiDroid is gradually migrating from Java to Kotlin. Kotlin developers can use the same AnkiDroid codebase.

**Advantages:**
- Null safety
- Coroutines for async operations
- Modern syntax
- Full Java interop

**Example:**
```kotlin
import com.ichi2.libanki.*

suspend fun readDeck(path: String) {
    withContext(Dispatchers.IO) {
        val col = Collection(context, path)

        val notes = col.findNotes("deck:\"My Deck\"")
        notes.forEach { noteId ->
            val note = col.getNote(noteId)
            println("Fields: ${note.fields}")
        }
    }
}
```

## Language-Agnostic Approach

For any language with SQLite and ZIP support, you can work with APKG files:

### Requirements

1. **ZIP library** - To extract/create APKG archives
2. **SQLite library** - To read/write the database
3. **Optional: zstd library** - For Latest format support

### Universal Algorithm

**Reading:**
```
1. Extract APKG (ZIP) to temporary directory
2. Detect format version (check which .anki* files exist)
3. If collection.anki21b exists, decompress with zstd
4. Open SQLite database
5. Query tables (notes, cards, col, etc.)
6. Parse fields (split by \x1F)
7. Map media files using media JSON
```

**Writing:**
```
1. Create SQLite database with proper schema
2. Populate tables (col, notes, cards, etc.)
3. Create media mapping JSON
4. If Latest format, compress database with zstd
5. Create ZIP archive with all files at root
6. Name as .apkg
```

## Comparison by Language

| Language | Library Quality | Use Cases | Difficulty |
|----------|----------------|-----------|------------|
| **Python** | ⭐⭐⭐⭐⭐ Excellent | Scripts, data science, backends | Easy |
| **JavaScript** | ⭐⭐⭐⭐ Good | Web apps, Node.js services | Easy |
| **Go** | ⭐⭐⭐ Good | CLI tools, microservices | Medium |
| **Rust** | ⭐⭐⭐⭐⭐ Excellent (official) | Performance-critical apps | Hard |
| **Ruby** | ⭐⭐ Limited | Legacy projects | Easy |
| **C#** | ⭐⭐⭐ Moderate | Windows apps, Unity | Medium |
| **Java** | ⭐⭐⭐⭐ Good (AnkiDroid) | Android apps, enterprise | Medium |
| **PHP** | ⭐⭐ Limited | Web backends | Easy |
| **Swift** | ⭐⭐⭐ Moderate | iOS/macOS apps | Medium |
| **Kotlin** | ⭐⭐⭐⭐ Good (AnkiDroid) | Android apps, modern JVM | Medium |

## Recommendations by Use Case

### Web Backend (API)
- **First choice:** Python (genanki) or Node.js (custom implementation)
- **Alternative:** Go (for high performance)

### Mobile Apps
- **iOS:** Swift with SQLite.swift and ZIPFoundation
- **Android:** Kotlin with AnkiDroid libraries
- **Cross-platform:** React Native with anki-reader

### Desktop Apps
- **Cross-platform:** Electron with JavaScript libraries
- **Windows:** C# with SQLite
- **macOS:** Swift
- **Linux:** Python or Go

### Command-Line Tools
- **Best:** Go or Python
- **Alternative:** Rust (for maximum performance)

### Game Integration (Unity)
- **C#** with custom SQLite implementation

### Research/Analysis
- **Python** with ankipandas (pandas integration)

## Creating Your Own Library

If your language isn't well-supported, implementing APKG support is straightforward:

### Minimal Requirements

1. **ZIP handling** - Virtually all languages have this
2. **SQLite support** - Available for all major languages
3. **Basic string manipulation** - For parsing fields

### Recommended Structure

```
MyAnkiLibrary/
├── models/
│   ├── Card
│   ├── Note
│   ├── Deck
│   └── Collection
├── database/
│   ├── Schema
│   └── Queries
├── io/
│   ├── Reader
│   └── Writer
└── utils/
    ├── FieldParser
    └── MediaHandler
```

### Testing

Always test with:
- Simple two-field cards
- Cards with media
- Cards with tags
- Cloze deletions
- Multiple decks
- All three format versions (Legacy 1, 2, Latest)

## Resources

- **Anki source code:** https://github.com/ankitects/anki (Rust - reference implementation)
- **AnkiDroid source:** https://github.com/ankidroid/Anki-Android (Java/Kotlin)
- **Format documentation:** See [apkg-format-specification.md](./apkg-format-specification.md)
- **Database schema:** See [database-structure.md](./database-structure.md)
- **GitHub search:** Search for "anki apkg" filtered by your language

## Contributing

If you create a library for a language not listed here, consider:
1. Publishing it with a clear license (MIT/Apache recommended)
2. Writing comprehensive documentation
3. Adding examples
4. Testing with various deck types
5. Supporting at least Legacy 2 format
6. Listing it on package managers
7. Contributing to this documentation

## Legal Considerations

- **Anki itself:** AGPL v3 (requires open-sourcing derivatives)
- **APKG format:** Not formally specified, no license restrictions on implementations
- **Third-party libraries:** Various licenses (check before using)
- **Your implementation:** You can license it however you want (MIT/Apache/BSD common)

The format itself is not protected, so you can create compatible implementations under any license.
