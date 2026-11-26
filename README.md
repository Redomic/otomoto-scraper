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
  "categoryCode": "osobowe",
  "categoryName": "Osobowe",
  "categoryId": 29,
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

Set `priceDroppedOnly` on a daily task and each run returns only cars that actually got cheaper. In a two-run sample, an offer first seen at 87,900 PLN came back the next day at 83,900 PLN with `previousPrice: 87900`, `priceChange: -4000`, `priceChangePct: -4.55` and `daysTracked: 1`. A car we have tracked in an earlier run is also checked against our earliest price for it, so one that never got cheaper is not sent as a false alert just because it is new to your account. The run status records how many cars were checked and how many had no price cut.

For a feed of newly listed cars, set `newWithinDays` to the number of days you care about, for example 7. The run keeps only cars our shared price history first saw in that window, so it works on your very first run instead of waiting for the Actor to build your own history. Every returned row's `firstSeen` falls inside the window, cars we have no first-sighting date for are left out, and the run status says how many cars were checked and dropped.

Business buyers need to know whether they can reclaim VAT. When Otomoto shows the badge, a row carries `invoiceIssued: true` for a seller who issues a VAT invoice (faktura VAT) and `vatIncluded: true` for a price that already includes VAT. Here is a real BMW from a search run with both badges:

```json
{
  "title": "BMW M2 sport",
  "priceAmount": 274900,
  "city": "Kraków",
  "invoiceIssued": true,
  "vatIncluded": true
}
```

Both fields are left off a car where Otomoto publishes no such badge, so a private sale never gets a made-up `false`.

Detail mode also returns `netPrice`, the VAT-exclusive figure Otomoto prints beside the gross price. It saves working the pre-tax price out yourself when you reclaim VAT. Here is a real Skoda Kamiq offer, where Otomoto shows 87,900 PLN gross and 71,464 PLN net:

```json
{
  "title": "Skoda Kamiq 1.0 TSI Selection",
  "priceAmount": 87900,
  "netPrice": 71464
}
```

The field is left off where Otomoto publishes no net price, so a search card or an offer page without the figure does not gain an empty column.

Detail mode also tells you who is selling. Each row carries the seller's public Otomoto badges, the year they joined the site and how many cars they currently have listed, so you can tell an established dealer from a one-off seller. A dealer's row also includes `sellerAddress`, the public street address on their Otomoto profile, so you can see where a rival's stock actually sits. It also carries `sellerPostalCode`, the postcode Otomoto publishes on the same dealer profile, so you can group a rival's listings by area or match them to your own catchment. Here is a dealer from a real run:

```json
{
  "sellerName": "INTERAUTO KRAKÓW KOCMYRZOWSKA",
  "sellerAddress": "KOCMYRZOWSKA 1A",
  "sellerPostalCode": "31-831",
  "sellerBadges": ["Firma", "Bardzo sprawnie odpowiada", "Sprzedający na OTOMOTO od 2023"],
  "sellerSinceYear": 2023,
  "sellerActiveAdCount": 23
}
```

Detail mode also gives a dealer's opening hours in `sellerWorkingHours`, so you can see when a rival's showroom is staffed. The hours come from the dealer's public Otomoto profile and are left out for private sellers or dealers who publish no timetable. Here is the same dealer's week:

```json
{
  "sellerWorkingHours": [
    { "day": "Mon", "openAt": "09:00", "closeAt": "18:00" },
    { "day": "Tue", "openAt": "09:00", "closeAt": "18:00" },
    { "day": "Wed", "openAt": "09:00", "closeAt": "18:00" },
    { "day": "Thu", "openAt": "09:00", "closeAt": "18:00" },
    { "day": "Fri", "openAt": "09:00", "closeAt": "18:00" },
    { "day": "Sat", "openAt": "09:00", "closeAt": "14:00" }
  ]
}
```

The badges and address are left out when the seller publishes none, and private sellers stay unnamed.

Every row also carries `sellerUuid`, Otomoto's stable identifier for the seller. A dealer keeps the same UUID when they rename their business or edit their profile, so you can group every car they list into one inventory — in search results, on an offer page and on their dealer inventory page. It is left off where Otomoto publishes none. Here is that same dealer's row:

```json
{
  "sellerName": "INTERAUTO KRAKÓW KOCMYRZOWSKA",
  "sellerId": "14399773",
  "sellerUuid": "f5ee8a55-62a5-46d7-a11a-eaf0e0daa724"
}
```

