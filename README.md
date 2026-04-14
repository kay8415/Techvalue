# Techvalue Categories

Production-ready web scrapers for various categories on **sheeel.com** (Kuwait e-commerce platform) managed under the Techvalue project.

## Categories

### Electronics
- **URL:** https://www.sheeel.com/ar/electronics.html
- **Type:** Hierarchical (with subcategories)
- **Status:** ✅ Active
- **Schedule:** Every 2 days at 3:00 AM UTC
- **S3 Folder:** `electronics/`
- **Features:**
  - Subcategory support with multi-sheet Excel output
  - Comprehensive product data extraction
  - Multiple image downloads
  - AWS S3 integration with date partitioning

## Project Structure

```
Techvalue/
├── electronics/
│   ├── scraper.py          # ElectronicsScraper class
│   ├── requirements.txt    # Python dependencies
│   ├── README.md           # Category documentation
│   ├── .gitignore
│   └── data/               # Local output (gitignored)
│       ├── images/
│       └── *.xlsx
└── .github/
    └── workflows/
        └── main.yml        # GitHub Actions workflow
```

## Automation

All categories run automatically via GitHub Actions:
- **Schedule:** Every 2 days at 3:00 AM UTC
- **Manual Trigger:** Actions → Techvalue Scrapers → Run workflow → Select category

## S3 Data Structure

```
s3://{bucket}/sheeel_data/
    └── year=YYYY/
        └── month=MM/
            └── day=DD/
                └── electronics/
                    ├── images/
                    │   └── {product_id}_{index}.{ext}
                    └── excel-files/
                        └── electronics_{timestamp}.xlsx
```

## Requirements

**GitHub Secrets Required:**
```
AWS_ACCESS_KEY_ID       → AWS IAM access key
AWS_SECRET_ACCESS_KEY   → AWS IAM secret key
S3_BUCKET_NAME          → Target S3 bucket name
```

## Adding New Categories

To add a new category to Techvalue:

1. **Create category folder:**
   ```bash
   cd Techvalue
   mkdir {category_name}
   ```

2. **Copy scraper template:**
   - For standard category: Copy from `electronics/scraper.py` if it has subcategories
   - Update `base_url` and `category` name

3. **Copy requirements.txt:**
   ```bash
   cp electronics/requirements.txt {category_name}/
   ```

4. **Update workflow:**
   Edit `Techvalue/.github/workflows/main.yml`:
   ```yaml
   matrix:
     category:
       - name: electronics
         display_name: Electronics
       - name: {category_name}          # ADD THIS
         display_name: {Display Name}   # ADD THIS
   ```

5. **Update workflow_dispatch options:**
   ```yaml
   workflow_dispatch:
     inputs:
       category:
         options:
           - all
           - electronics
           - {category_name}  # ADD THIS
   ```

## Local Testing

```bash
cd Techvalue/{category_name}

# Install dependencies
pip install -r requirements.txt
playwright install chromium

# Set environment variables
export AWS_ACCESS_KEY_ID="your_key"
export AWS_SECRET_ACCESS_KEY="your_secret"
export S3_BUCKET_NAME="your_bucket"

# Run scraper
python scraper.py
```

## Monitoring

Check workflow runs:
```bash
gh workflow list
gh run list --workflow=main.yml
```

Check S3 data:
```bash
# List all categories for today
aws s3 ls s3://{bucket}/sheeel_data/year=2026/month=04/day=14/ --recursive

# Check specific category
aws s3 ls s3://{bucket}/sheeel_data/year=2026/month=04/day=14/electronics/ --recursive
```

## Technical Stack

- **Python 3.11**
- **Playwright 1.40.0** - Headless browser automation
- **Pandas 2.1.0** - Data processing
- **Boto3 1.34.0** - AWS S3 client
- **Ubuntu 22.04** - GitHub Actions runner

## Features

✅ **Concurrent Subcategory Scraping (NEW!)**
- Async scraping with semaphore for 2.5x speed improvement
- Scrapes 3 subcategories simultaneously (configurable)
- Maintains 100% data accuracy
- See [CONCURRENT_OPTIMIZATION.md](../CONCURRENT_OPTIMIZATION.md) for details

✅ **Subcategory Support**
- Automatic subcategory detection
- Independent scraping per subcategory
- Multi-sheet Excel output

✅ **Robust Pagination**
- Next button detection (works for unlimited pages)
- No dependency on visible page count

✅ **Multi-Image Support**
- Downloads all product images
- Indexed naming: `{product_id}_0.jpg`, `{product_id}_1.jpg`, etc.
- Stored as arrays (not flattened)

✅ **Complete Data Extraction**
- 20+ fields per product
- Features & specifications
- Box contents, warranty
- Prices, availability, deal timers

✅ **AWS S3 Integration**
- Date partitioning
- Organized folder structure
- Automatic upload

## Notes

- **Schedule Offset:** Runs at 3:00 AM UTC to avoid overlap with other projects (Codinity: 1:00 AM, ThinkGrid: 12:00 AM)
- **Fail-safe:** `fail-fast: false` ensures partial success if one category fails
- **Artifact Backup:** 7-day retention on GitHub

---

**Last Updated:** April 14, 2026