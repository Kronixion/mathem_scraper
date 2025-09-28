# Mathem Scrapy Crawler

This project contains a Scrapy-based spider that extracts the full product catalogue from [Mathem](https://www.mathem.se/), including nutritional information for each item.

## Project Layout

- `venv/` – Python virtual environment containing Scrapy and dependencies.
- `requirements.txt` – dependency pin (`scrapy`).
- `scraper/`
  - `scrapy.cfg` – Scrapy configuration file.
  - `mathem/items.py` – item definitions (Scrapy item fields).
  - `mathem/spiders/mathem_spider.py` – main spider implementation.
  - `mathem/settings.py` – project settings (user agent, throttling, etc.).

## Installation

1. Create and activate a virtual environment (for example):
   ```bash
   python -m venv venv
   source venv/bin/activate  # Windows: venv\Scripts\activate
   ```
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

## Running the Spider

1. Change into the Scrapy project directory:
   ```bash
   cd scraper
   ```
2. Run the spider, exporting to a timestamped JSONL:
   ```bash
   mkdir -p exports
   ../venv/bin/scrapy crawl mathem -O exports/products_$(date +%Y%m%d_%H%M%S).jsonl
   ```

The spider will iterate every category and subcategory, following pagination cursors until all products are collected. Each output record includes fields such as `category`, `subcategory`, `subcategory_slug`, `price`, `unit_price`, `quantity_type`, `nutrition`, and `availability`.

## Respecting Mathem's Policies

Mathem requests that automated clients:
- Use a user agent containing the word "bot" and contact information.
- Respect throttling and back off on HTTP 429/5xx responses (the spider uses Scrapy's AutoThrottle and a download delay of 0.5s).
- Obey `robots.txt`. The spider avoids the explicitly disallowed endpoints.

## Output Schema

Each line in the exported JSONL corresponds to one product with the following keys:

- `category`: Top-level category slug (e.g. `264-dryck`).
- `subcategory`: Human-readable subcategory name.
- `subcategory_slug`: Subcategory slug relative to the category.
- `name`: Product name.
- `url`: Canonical Mathem product URL.
- `price`: Item price in SEK.
- `unit_price`: Price per unit/quantity.
- `unit_quantity_name` / `unit_quantity_abbrev`: Unit descriptors.
- `currency`: Currency code (SEK).
- `quantity_type`: Quantity descriptor from product details, if available.
- `nutrition`: Dictionary mapping nutrition keys to values.
- `availability`: Stock state metadata.

## Notes

- The spider relies on Mathem's Next.js `_next/data` endpoints for structured data.
- Subcategory discovery uses the navigation chips and sections in each category page, ensuring 100% coverage.
- Nutrition information is normalised from the detail page's nutrition table when raw `nutritionFacts` are absent.