Detail mode also returns the dealer's Otomoto advertising package in `packages` — the subscription tier Otomoto shows on the offer page, a useful signal of how much a rival invests in their listings. That same dealer came back as:

```json
{
  "sellerName": "INTERAUTO KRAKÓW KOCMYRZOWSKA",
  "packages": ["Pro Plus"]
}
```

The field is left off where Otomoto publishes no package, so a private sale or an offer without one does not gain an empty column.

Detail mode also returns the dealer's published services in `sellerServices` — Otomoto's own codes for what the business offers, such as financing, repairs, insurance and towing. It tells you what a rival dealer can do for a buyer, and the field is left out for private sellers and dealers who publish no services. Here is the same dealer from a real offer:

```json
{
  "sellerName": "INTERAUTO KRAKÓW KOCMYRZOWSKA",
  "sellerServices": ["financing", "repair", "tinsmith", "car-wash", "towing", "rent", "itp", "insurance"]
}
```

Detail mode also gives you the dealer's own logo image in `sellerLogoUrl`, the branding the leading Otomoto scrapers publish from their seller block. It lets you label a rival's stock with their business mark in your own catalogue. Only dealers who upload a logo get the field, so private sellers and dealers without one leave no empty column. Here is the INTERAUTO dealer from a real offer:

```json
{
  "sellerName": "INTERAUTO KRAKÓW KOCMYRZOWSKA",
  "sellerWebsite": "https://interauto.krakow.pl/oferty/",
  "sellerLogoUrl": "https://ireland.apollo.olxcdn.com/v1/files/eyJmbiI6InFtYjBoZ3d1dTZkeDEtT1RPTU9UT1BMIn0.1eJfMMjBfvbY4EmmH70UxNWS5-faj0OxK1qNXKoRfCc/image"
}
```

For sourcing comps, detail mode gives each car a `spatialHash` — a short area code taken from the seller's map pin, so you can group nearby stock without geocoding it yourself. It also gives `firstSeenAt`, the day our own history first saw that exact listing, and `daysToFirstList`, the whole days from that first sighting to the run; both are observations from our data, not a claim about when the seller published the ad, and they stay off for a car we have no record of. Here is a Kraków offer from a real detail run:

```json
{
  "title": "Skoda Kamiq 1.0 TSI Selection",
  "city": "Kraków",
  "spatialHash": "u2yhv"
}
```

And a BMW from our saved daily history on the run whose first-sighting day we could confirm:

```json
{
  "id": "6151797127",
  "title": "BMW Seria 3",
  "firstSeenAt": "2026-11-20T20:02:23Z",
  "daysToFirstList": 4
}
```

Every row also carries the listing URL, Otomoto's own price rating (below, in or above market), the photos, the full description and the equipment list. When the seller publishes them, you also get the VIN and the CEPiK history: first registration, number of owners, and the accident-free and service-record flags.

Detail mode also reports whether Otomoto's CEPiK vehicle-history check is verified. `verifiedCar: true` means the car has a verified history record — a useful trust signal when you compare a dealer's stock with a private sale. The Skoda Kamiq offer from this run came back as:

```json
{
  "title": "Skoda Kamiq 1.0 TSI Selection",
  "priceAmount": 87900,
  "city": "Kraków",
  "verifiedCar": true,
  "firstRegistrationDate": "2025-07-15"
}
```

Search cards carry the same flag where Otomoto publishes it, so you get the signal before opening the offer. The field is left off a car with no published verification flag, rather than filled with a guessed `false`.

Set `noAccident` to `true` to keep only cars the seller marked as accident-free. In detail mode, cars Otomoto reports as accident-damaged are left out of the result. Here is the Skoda Kamiq from the offer page above, returned by an accident-free search:

```json
{
  "title": "Skoda Kamiq 1.0 TSI Selection",
  "priceAmount": 87900,
  "year": 2025,
  "mileageKm": 18500,
  "noAccident": true,
  "damaged": false,
  "serviceRecord": true
}
```

Every row names the vehicle section it belongs to in `categoryCode` (`osobowe`, `dostawcze`, `ciezarowe` and so on) and `categoryName` (`Osobowe`, `Dostawcze`, `Ciężarowe`). The value comes from Otomoto's own listing address or the category published on the offer page, so a run that mixes pasted searches from several sections stays easy to sort and filter.

Each row also carries Otomoto's numeric `categoryId` (`29` for `osobowe`), the section id the leading Otomoto scrapers publish as `category.id`. It lets a catalogue key cars to Otomoto's own section instead of translating a Polish label; a dealer inventory ad that publishes no category id leaves the field out.

