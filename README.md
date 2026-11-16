# Otomoto Scraper

Export car listings from Otomoto.pl, Poland's largest car marketplace, to a spreadsheet or an API. Each row is one car: price in PLN, make, model, year, mileage, engine, fuel, gearbox, location and seller. Paste a search URL from otomoto.pl or set a few filters, and the first results arrive in about 30 seconds.

**Run it in the cloud, no setup:** [https://apify.com/redomic/otomoto-scraper](https://apify.com/redomic/otomoto-scraper). Works on Apify's free plan; about $0.75 per 1,000 results
(less on paid plans). Schedules, API, webhooks and CSV/Excel/JSON export are built in.

## What you get
A run with the default input (BMW, detail mode) on 16 November 2026 returned 24 cars, among them:

| Car | Price (PLN) | Year | Mileage (km) | Fuel | City | Seller | Market median (PLN) | vs market | Below market |
|---|---|---|---|---|---|---|---|---|---|
| BMW X1 | 76,000 | 2021 | 89,000 | plug-in hybrid | Poznań | private | 83,900 | -9.4% | true |
| BMW Seria 5 540d xDrive mHEV M Sport Edition | 257,800 | 2024 | 79,900 | diesel | Nowogard | private | 278,850 | -7.6% | true |
| BMW X4 xDrive20i GPF M Sport | 182,900 | 2022 | 28,100 | petrol | Rzeszów | dealer | 184,900 | -1.1% | true |
| BMW iX3 40 | 308,650 | 2026 | 10,000 | electric | Bydgoszcz | dealer | 308,325 | +0.1% | false |
| BMW Seria 7 | 499,900 | 2024 | 34,000 | diesel | Wrocław | dealer | 474,900 | +5.3% | false |

The market median compares each car with other Otomoto listings of the same model, one model year either side, and within 30,000 km when at least 5 such cars are for sale. With fewer than 5 comparable cars the market columns stay empty rather than guess; in this run 20 of the 24 cars had a market price. The comparison listings are looked up for you and are not charged.

The `belowMarket` flag marks a car whose price is under that median. Here is one of the rows above as it appears in the dataset:

```json
{
  "title": "BMW X1",
  "priceAmount": 76000,
  "year": 2021,
  "mileageKm": 89000,
  "city": "Poznań",
  "marketMedianPrice": 83900,
  "priceVsMarketPct": -9.4,
  "belowMarket": true
}
```

Turn on `onlyBelowMarket` and the run returns only cars with a market median whose price is under it — the deals, not the full search. You pay only for the rows returned.

Set `priceDroppedOnly` on a daily task and each run returns only cars that actually got cheaper. In a two-run sample, an offer first seen at 87,900 PLN came back the next day at 83,900 PLN with `previousPrice: 87900`, `priceChange: -4000`, `priceChangePct: -4.55` and `daysTracked: 1`. The first run carries `firstSeen` and `daysTracked: 0` but no prior price, so nothing is invented.

Every row also carries the listing URL, Otomoto's own price rating (below, in or above market), the photos, the full description and the equipment list. When the seller publishes them, you also get the VIN and the CEPiK history: first registration, number of owners, and the accident-free and service-record flags.

You can download the results as CSV, Excel or JSON, or read them through the Apify API.

## Sample output
Real rows from a recent run: [`sample.json`](sample.json) · [`sample.csv`](sample.csv)

```json
{
  "id": "6151625062",
  "url": "https://www.otomoto.pl/osobowe/oferta/bmw-x5-ID6IjAH4.html",
  "title": "BMW X5 xDrive35i",
  "priceAmount": 48999,
  "priceCurrency": "PLN",
  "priceIsGross": true,
  "make": "BMW",
  "makeSlug": "bmw",
  "model": "X5",
  "modelSlug": "x5",
  "version": "xDrive35i",
  "year": 2017,
  "mileageKm": 97808,
  "fuelType": "petrol",
  "gearbox": "automatic",
  "engineCapacityCm3": 2979,
  "enginePowerHp": 306,
  "city": "Warszawa",
  "region": "Mazowieckie",
  "thumbnailUrl": "https://ireland.apollo.olxcdn.com/v1/files/vfaesy80qlyv3-OTOMOTOPL/image;s=640x480",
  "sellerType": "business",
  "sellerId": "18253685",
  "sellerName": "Pan America",
  "sellerWebsite": "https://panamerica.otomoto.pl",
  "priceEvaluation": null,
  "priceDropPercentage": null,
  "priceDropLowestPrice": null,
  "priceDropCurrency": null,
  "cepikVerified": false,
  "createdAt": "2026-11-10T13:54:47Z",
  "shortDescription": "BMW X5 2017 F15 35i Steptronic xDrive",
  "scrapedAt": "2026-11-16T20:07:21Z",
  "marketMedianPrice": 82750,
  "priceVsMarketPct": -40.79,
  "marketCohortCount": 10,
  "belowMarket": true,
  "daysListed": 6,
  "firstSeen": "2026-11-16T18:49:00Z",
  "daysTracked": 0,
  "previousPrice": 48999,
  "priceChange": 0,
  "priceChangePct": 0
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
