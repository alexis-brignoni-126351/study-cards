# Book Filtering

Limit O'Reilly searches to specific books instead of searching the entire catalog.

## Quick Start

**1. Copy example config:**
```bash
cp .claude/study-config.json.example .claude/study-config.json
```

**2. Edit with your books:**
```json
{
  "oreilly_books": [
    "Managing AI Projects",
    "Generative AI on Microsoft Azure"
  ]
}
```

**3. Done!** Now `/study deep` only searches these books.

## Three Ways to Filter

### Option 1: Config File (Persistent)

Best for: Program-specific book lists that apply to all searches.

**Create:** `.claude/study-config.json`
```json
{
  "oreilly_books": [
    "Book Title 1",
    "Book Title 2"
  ]
}
```

**Works with:**
- Book titles (partial match)
- Book URNs (exact match)

### Option 2: Command Flag (One-Time)

Best for: Exploring outside your usual books.

```bash
/study deep "topic" --books "Book1,Book2"
```

### Option 3: No Filter (Default)

Best for: Maximum flexibility.

Don't create config, don't use `--books` flag.

## Examples

**Filter to AIUP books:**
```json
{
  "oreilly_books": [
    "Managing AI Projects",
    "Generative AI on Microsoft Azure"
  ]
}
```

**Use URNs for exact match:**
```json
{
  "oreilly_books": [
    "urn:orm:book:9798341641006",
    "urn:orm:book:9798341623279"
  ]
}
```

**One-time search outside config:**
```bash
/study deep "Kubernetes" --books "Kubernetes Up & Running"
```

**Temporarily ignore config:**
```bash
/study deep "RAG" --all-books
```

## How It Works

1. System checks for filters (config file or `--books` flag)
2. Searches O'Reilly catalog
3. Filters results to matching books (if configured)
4. Shows you filtered chapters
5. You pick which to read

**No filter?** Shows everything.

## Finding Book URNs

**Method 1: From URL**
```
https://learning.oreilly.com/library/view/-/9798341641006/
                                        ^^^^^^^^^^^^^^^^
                                        This is the URN
```

**Method 2: Search once, note URN**
```bash
/study deep "any topic" --all-books
# Results show: "Managing AI Projects (urn:orm:book:9798341641006)"
# Copy URN to config
```

## Tips

- **Use titles for convenience** - easier to read, works with partial matches
- **Use URNs for precision** - exact match, won't grab wrong book
- **Mix both** - some titles, some URNs in same config
- **Start broad** - no filter first, then add config once you know which books matter

---

See [docs/oreilly-integration.md](docs/oreilly-integration.md) for complete O'Reilly setup.
