# Otomoto Scraper

Scrape Otomoto.pl Polish car listings: one run returns used-car rows with price, make, model, year, mileage, fuel, gearbox, seller and URL.

**Run it in the cloud, no setup:** [https://apify.com/redomic/otomoto-scraper](https://apify.com/redomic/otomoto-scraper). Works on Apify's free plan; about $3.00 per 1,000 results
(less on paid plans). Schedules, API, webhooks and CSV/Excel/JSON export are built in.

## What you get
- Listing cards from any Otomoto.pl search, offer or dealer inventory URL, or from simple structured filters (make, model, location, year, price, mileage, fuel, gearbox, body type, seller type) — no need to hand-build an Otomoto URL. Body type filter uses Otomoto slugs such as `sedan`, `kombi`, `suv`, `compact`, `coupe`, `cabrio`, `minivan`, `city-car`.
- Flat, analysis-ready rows: stable ID, URL, title, numeric price with ISO 4217 currency, make/model/version, year, mileage in km, engine capacity and power, fuel, gearbox, body type, country of origin, city/region, thumbnail, seller type and name, Otomoto's own price evaluation (`in`/`below`/`above`/`none`), price-drop percentage, CEPiK verification flag and created date.
- **In-run market benchmark (search/detail and monitor changes):** every priced row in a make+model group with 3+ listings gets `marketMedianPrice` and `priceVsMarketPct` (negative = below the scraped set median), and the run summary includes a `market` block (count, min/median/mean/max price, median mileage) so you can spot underpriced cars without a second run.
- **Detail mode** (`"mode": "detail"`): opens every offer page and adds the full description (contacts scrubbed), photo gallery, equipment list, VIN (when Otomoto publishes it), CEPiK vehicle history (first registration, owners, insurance/inspection validity, damaged, accident-free, service record, original owner), color, doors, seats, transmission and the seller's GPS pin.
- **Monitor mode** (`"mode": "monitor"`): run on a schedule and get only `new`, `changed` (e.g. a price change) and `removed` listings since the previous run, with `previousPriceAmount`, `firstSeenAt`, `lastSeenAt`, a bounded `priceHistory`, `firstPrice`, `lowestPrice`, `priceDropPct` and `daysOnMarket`. State is kept per source URL in a named key-value store. Unchanged runs bill $0; each `new`/`changed` row is charged once — the $0.003/change event buys Otomoto price history and time-on-market (`firstPrice`, `lowestPrice`, `priceDropPct`, `daysOnMarket`) that list-only scrapers do not return.
- **Radius search**: set a `location` plus `radiusKm` to let Otomoto filter server-side, or set `lat`/`lng` in detail mode to get `distanceKm` from the seller's map pin.
- **Dealer inventory crawl**: pass a dealer `*.otomoto.pl/inventory` URL to paginate the whole stock, with the dealer's name and ID on every row.
- Per-source status rows and a run summary. Empty, sold or failed sources are reported, never silently skipped.

## Sample output
Real rows from a recent run: [`sample.json`](sample.json) · [`sample.csv`](sample.csv)

```json
{
  "id": "6151685962",
  "url": "https://www.otomoto.pl/osobowe/oferta/bmw-seria-3-ID6IjQxk.html",
  "title": "BMW Seria 3 330i M Sport Shadow",
  "priceAmount": 46999,
  "priceCurrency": "PLN",
  "priceIsGross": true,
  "make": "BMW",
  "makeSlug": "bmw",
  "model": "Seria 3",
  "modelSlug": "seria-3",
  "version": "330i M Sport Shadow",
  "year": 2018,
  "mileageKm": 74000,
  "fuelType": "petrol",
  "gearbox": "automatic",
  "engineCapacityCm3": 1998,
  "enginePowerHp": 252,
  "city": "Warszawa",
  "region": "Mazowieckie",
  "thumbnailUrl": "https://ireland.apollo.olxcdn.com/v1/files/3tclzx4ir0gj1-OTOMOTOPL/image;s=640x480",
  "sellerType": "business",
  "sellerId": "16260395",
  "sellerName": "WINCARS",
  "sellerWebsite": "https://WINCARS.otomoto.pl",
  "priceEvaluation": "none",
  "priceDropPercentage": null,
  "priceDropLowestPrice": null,
  "priceDropCurrency": null,
  "cepikVerified": false,
  "createdAt": "2026-11-13T17:47:28Z",
  "shortDescription": "M Sport Shadow | B48 | Duża nawigacja",
  "scrapedAt": "2026-11-13T22:03:47Z",
  "marketMedianPrice": 49399.5,
  "priceVsMarketPct": -4.86,
  "daysListed": 0
}
```

## Run it from code
Python ([apify-client](https://pypi.org/project/apify-client/)):

```python
from apify_client import ApifyClient

client = ApifyClient("<YOUR_APIFY_TOKEN>")
run = client.actor("redomic/otomoto-scraper").call(run_input={"startUrls": [{"url": "https://www.otomoto.pl/osobowe/bmw"}], "maxItems": 24})
for item in client.dataset(run["defaultDatasetId"]).iterate_items():
    print(item)
```

curl (waits for the run and returns the rows):

```bash
curl -X POST "https://api.apify.com/v2/acts/redomic~otomoto-scraper/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"startUrls": [{"url": "https://www.otomoto.pl/osobowe/bmw"}], "maxItems": 24}'
```

## All inputs, modes and pricing
See the full documentation on the Apify page: [https://apify.com/redomic/otomoto-scraper](https://apify.com/redomic/otomoto-scraper)
