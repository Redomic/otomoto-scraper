# Otomoto Scraper

Export car listings from Otomoto.pl, Poland's largest car marketplace, to a spreadsheet or an API. Each row is one car: price in PLN, make, model, year, mileage, engine, fuel, gearbox, location and seller. Paste a search URL from otomoto.pl or set a few filters, and the first results arrive in about 30 seconds.

**Run it in the cloud, no setup:** [https://apify.com/redomic/otomoto-scraper](https://apify.com/redomic/otomoto-scraper). Works on Apify's free plan; about $0.75 per 1,000 results
(less on paid plans). Schedules, API, webhooks and CSV/Excel/JSON export are built in.

## What you get
A run with the default input (BMW, detail mode) on 15 November 2026 returned these cars, among others:

| Car | Price (PLN) | Year | Mileage (km) | Fuel | Power (hp) | City | Seller |
|---|---|---|---|---|---|---|---|
| BMW Seria 3 330i M Sport Shadow | 46,999 | 2018 | 74,000 | petrol | 252 | Warszawa | dealer |
| BMW Seria 5 530e PHEV | 73,677 | 2023 | 54,622 | plug-in hybrid | 299 | Warszawa | private |
| BMW X3 xDrive30i Luxury Line | 97,000 | 2020 | 49,597 | petrol | 252 | Bielsk Podlaski | private |
| BMW Seria 7 740i sport | 54,900 | 2019 | 115,000 | petrol | 340 | Warszawa | dealer |
| BMW iX3 40 | 308,650 | 2026 | 10,000 | electric | 320 | Bydgoszcz | dealer |

Every row also carries the listing URL, Otomoto's own price rating (below, in or above market), the photos, the full description and the equipment list. When the seller publishes them, you also get the VIN and the CEPiK history: first registration, number of owners, and the accident-free and service-record flags.

You can download the results as CSV, Excel or JSON, or read them through the Apify API.

## Sample output
Real rows from a recent run: [`sample.json`](sample.json) · [`sample.csv`](sample.csv)

```json
{
  "id": "6151608344",
  "url": "https://www.otomoto.pl/osobowe/oferta/bmw-seria-4-ID6Ijvlq.html",
  "title": "BMW Seria 4 430i GPF xDrive M Sport",
  "priceAmount": 41400,
  "priceCurrency": "PLN",
  "priceIsGross": true,
  "make": "BMW",
  "makeSlug": "bmw",
  "model": "Seria 4",
  "modelSlug": "seria-4",
  "version": "430i GPF xDrive M Sport",
  "year": 2018,
  "mileageKm": 84935,
  "fuelType": "petrol",
  "gearbox": "automatic",
  "engineCapacityCm3": 1998,
  "enginePowerHp": 252,
  "city": "Warszawa",
  "region": "Mazowieckie",
  "thumbnailUrl": "https://ireland.apollo.olxcdn.com/v1/files/brf5iofxpueo-OTOMOTOPL/image;s=640x480",
  "sellerType": "business",
  "sellerId": "18253685",
  "sellerName": "Pan America",
  "sellerWebsite": "https://panamerica.otomoto.pl",
  "priceEvaluation": null,
  "priceDropPercentage": null,
  "priceDropLowestPrice": null,
  "priceDropCurrency": null,
  "cepikVerified": false,
  "createdAt": "2026-11-09T16:11:14Z",
  "shortDescription": "BMW 430i Coupe 2018 | M Pakiet | xDrive | Automat",
  "scrapedAt": "2026-11-15T20:31:24Z",
  "daysListed": 6
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