Each row also carries the same hierarchy a buyer sees on the page: `breadcrumbs` lists the section, make and model in order, and `breadcrumbUrls` gives the matching Otomoto category, make and model addresses. It is built from the section and the make/model Otomoto publishes on the listing, so cars from different searches file cleanly into your own catalogue. Here is the Skoda Kamiq offer from above:

```json
{
  "breadcrumbs": ["Osobowe", "Skoda", "Kamiq"],
  "breadcrumbUrls": [
    "https://www.otomoto.pl/osobowe",
    "https://www.otomoto.pl/osobowe/skoda",
    "https://www.otomoto.pl/osobowe/skoda/kamiq"
  ]
}
```

In detail mode each car also carries Otomoto's own `status`, so a dealer watching a rival's stock can tell an active ad from one Otomoto has marked inactive, sold or expired. A Skoda Kamiq from a real offer page came back as:

```json
{
  "title": "Skoda Kamiq 1.0 TSI Selection",
  "priceAmount": 87900,
  "status": "ACTIVE"
}
```

Search cards and dealer ads leave `status` out rather than filling it with a constant, because Otomoto does not publish it there.

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

Detail mode also returns the full spec sheet in `details`, each published fact with its key, label, value and the category Otomoto puts it in, plus `detailsGroups` naming those categories in the order the page shows them. It is the same grouped view a buyer sees on the offer page. Here is an excerpt of a real car's spec sheet:

```json
{
  "details": [
    { "key": "make", "label": "Marka pojazdu", "value": "Skoda", "group": "basic_information" },
    { "key": "year", "label": "Rok produkcji", "value": "2025", "group": "basic_information" },
    { "key": "fuel_type", "label": "Rodzaj paliwa", "value": "Benzyna", "group": "technical_specs" },
    { "key": "engine_capacity", "label": "Pojemność skokowa", "value": "999 cm3", "group": "technical_specs" },
    { "key": "mileage", "label": "Przebieg", "value": "18 500 km", "group": "condition_history" }
  ],
  "detailsGroups": [
    { "key": "basic_information", "label": "Podstawowe" },
    { "key": "technical_specs", "label": "Specyfikacja" },
    { "key": "condition_history", "label": "Stan i historia" },
    { "key": "financial_information", "label": "Finansowanie" }
  ]
}
```

Offers with no published spec rows have neither field; blank or hidden values such as an encrypted registration date are left out rather than returned as noise.

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

When you track competitors, it helps to know which of their cars pay for visibility. Search cards, offer pages and dealer ads carry `promotionServices`, the add-ons Otomoto publishes for the ad: `topads` buys top placement, `highlight`, `bump_up` and `ad_homepage` buy extra exposure, and `export_olx` also lists the car on OLX. A listing with no add-ons leaves the field out. Here is a real BMW search row:

```json
{
  "title": "BMW Seria 4 430i GPF xDrive M Sport",
  "priceAmount": 41400,
  "city": "Warszawa",
  "promotionServices": ["topads", "export_olx"]
}
```

A search card also tells you how much a rival invests in their Otomoto presence: it carries the dealer's advertising package and the services they publish, so a fast search run returns that competitive signal without opening the offer. Here is a BMW from a real search:

```json
{
  "title": "BMW Seria 4 430i GPF xDrive M Sport",
  "priceAmount": 41400,
  "city": "Warszawa",
  "packages": ["Ultra"],
  "sellerServices": ["financing", "repair", "fast-repair", "tinsmith"]
}
```

Otomoto also flags the ads it places at the top of a search. Where that flag is published, a row carries `topAd: true` — it is a separate signal from the paid add-ons in `promotionServices`, because a top-placed ad does not always buy a `topads` add-on. Here is a real Volkswagen Golf:

```json
{
  "title": "Volkswagen Golf",
  "priceAmount": 33500,
  "city": "Sobolew",
  "region": "Mazowieckie",
  "promotionServices": ["bump_up", "export_olx"],
  "topAd": true
}
```

Both stay off a row where Otomoto publishes no such value, so private sellers and ordinary ads get no empty column.

You can download the results as CSV, Excel or JSON, or read them through the Apify API.

If your pipeline expects the grouped JSON shape the leading Otomoto scrapers publish, set `outputShape` to `nested`: each car keeps its id, url and title at the top and groups price under `pricing`, specs under `vehicle`, city and region under `location`, dealer facts under `seller`, photos under `media`, equipment under `features` and the vehicle section under `category`. It is the same data from the same run, with no extra request and no change to what you pay.

