# Otomoto Scraper

Scrape Otomoto.pl Polish car listings: one run returns used-car rows with price, make, model, year, mileage, fuel, gearbox, seller and URL.

**Run it in the cloud, no setup:** [https://apify.com/redomic/otomoto-scraper](https://apify.com/redomic/otomoto-scraper). Works on Apify's free plan; about $3.00 per 1,000 results
(less on paid plans). Schedules, API, webhooks and CSV/Excel/JSON export are built in.

## What you get
- Listing cards from any Otomoto.pl search, offer or dealer inventory URL, or from simple structured filters (make, model, location, year, price, mileage, fuel, gearbox, seller type) — no need to hand-build an Otomoto URL.
- Flat, analysis-ready rows: stable ID, URL, title, numeric price with ISO 4217 currency, make/model/version, year, mileage in km, engine capacity and power, fuel, gearbox, body type, country of origin, city/region, thumbnail, seller type and name, Otomoto's own price evaluation (`in`/`below`/`above`/`none`), price-drop percentage, CEPiK verification flag and created date.
- **Detail mode** (`"mode": "detail"`): opens every offer page and adds the full description (contacts scrubbed), photo gallery, equipment list, VIN (when Otomoto publishes it), CEPiK vehicle history (first registration, owners, insurance/inspection validity, damaged, accident-free, service record, original owner), color, doors, seats, transmission and the seller's GPS pin.
- **Monitor mode** (`"mode": "monitor"`): run on a schedule and get only `new`, `changed` (e.g. a price change) and `removed` listings since the previous run, with `previousPriceAmount`, `priceDelta`, `firstSeenAt`, `lastSeenAt`, a bounded `priceHistory`, `firstPrice`, `lowestPrice` and `priceDropPct`. State is kept per source URL in a named key-value store.
- **Radius search**: set a `location` plus `radiusKm` to let Otomoto filter server-side, or set `lat`/`lng` in detail mode to get `distanceKm` from the seller's map pin.
- **Dealer inventory crawl**: pass a dealer `*.otomoto.pl/inventory` URL to paginate the whole stock, with the dealer's name and ID on every row.
- Per-source status rows and a run summary. Empty, sold or failed sources are reported, never silently skipped.

## Sample output
Real rows from a recent run: [`sample.json`](sample.json) · [`sample.csv`](sample.csv)

```json
{
  "id": "6151476804",
  "url": "https://www.otomoto.pl/osobowe/oferta/bmw-seria-3-ID6IiY7O.html",
  "title": "BMW Seria 3 330i Edition M Sport Shadow",
  "priceAmount": 59000,
  "priceCurrency": "PLN",
  "priceIsGross": true,
  "make": "BMW",
  "makeSlug": "bmw",
  "model": "Seria 3",
  "modelSlug": "seria-3",
  "version": "330i Edition M Sport Shadow",
  "year": 2020,
  "mileageKm": 31500,
  "fuelType": "petrol",
  "gearbox": "automatic",
  "engineCapacityCm3": 1998,
  "enginePowerHp": 252,
  "bodyType": null,
  "countryOrigin": "usa",
  "city": "Warszawa",
  "region": "Mazowieckie",
  "thumbnailUrl": "https://ireland.apollo.olxcdn.com/v1/files/pk8re30p9k0s-OTOMOTOPL/image;s=640x480",
  "sellerType": "business",
  "sellerId": "18253685",
  "sellerName": "Pan America",
  "sellerWebsite": "https://panamerica.otomoto.pl",
  "priceEvaluation": "none",
  "priceDropPercentage": null,
  "priceDropLowestPrice": null,
  "priceDropCurrency": null,
  "cepikVerified": false,
  "createdAt": "2026-11-09T15:33:16Z",
  "shortDescription": "BMW 330i xDrive 2020 | 2.0 Turbo | Automat | M Sport Shadow",
  "isPremiumTopAd": null,
  "scrapedAt": "2026-11-10T05:37:07Z"
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
