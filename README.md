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

Detail mode also tells you who is selling. Each row carries the seller's public Otomoto badges, the year they joined the site and how many cars they currently have listed, so you can tell an established dealer from a one-off seller. Here is a dealer from a real run:

```json
{
  "sellerName": "INTERAUTO KRAKÓW KOCMYRZOWSKA",
  "sellerBadges": ["Firma", "Bardzo sprawnie odpowiada", "Sprzedający na OTOMOTO od 2023"],
  "sellerSinceYear": 2023,
  "sellerActiveAdCount": 23
}
```

The badges are left out when the seller has none, and private sellers stay unnamed.

Every row also carries the listing URL, Otomoto's own price rating (below, in or above market), the photos, the full description and the equipment list. When the seller publishes them, you also get the VIN and the CEPiK history: first registration, number of owners, and the accident-free and service-record flags.

Each row carries `mainFeatures`, the four highlights Otomoto prints above the price — production year, mileage, engine size and fuel — so a buyer can compare the essentials at a glance. On offer pages these come straight from the listing; on search cards and dealer ads they are read from the same parameters Otomoto shows. Here is one from a real BMW search:

```json
{
  "title": "BMW Seria 4 430i GPF xDrive M Sport",
  "mainFeatures": ["2018", "84935 km", "1998 cm3", "Benzyna"]
}
```

Detail mode also groups the equipment by the categories Otomoto publishes — audio, comfort, driver assistance, performance and safety — in `equipmentGroups`. Each option keeps its own key and label, so you can compare trims or map features into your own catalogue instead of working from one long list. Here is the start of a real car's grouped equipment:

```json
{
  "equipmentGroups": [
    {
      "key": "audio_and_computing",
      "label": "Audio i multimedia",
      "values": [
        { "key": "apple_carplay", "label": "Apple CarPlay" },
        { "key": "android_auto", "label": "Android Auto" },
        { "key": "bluetooth_interface", "label": "Interfejs Bluetooth" }
      ]
    },
    {
      "key": "security",
      "label": "Bezpieczeństwo",
      "values": [
        { "key": "antilock_brake_system", "label": "ABS" },
        { "key": "esp", "label": "ESP" }
      ]
    }
  ]
}
```

Cars with no published equipment have no `equipmentGroups` field, so the column stays empty only where the seller published nothing.

Every row also carries `attributes`, the full dictionary of parameters Otomoto publishes for the car rather than a curated subset. Each entry gives Otomoto's normalized `value` and the `label` it shows a buyer, so facts such as import status, left- or right-hand drive, VIN presence and the complete equipment flags are ready to filter or enrich downstream. Search cards carry the parameters published there; offer pages carry the full set. An excerpt from a real run:

```json
{
  "attributes": {
    "year": { "value": "2025", "label": "2025" },
    "color": { "value": "black", "label": "Czarny" },
    "fuel_type": { "value": "petrol", "label": "Benzyna" },
    "country_origin": { "value": "pl", "label": "Polska" },
    "is_imported_car": { "value": "0", "label": "Nie" },
    "rhd": { "value": "0", "label": "Po lewej" }
  }
}
```

You can download the results as CSV, Excel or JSON, or read them through the Apify API.

## Sample output
Real rows from a recent run: [`sample.json`](sample.json) · [`sample.csv`](sample.csv)

```json
{
  "id": "6151744015",
  "url": "https://www.otomoto.pl/ciezarowe/oferta/man-tgs-18-360-hydrodrive-4x4-smieciarka-faun-ID6Ik5DF.html",
  "title": "MAN TGS 18.360 HYDRODRIVE 4x4 ŚMIECIARKA FAUN",
  "priceAmount": 103197,
  "priceCurrency": "PLN",
  "priceIsGross": false,
  "make": "MAN",
  "makeSlug": "man",
  "model": "TGS 18.360 HYDRODRIVE 4x4 ŚMIECIARKA FAUN",
  "modelSlug": "TGS 18.360 HYDRODRIVE 4x4 ŚMIECIARKA FAUN",
  "version": null,
  "year": 2011,
  "mileageKm": 147613,
  "fuelType": "diesel",
  "gearbox": null,
  "engineCapacityCm3": 10518,
  "enginePowerHp": 360,
  "city": "Mąkowarsko",
  "region": "Kujawsko-pomorskie",
  "thumbnailUrl": "https://ireland.apollo.olxcdn.com/v1/files/2fykrlhq6vhb-OTOMOTOPL/image;s=640x480",
  "sellerType": "business",
  "sellerId": "15311955",
  "sellerName": "Gruchała",
  "sellerWebsite": null,
  "cepikVerified": null,
  "createdAt": "2026-11-17T21:15:22Z",
  "shortDescription": "MAN TGS 18.360 / HYDRODRIVE 4x4 / Śmieciarka Faun Dwuosiowa",
  "mainFeatures": [
    "2011",
    "147613 km",
    "10518 cm3",
    "Diesel"
  ],
  "attributes": {
    "make": {
      "value": "man",
      "label": "MAN"
    },
    "fuel_type": {
      "value": "diesel",
      "label": "Diesel"
    },
    "mileage": {
      "value": "147613",
      "label": "147613 km"
    },
    "engine_capacity": {
      "value": "10518",
      "label": "10518 cm3"
    },
    "engine_power": {
      "value": "360",
      "label": "360 KM"
    },
    "model": {
      "value": "TGS 18.360 HYDRODRIVE 4x4 ŚMIECIARKA FAUN",
      "label": "TGS 18.360 HYDRODRIVE 4x4 ŚMIECIARKA FAUN"
    },
    "year": {
      "value": "2011",
      "label": "2011"
    }
  },
  "scrapedAt": "2026-11-17T22:59:36Z",
  "daysListed": 0,
  "firstSeen": "2026-11-17T22:00:05Z",
  "daysTracked": 0,
  "previousPrice": 103197,
  "priceChange": 0,
  "priceChangePct": 0
}
```

## Run it from code
Python ([apify-client](https://pypi.org/project/apify-client/)):

```python
from apify_client import ApifyClient

client = ApifyClient("<YOUR_APIFY_TOKEN>")
run = client.actor("redomic/otomoto-scraper").call(run_input={"mode": "search", "make": "BMW", "model": "Seria 3", "maxItems": 50, "priceDroppedOnly": true})
for item in client.dataset(run["defaultDatasetId"]).iterate_items():
    print(item)
```

curl (waits for the run and returns the rows):

```bash
curl -X POST "https://api.apify.com/v2/acts/redomic~otomoto-scraper/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"mode": "search", "make": "BMW", "model": "Seria 3", "maxItems": 50, "priceDroppedOnly": true}'
```

## All inputs, modes and pricing
See the full documentation on the Apify page: [https://apify.com/redomic/otomoto-scraper](https://apify.com/redomic/otomoto-scraper)