## Sample output
Real rows from a recent run: [`sample.json`](sample.json) · [`sample.csv`](sample.csv)

```json
{
  "id": "6151889103",
  "url": "https://www.otomoto.pl/osobowe/oferta/audi-a4-avant-ID6IkHnN.html",
  "title": "Audi A4 Avant 40 TDI mHEV Quattro S Line S tronic",
  "priceAmount": 59900,
  "priceCurrency": "PLN",
  "priceIsGross": true,
  "make": "Audi",
  "makeSlug": "audi",
  "model": "A4 Avant",
  "modelSlug": "a4-avant",
  "version": "40 TDI mHEV Quattro S Line S tronic",
  "year": 2022,
  "mileageKm": 130818,
  "fuelType": "diesel",
  "gearbox": "automatic",
  "engineCapacityCm3": 1968,
  "enginePowerHp": 204,
  "city": "Warszawa",
  "region": "Mazowieckie",
  "thumbnailUrl": "https://ireland.apollo.olxcdn.com/v1/files/psqmy18sxbc01-OTOMOTOPL/image;s=640x480",
  "sellerType": "business",
  "sellerId": "18278851",
  "sellerName": "Auto",
  "sellerWebsite": "https://bydpolczynska.otomoto.pl",
  "createdAt": "2026-11-26T19:18:33Z",
  "shortDescription": null,
  "categoryId": 29,
  "categoryCode": "osobowe",
  "categoryName": "Osobowe",
  "breadcrumbs": [
    "Osobowe",
    "Audi",
    "A4 Avant"
  ],
  "breadcrumbUrls": [
    "https://www.otomoto.pl/osobowe",
    "https://www.otomoto.pl/osobowe/audi",
    "https://www.otomoto.pl/osobowe/audi/a4-avant"
  ],
  "verifiedCar": false,
  "promotionServices": [
    "topads",
    "export_olx"
  ],
  "packages": [
    "Pro Plus"
  ],
  "sellerUuid": "d0f5a189-3fa2-44eb-b500-f738b49e6e46",
  "mainFeatures": [
    "2022",
    "130818 km",
    "1968 cm3",
    "Diesel"
  ],
  "attributes": {
    "make": {
      "value": "audi",
      "label": "Audi"
    },
    "fuel_type": {
      "value": "diesel",
      "label": "Diesel"
    },
    "gearbox": {
      "value": "automatic",
      "label": "Automatyczna"
    },
    "mileage": {
      "value": "130818",
      "label": "130818 km"
    },
    "engine_capacity": {
      "value": "1968",
      "label": "1968 cm3"
    },
    "engine_power": {
      "value": "204",
      "label": "204 KM"
    },
    "model": {
      "value": "a4-avant",
      "label": "A4 Avant"
    },
    "version": {
      "value": "ver-40-tdi-mhev-quattro-s-line-s-tronic",
      "label": "40 TDI mHEV Quattro S Line S tronic"
    },
    "year": {
      "value": "2022",
      "label": "2022"
    }
  },
  "scrapedAt": "2026-11-26T21:52:05Z",
  "marketMedianPrice": 89900,
  "priceVsMarketPct": -33.37,
  "marketCohortCount": 5,
  "marketOutlier": false,
  "belowMarket": true,
  "daysListed": 0,
  "firstSeen": "2026-11-26T20:01:18Z",
  "daysTracked": 0,
  "previousPrice": 59900,
  "priceChange": 0,
  "priceChangePct": 0,
  "firstSeenAt": "2026-11-26T20:01:18Z",
  "daysToFirstList": 0
}
```

## Run it from code
Python ([apify-client](https://pypi.org/project/apify-client/)):

```python
from apify_client import ApifyClient

client = ApifyClient("<YOUR_APIFY_TOKEN>")
run = client.actor("redomic/otomoto-scraper").call(run_input={"mode": "search", "make": "BMW", "model": "Seria 3", "maxItems": 50})
for item in client.dataset(run["defaultDatasetId"]).iterate_items():
    print(item)
```

curl (waits for the run and returns the rows):

```bash
curl -X POST "https://api.apify.com/v2/acts/redomic~otomoto-scraper/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"mode": "search", "make": "BMW", "model": "Seria 3", "maxItems": 50}'
```

## All inputs, modes and pricing
See the full documentation on the Apify page: [https://apify.com/redomic/otomoto-scraper](https://apify.com/redomic/otomoto-scraper)
