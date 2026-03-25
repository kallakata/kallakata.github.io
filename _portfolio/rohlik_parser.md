---
title: "Rohlik Parser"
excerpt: "PDF invoice processor for Rohlik and other online checkout systems"
layout: page
collection: portfolio
---

[Rohlik Parser](https://github.com/kallakata/rohlik-parser)

---

# Rohlik PDF Invoice Parser

OCR-based invoice parser for Rohlik grocery delivery service and other stores. Extracts data from PDF invoices and receipts, combines them per order, and exports to CSV for analysis with an interactive dashboard, Power BI, Looker Studio, or other tools.

## Features

- **Template-Based Parsing**: Flexible YAML templates support multiple stores and invoice formats
- **OCR Text Extraction**: Uses Tesseract OCR to extract text from PDF invoices
- **Dual Document Processing**: Combines data from both invoice and receipt PDFs
- **Intelligent Data Merging**: Matches invoice-receipt pairs per order and merges without duplication
- **Interactive Dashboard**: Beautiful web dashboard with 10+ visualizations and metrics
- **CSV Export**: Flat CSV schema optimized for Power BI, Looker Studio, or any visualization tool
- **Configurable Paths**: Environment variables or defaults for input/output folders
- **Multi-Store Support**: Pre-built templates for Rohlik and generic invoices, easy to add custom templates
- **Partial Data Handling**: Processes orders even if only one PDF type is available

## Requirements

- Python 3.9+
- Tesseract OCR engine
- Poppler (for PDF to image conversion)
- macOS (tested), Linux, or Windows

## Installation

### 1. Install System Dependencies (macOS)

```bash
# Install Homebrew if not already installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Tesseract OCR
brew install tesseract

# Install Czech language pack
brew install tesseract-lang

# Install Poppler (for PDF conversion)
brew install poppler

# Verify installation
tesseract --version
tesseract --list-langs | grep -E "(ces|eng)"
```

### 2. Clone Repository

```bash
git clone <your-repo-url>
cd rohlik-parser
```

### 3. Setup Python Environment

```bash
# Create virtual environment
python3 -m venv venv

# Activate virtual environment
source venv/bin/activate  # macOS/Linux
# or
venv\Scripts\activate  # Windows

# Install Python dependencies
pip install --upgrade pip
pip install -r requirements.txt
```

## Project Structure

```
rohlik-parser/
├── src/
│   ├── __init__.py              # Package initializer
│   ├── ocr_extractor.py         # Tesseract OCR wrapper
│   ├── pdf_parser.py            # Template-based invoice/receipt parsers
│   ├── data_combiner.py         # Data merging logic
│   ├── csv_exporter.py          # CSV export functionality
│   ├── template_loader.py       # YAML template loader
│   └── utils.py                 # Helper functions
├── templates/
│   ├── rohlik.yaml              # Rohlik.cz template
│   └── generic.yaml             # Generic invoice template
├── data/
│   ├── invoices/                # Input PDFs (default location)
│   └── output/                  # Output CSVs
├── ocr-parser.py                # Main CLI script
├── dashboard.py                 # Interactive web dashboard
├── config.py                    # Configuration settings
├── requirements.txt             # Python dependencies
├── README.md                    # This file
└── TEMPLATES.md                 # Template creation guide
```

## Usage

### Basic Usage

```bash
# Activate virtual environment
source venv/bin/activate

# Process all PDFs with default Rohlik template
python ocr-parser.py

# List available templates
python ocr-parser.py --list-templates

# Use generic template for other stores
python ocr-parser.py --template generic

# Use custom template
python ocr-parser.py --custom-template templates/mystore.yaml
```

### Template System

The parser uses **YAML templates** to support multiple invoice formats. This allows you to parse invoices from any store without modifying Python code.

**Built-in Templates:**
- `rohlik` (default) - Rohlik.cz invoices and receipts
- `generic` - Common invoice formats from various stores

**Creating Custom Templates:**

See **[TEMPLATES.md](TEMPLATES.md)** for a comprehensive guide on creating custom templates.

Quick example:
```yaml
name: "My Store"
description: "Template for My Store invoices"

file_patterns:
  invoice: "invoice_*.pdf"
  receipt: "receipt_*.pdf"

invoice:
  order_id:
    pattern: "Order #(\\d+)"
  date:
    pattern: "Date: ([\\d/]+)"
    formats: ["DD/MM/YYYY"]
  total:
    pattern: "Total: \\$([\\d.]+)"
```

Save as `templates/mystore.yaml` and use with:
```bash
python ocr-parser.py --custom-template templates/mystore.yaml
```

### Custom Input/Output

```bash
# Process PDFs from custom folder
python ocr-parser.py --input /path/to/your/invoices

# Specify custom output file
python ocr-parser.py --output /path/to/output.csv

# Both custom paths
python ocr-parser.py --input ~/Downloads/rohlik --output ~/Desktop/orders.csv
```

### Advanced Options

```bash
# Use specific template
python ocr-parser.py --template generic

# Use custom template file
python ocr-parser.py --custom-template /path/to/template.yaml

# List available templates
python ocr-parser.py --list-templates

# Custom input/output paths
python ocr-parser.py --input ~/Downloads/invoices --output ~/Desktop/orders.csv

# Enable debug logging
python ocr-parser.py --log-level DEBUG

# Combine multiple options
python ocr-parser.py --template generic --input ~/invoices --log-level DEBUG

# Get help
python ocr-parser.py --help
```

### Configuration via Environment Variables

```bash
# Set custom input folder
export ROHLIK_INVOICE_FOLDER=/path/to/invoices

# Set custom output folder
export ROHLIK_OUTPUT_FOLDER=/path/to/output

# Run parser
python ocr-parser.py
```

## File Naming Convention

The parser uses **template-based file patterns** to identify invoices and receipts. The default Rohlik template expects:

```
rhl-invoice-YYYY-MM-DD-ORDERID.pdf    # Invoice (Czech, VAT details)
rhl-receipt-YYYY-MM-DD-ORDERID.pdf    # Receipt (English, items list)
```

Example:
```
rhl-invoice-2025-08-18-1107653719.pdf
rhl-receipt-2025-08-18-1107653719.pdf
```

**Custom Patterns:**

Different templates can specify their own patterns. For example, the generic template supports:
- `invoice_*.pdf`
- `receipt_*.pdf`
- `INV-*.pdf`
- Any custom pattern in your template YAML

See [TEMPLATES.md](TEMPLATES.md) for details on configuring file patterns.

## CSV Output Schema

The exported CSV uses a **flat schema** (Option A) with one row per item:

| Column          | Description                              | Example        |
|-----------------|------------------------------------------|----------------|
| `order_id`      | Order ID                                 | 1107653719     |
| `order_date`    | Order date (YYYY-MM-DD)                  | 2025-08-19     |
| `item_name`     | Product name (English)                   | Banana 1pc     |
| `quantity`      | Quantity purchased                       | 1.68           |
| `unit_price`    | Price per unit (CZK)                     | 24.90          |
| `total_price`   | Total price for line item                | 41.83          |
| `vat_rate`      | VAT rate (percentage)                    | 12             |
| `vat_amount`    | VAT amount (CZK)                         | 4.48           |
| `invoice_total` | Total invoice amount (repeated per row)  | 657.27         |
| `payment_method`| Payment method                           | Credits        |
| `source_file`   | Data source                              | invoice+receipt|

### Sample Output

```csv
order_id,order_date,item_name,quantity,unit_price,total_price,vat_rate,vat_amount,invoice_total,payment_method,source_file
1107653719,2025-08-19,Banana 1pc,1.68,24.9,41.83,12,4.48,657.27,Credits,invoice+receipt
1107653719,2025-08-19,Asia World Sweet chili sauce 150ml,1.0,34.9,34.9,12,4.19,657.27,Credits,invoice+receipt
```

## Visualization Options

The CSV output is compatible with multiple visualization tools:

### Option 1: Interactive Python Dashboard (Recommended for Local Use)

The project includes a beautiful, interactive web dashboard built with Plotly Dash:

**Features:**
- 📊 **10+ Interactive Visualizations**
  - Monthly spending trends & cumulative spending
  - Month-over-month comparison across years
  - Shopping frequency heatmap (day × week)
  - Repeat purchase analysis
  - Product category distribution
  - Orders by weekday & order value distribution
  
- 🎯 **Key Metrics Cards**
  - Orders & spending this month (with % change vs last month)
  - Average days between orders
  - Most active shopping day
  - Budget tracking with progress bar

- 🔍 **Interactive Filters**
  - Quick date range buttons (7d, 30d, 3m, all)
  - Custom date range picker
  - Payment method filter
  - Product search

- 🌐 **Local & Fast**
  - Runs in your browser
  - No cloud dependencies
  - Auto-updates when you add new invoices

**Quick Start:**
```bash
# Activate virtual environment
source venv/bin/activate

# Run the dashboard
python dashboard.py

# Open browser to http://localhost:8050
```

**Advanced Options:**
```bash
# Custom CSV file
python dashboard.py --csv /path/to/orders.csv

# Set monthly budget (in CZK or your currency)
python dashboard.py --budget 15000

# Custom port
python dashboard.py --port 8080

# Debug mode with auto-reload
python dashboard.py --debug

# Combine options
python dashboard.py --csv data/output/rohlik_orders.csv --budget 10000 --debug
```

**Dashboard Sections:**

1. **Key Metrics (Top Cards)**
   - Orders this month (↑/↓ % vs last month)
   - Spending this month (↑/↓ % vs last month)
   - Average days between orders
   - Most active shopping day
   - Budget tracking (if budget > 0)

2. **Trend Analysis**
   - Monthly spending over time
   - Cumulative spending chart
   - Month-over-month comparison (current vs previous years)

3. **Shopping Patterns**
   - Frequency heatmap (day of week × week of month)
   - Orders by weekday
   - Repeat purchase analysis (top 20 items)

4. **Product Analysis**
   - Category distribution pie chart
   - Top products by spending & quantity
   - Price distribution histogram

5. **Order Insights**
   - Order value distribution
   - Payment method breakdown
   - Average order value trends

### Option 2: Power BI

See **[POWERBI_GUIDE.md](POWERBI_GUIDE.md)** for detailed setup instructions including:
- 15+ DAX measures
- 4 dashboard pages
- Complete visualization guide

**Quick Start:**
1. Open Power BI Desktop
2. Get Data → Text/CSV
3. Load `data/output/rohlik_orders.csv`
4. Create visualizations

### Option 3: Looker Studio (Google Data Studio)

See **[LOOKER_STUDIO_GUIDE.md](LOOKER_STUDIO_GUIDE.md)** for detailed setup instructions including:
- Google Sheets integration
- Calculated fields
- 5 dashboard pages
- Free and cloud-based

**Quick Start:**
1. Upload CSV to Google Sheets
2. Connect to Looker Studio
3. Create visualizations
4. Share via link (no license needed)

**Looker Studio Benefits:**
- ✅ Completely free
- ✅ Cloud-based (access anywhere)
- ✅ Easy sharing (just send a link)
- ✅ Google ecosystem integration
- ✅ Automatic data refresh with Google Sheets

### Option 4: Excel / Google Sheets

The CSV can be opened directly in Excel or Google Sheets for:
- Pivot tables
- Charts and graphs
- Custom analysis
- Data filtering and sorting

---

## Power BI Integration

### Import CSV into Power BI

1. Open Power BI Desktop
2. Click **Get Data** → **Text/CSV**
3. Browse to `data/output/rohlik_orders.csv`
4. Click **Load**

### Recommended Visualizations

1. **Orders Overview Dashboard**
   - Card: Total orders, total spent, average order value
   - Line chart: Orders over time
   - Bar chart: Payment method distribution

2. **Product Analysis**
   - Table: All items with quantities
   - Bar chart: Top 20 products by revenue
   - Bar chart: Top 20 products by quantity

3. **Price & VAT Analysis**
   - Column chart: Spending by VAT rate
   - Treemap: Category spending

### Power BI DAX Measures (Examples)

```dax
// Total Spent
Total Spent = SUM('rohlik_orders'[total_price])

// Average Order Value
Avg Order Value = DIVIDE([Total Spent], DISTINCTCOUNT('rohlik_orders'[order_id]))

// Item Count
Item Count = COUNTROWS('rohlik_orders')

// Orders Count
Orders Count = DISTINCTCOUNT('rohlik_orders'[order_id])
```

## Configuration

Edit `config.py` to customize settings:

```python
# Folder paths
INPUT_FOLDER = './data/invoices'
OUTPUT_FOLDER = './data/output'

# Template settings
TEMPLATE_DIR = './templates'
DEFAULT_TEMPLATE = 'rohlik'  # Default template name
CUSTOM_TEMPLATE_PATH = None  # Path to custom template file

# Dashboard settings
MONTHLY_BUDGET = 0  # Set monthly budget for tracking (0 = disabled)

# OCR settings
TESSERACT_LANG = 'ces+eng'  # Czech + English
OCR_DPI = 300               # Higher = better quality, slower

# Image preprocessing
PREPROCESS_GRAYSCALE = True
PREPROCESS_DENOISE = True
PREPROCESS_THRESHOLD = True

# CSV settings
OUTPUT_FILENAME = 'rohlik_orders.csv'
CSV_ENCODING = 'utf-8'
DATE_FORMAT = '%Y-%m-%d'

# Legacy file patterns (for backward compatibility)
INVOICE_PATTERN = 'rhl-invoice-*.pdf'
RECEIPT_PATTERN = 'rhl-receipt-*.pdf'
```

### Environment Variables

You can also configure via environment variables:

```bash
# Set custom input folder
export ROHLIK_INVOICE_FOLDER=/path/to/invoices

# Set custom output folder
export ROHLIK_OUTPUT_FOLDER=/path/to/output

# Set default template
export ROHLIK_DEFAULT_TEMPLATE=generic

# Set monthly budget
export ROHLIK_MONTHLY_BUDGET=15000

# Run parser
python ocr-parser.py
```

## Troubleshooting

### Tesseract Not Found

```bash
# Check if Tesseract is installed
which tesseract

# If not found, install it
brew install tesseract tesseract-lang
```

### Czech Language Pack Missing

```bash
# Install language pack
brew install tesseract-lang

# Verify Czech is available
tesseract --list-langs | grep ces
```

### OCR Accuracy Issues

- Ensure PDF quality is good (not heavily compressed)
- Try increasing `OCR_DPI` in `config.py` (e.g., 400 or 600)
- Check image preprocessing settings in `config.py`

### Import Errors

```bash
# Make sure virtual environment is activated
source venv/bin/activate

# Reinstall dependencies
pip install -r requirements.txt
```

### No PDFs Found

- Check that PDFs are in the correct folder (`data/invoices/` by default)
- Verify file naming matches pattern: `rhl-invoice-*` or `rhl-receipt-*`
- Use `--input` flag to specify custom folder

## Development

### Running Tests

```bash
# Test OCR extractor
python src/ocr_extractor.py data/invoices/rhl-invoice-2025-08-18-1107653719.pdf

# Test parser
python src/pdf_parser.py data/invoices/rhl-invoice-2025-08-18-1107653719.pdf

# Test data combiner
python src/data_combiner.py

# Test CSV exporter
python src/csv_exporter.py
```

### Module Documentation

Each module can be run standalone for testing:

```bash
# OCR extraction
python src/ocr_extractor.py <pdf_file>

# Parsing with default template
python src/pdf_parser.py <pdf_file>

# Parsing with custom template
python -c "from src.pdf_parser import InvoiceParser; from src.template_loader import load_template; p = InvoiceParser(load_template('generic')); print(p.parse('invoice.pdf'))"

# List available templates
python -c "from src.template_loader import list_templates; print(list_templates())"

# File matching
python src/utils.py

# Data combining
python src/data_combiner.py

# CSV export
python src/csv_exporter.py
```

## Architecture

### Data Flow

```
1. PDF Files (Invoice + Receipt)
   ↓
2. OCR Extraction (Tesseract)
   ↓
3. Text Parsing (Regex patterns)
   ↓
4. Data Merging (Intelligent combining)
   ↓
5. CSV Export (Flat schema)
   ↓
6. Power BI Visualization
```

### Key Design Decisions

1. **Template-Based Architecture**: YAML templates enable multi-store support without code changes
2. **Backward Compatibility**: Defaults to Rohlik template, maintaining existing functionality
3. **Prefer Invoice Data**: When conflicts arise, invoice metadata is preferred
4. **Multi-Language Support**: Templates can specify language-specific patterns (e.g., Czech + English)
5. **Partial Processing**: Orders with only one PDF type are still processed
6. **Flat Schema**: CSV uses denormalized structure for visualization tool simplicity
7. **Flexible Pattern Matching**: Supports regex patterns for dates, currencies, and item extraction

## Known Limitations

- OCR accuracy depends on PDF quality
- Some item names may be incomplete if split across lines
- VAT details require invoice PDF (not available in receipt-only orders)
- Template patterns may need adjustment for unusual invoice formats
- Currently processes all PDFs in folder (no incremental processing)

## Contributing

Contributions are welcome! Areas for improvement:

- Additional pre-built templates for popular stores (Tesco, Walmart, Amazon, etc.)
- Template validation CLI tool
- Automated template testing
- Enhanced OCR preprocessing
- Multi-currency support in dashboard
- ML-based product categorization

## Resources

- **[TEMPLATES.md](TEMPLATES.md)** - Complete guide to creating custom templates
- **[POWERBI_GUIDE.md](POWERBI_GUIDE.md)** - Power BI dashboard setup (if available)
- **[LOOKER_STUDIO_GUIDE.md](LOOKER_STUDIO_GUIDE.md)** - Looker Studio setup (if available)

## License

[Your License Here]

## Support

For issues or questions:
1. Check [TEMPLATES.md](TEMPLATES.md) for template creation help
2. Review [Troubleshooting](#troubleshooting) section
3. Open an issue on GitHub with sample (redacted) PDFs if needed


