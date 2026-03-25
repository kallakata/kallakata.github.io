---
title: "Reality Scraper"
excerpt: "Multi-faceted tool to scrape different real estate websites, with additional LLM layer for qe"
layout: page
collection: portfolio
---

[Reality Scraper](https://github.com/kallakata/reality_scraper)

---

# Multi-Site Real Estate Scraper

Scrapes apartment listings for sale from multiple Czech real estate sites and saves them to a single CSV file with automatic duplicate detection.

## Supported Sites

- **Realingo.cz** - Browser automation (Playwright)
- **Sreality.cz** - REST API (fastest!)
- **Bezrealitky.cz** - Browser automation (Playwright)

All sites can be enabled/disabled individually in `config.yaml`.

## Features

- ✅ **Multi-site scraping** - Scrape from multiple sites in one run
- ✅ **Duplicate detection** - Automatically detect cross-site duplicates with confidence scoring
- ✅ **Single CSV output** - All results in one file with source column
- ✅ **Incremental updates** - Only new listings are added on subsequent runs
- ✅ **TUI interface** - Beautiful terminal UI with real-time stats
- ✅ **Filtering** - Location, disposition, price range
- ✅ **Analysis tools** - Export unique properties, generate reports

## Setup

```bash
# Install dependencies
pip install -e .

# Or with dev tools (testing)
pip install -e ".[dev]"

# Install Playwright browsers
python -m playwright install chromium
```

## Quick Start

```bash
# Run with default config (CLI mode)
python scraper.py

# Run with TUI (recommended!)
python scraper.py --tui

# Use custom config
python scraper.py my_config.yaml --tui
```

## Configuration

Edit `config.yaml` to customize search criteria and enabled sites:

```yaml
# Enable/disable sites
sites:
  realingo:
    enabled: true
  sreality:
    enabled: true
  bezrealitky:
    enabled: true

# Search criteria (applies to all sites)
location: "Praha"              # City or district
location_radius: "2"           # Search radius in km
disposition:                   # Apartment layout
  - "3+kk"
  - "3+1"
price_min: null                # Min price in CZK
price_max: 11000000            # Max price in CZK

# Duplicate detection
duplicate_detection:
  enabled: true
  min_confidence: 0.60         # 0.60-1.00 (higher = stricter)
  area_tolerance_m2: 5         # ±5m² for matching

# Output
output_file: "listings.csv"
headless: true                 # Set false to see browser
```

**Location examples:** `Praha`, `Brno`, `Praha 7`, `Nusle`, `Holešovice`

**Disposition options:** `1+kk`, `1+1`, `2+kk`, `2+1`, `3+kk`, `3+1`, `4+kk`, `4+1`, `5+kk`, `5+1`

## Output

CSV file (`listings.csv`) with columns:
- `id` - Unique listing ID
- `source` - Site source (realingo, sreality, bezrealitky)
- `url` - Full listing URL  
- `price` - Price
- `currency` - CZK or EUR
- `location` - Address/location
- `disposition` - Apartment layout (3+kk, etc.)
- `area_m2` - Size in m²
- `scraped_at` - When scraped
- `duplicate_group_id` - Hash for duplicate detection
- `duplicate_confidence` - Confidence score (0.0-1.0)
- `first_seen_as_duplicate` - Timestamp when first marked as duplicate

## TUI Features

Run with `--tui` flag for an interactive terminal interface:

- 📊 **Real-time stats** - Total, unique, duplicates, new listings
- 🎨 **Color highlighting** - Each duplicate group gets a color
- 🔍 **Filtering** - Show all, unique only, or duplicates only
- ⌨️ **Keyboard shortcuts**:
  - `S` - Start scraping
  - `X` - Stop scraping
  - `D` - Toggle duplicate highlighting
  - `U` - Show unique only
  - `G` - Show duplicates only
  - `A` - Show all
  - `R` - Refresh table
  - `O` / `Enter` - Open selected URL in browser
  - `Q` - Quit

## Analysis Tools

### Analyze Duplicates

```bash
# Show statistics and top duplicate groups
python scripts/analyze_duplicates.py listings.csv --report

# Export unique properties only (one per duplicate group)
python scripts/analyze_duplicates.py listings.csv --unique-only unique.csv

# Export duplicates only
python scripts/analyze_duplicates.py listings.csv --duplicates-only dups.csv

# Show top 20 duplicate groups
python scripts/analyze_duplicates.py listings.csv --report --top 20
```

## Testing

```bash
# Run unit tests
pytest

# Run with verbose output
pytest -v

# Test specific module
pytest tests/test_deduplication.py
```

## Notes

- The site shows ~40 listings per map viewport
- Run periodically to capture new listings over time
- Respects rate limits with configurable delay between requests

