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

Detail mode also tells you who is selling. Each row carries the seller's public Otomoto badges, the year they joined the site and how many cars they currently have listed, so you can tell an established dealer from a one-off seller. A dealer's row also includes `sellerAddress`, the public street address on their Otomoto profile, so you can see where a rival's stock actually sits. Here is a dealer from a real run:

```json
{
  "sellerName": "INTERAUTO KRAKÓW KOCMYRZOWSKA",
  "sellerAddress": "KOCMYRZOWSKA 1A",
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

Every row also carries the listing URL, Otomoto's own price rating (below, in or above market), the photos, the full description and the equipment list. When the seller publishes them, you also get the VIN and the CEPiK history: first registration, number of owners, and the accident-free and service-record flags.

Every row names the vehicle section it belongs to in `categoryCode` (`osobowe`, `dostawcze`, `ciezarowe` and so on) and `categoryName` (`Osobowe`, `Dostawcze`, `Ciężarowe`). The value comes from Otomoto's own listing address or the category published on the offer page, so a run that mixes pasted searches from several sections stays easy to sort and filter.

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

You can download the results as CSV, Excel or JSON, or read them through the Apify API.

If your pipeline expects the grouped JSON shape the leading Otomoto scrapers publish, set `outputShape` to `nested`: each car keeps its id, url and title at the top and groups price under `pricing`, specs under `vehicle`, city and region under `location`, dealer facts under `seller`, photos under `media`, equipment under `features` and the vehicle section under `category`. It is the same data from the same run, with no extra request and no change to what you pay.

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
  "city": "Warszawa",
  "region": "Mazowieckie",
  "thumbnailUrl": "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6InBrOHJlMzBwOWswcy1PVE9NT1RPUEwiLCJ3IjpbeyJmbiI6IndnNGducXA2eTFmLU9UT01PVE9QTCIsInMiOiIxNiIsImEiOiIwIiwicCI6IjEwLC0xMCJ9XX0.PuaacUqt_B1WSJSOVuOiel9xKMqpVLf3rAXFlZkU9NU/image",
  "sellerType": "business",
  "sellerId": "18253685",
  "sellerName": "Pan America",
  "sellerWebsite": "https://panamericaauto.pl/",
  "cepikVerified": false,
  "createdAt": "2026-11-09T15:33:16Z",
  "shortDescription": "BMW 330i xDrive 2020 | 2.0 Turbo | Automat | M Sport Shadow",
  "categoryCode": "osobowe",
  "categoryName": "Osobowe",
  "promotionServices": [
    "export_olx",
    "topads",
    "bump_up"
  ],
  "sellerUuid": "a99f9b81-027b-415c-a321-85390571359c",
  "mainFeatures": [
    "2020",
    "31 500 km",
    "1 998 cm3",
    "Benzyna"
  ],
  "attributes": {
    "has_vin": {
      "value": "1",
      "label": "Tak"
    },
    "is_imported_car": {
      "value": "1",
      "label": "Tak"
    },
    "has_registration": {
      "value": "1",
      "label": "Tak"
    },
    "catalog_urn": {
      "value": "orn:otomotopl:catalog:eurotax_de-10136583",
      "label": "orn:otomotopl:catalog:eurotax_de-10136583"
    },
    "make": {
      "value": "bmw",
      "label": "BMW"
    },
    "model": {
      "value": "seria-3",
      "label": "Seria 3"
    },
    "version": {
      "value": "ver-330i-edition-m-sport-shadow",
      "label": "330i Edition M Sport Shadow"
    },
    "generation": {
      "value": "gen-g20-2019",
      "label": "G20/G21 (2019-)"
    },
    "year": {
      "value": "2020",
      "label": "2020"
    },
    "mileage": {
      "value": "31500",
      "label": "31 500 km"
    },
    "engine_capacity": {
      "value": "1998",
      "label": "1 998 cm3"
    },
    "fuel_type": {
      "value": "petrol",
      "label": "Benzyna"
    },
    "engine_power": {
      "value": "252",
      "label": "252 KM"
    },
    "gearbox": {
      "value": "automatic",
      "label": "Automatyczna"
    },
    "transmission": {
      "value": "rear-wheel",
      "label": "Na tylne koła"
    },
    "particle_filter": {
      "value": "1",
      "label": "Tak"
    },
    "extra_urban_consumption": {
      "value": "5.5",
      "label": "5.5 l/100km"
    },
    "bluetooth_interface": {
      "value": "1",
      "label": "Tak"
    },
    "urban_consumption": {
      "value": "7.3",
      "label": "7.3 l/100km"
    },
    "damaged": {
      "value": "0",
      "label": "Nie"
    },
    "body_type": {
      "value": "sedan",
      "label": "Sedan"
    },
    "co2_emissions": {
      "value": "142",
      "label": "142 g/km"
    },
    "door_count": {
      "value": "5",
      "label": "5"
    },
    "nr_seats": {
      "value": "5",
      "label": "5"
    },
    "color": {
      "value": "grey",
      "label": "Szary"
    },
    "radio": {
      "value": "1",
      "label": "Tak"
    },
    "rhd": {
      "value": "0",
      "label": "Po lewej"
    },
    "hands_free_system": {
      "value": "1",
      "label": "Tak"
    },
    "usb_in": {
      "value": "1",
      "label": "Tak"
    },
    "country_origin": {
      "value": "usa",
      "label": "Stany Zjednoczone"
    },
    "registered": {
      "value": "0",
      "label": "Nie"
    },
    "original_owner": {
      "value": "0",
      "label": "Nie"
    },
    "no_accident": {
      "value": "1",
      "label": "Tak"
    },
    "service_record": {
      "value": "1",
      "label": "Tak"
    },
    "historical_vehicle": {
      "value": "0",
      "label": "Nie"
    },
    "tuning": {
      "value": "0",
      "label": "Nie"
    },
    "approval_for_goods": {
      "value": "0",
      "label": "Nie"
    },
    "internet_access": {
      "value": "1",
      "label": "Tak"
    },
    "air_conditioning_type": {
      "value": "dualzone-automatic-climate-control",
      "label": "Klimatyzacja automatyczna: dwustrefowa"
    },
    "upholstery_type": {
      "value": "leather-upholstery",
      "label": "Tapicerka skórzana"
    },
    "heated_seat_driver": {
      "value": "1",
      "label": "Tak"
    },
    "heated_seat_passenger": {
      "value": "1",
      "label": "Tak"
    },
    "sport_seats_front": {
      "value": "1",
      "label": "Tak"
    },
    "armrest_front": {
      "value": "1",
      "label": "Tak"
    },
    "armrest_rear": {
      "value": "1",
      "label": "Tak"
    },
    "leather_steering_wheel": {
      "value": "1",
      "label": "Tak"
    },
    "sports_steering_wheel": {
      "value": "1",
      "label": "Tak"
    },
    "steering_wheel_with_radio_operation": {
      "value": "1",
      "label": "Tak"
    },
    "multi_functional_steering_wheel": {
      "value": "1",
      "label": "Tak"
    },
    "leather_gearshifterswitch": {
      "value": "1",
      "label": "Tak"
    },
    "digital_key": {
      "value": "1",
      "label": "Tak"
    },
    "keyless_engine_start": {
      "value": "1",
      "label": "Tak"
    },
    "rain_sensor": {
      "value": "1",
      "label": "Tak"
    },
    "power_windows_front": {
      "value": "1",
      "label": "Tak"
    },
    "power_windows_rear": {
      "value": "1",
      "label": "Tak"
    },
    "cruisecontrol_type": {
      "value": "cruise-control",
      "label": "Tempomat"
    },
    "headlight_lamp_type": {
      "value": "led-front-dim-light",
      "label": "Lampy przednie w technologii LED"
    },
    "park_distance_control_rear": {
      "value": "1",
      "label": "Tak"
    },
    "door_mirrors_heated": {
      "value": "1",
      "label": "Tak"
    },
    "speed_limiter": {
      "value": "1",
      "label": "Tak"
    },
    "traction_control": {
      "value": "1",
      "label": "Tak"
    },
    "automatic_dimlight_activation": {
      "value": "1",
      "label": "Tak"
    },
    "led_daytime_running_lights": {
      "value": "1",
      "label": "Tak"
    },
    "led_fog_lamps": {
      "value": "1",
      "label": "Tak"
    },
    "led_rear_lights": {
      "value": "1",
      "label": "Tak"
    },
    "led_interior_lighting": {
      "value": "1",
      "label": "Tak"
    },
    "startstop_system": {
      "value": "1",
      "label": "Tak"
    },
    "sport_suspension": {
      "value": "1",
      "label": "Tak"
    },
    "antilock_brake_system": {
      "value": "1",
      "label": "Tak"
    },
    "esp": {
      "value": "1",
      "label": "Tak"
    },
    "power_assisted_brakes": {
      "value": "1",
      "label": "Tak"
    },
    "automatic_emergency_call": {
      "value": "1",
      "label": "Tak"
    },
    "driver_airbag": {
      "value": "1",
      "label": "Tak"
    },
    "passenger_airbag": {
      "value": "1",
      "label": "Tak"
    },
    "head_airbags_front": {
      "value": "1",
      "label": "Tak"
    },
    "side_airbag_driver_and_passenger": {
      "value": "1",
      "label": "Tak"
    },
    "head_airbags_rear": {
      "value": "1",
      "label": "Tak"
    },
    "child_seat_fixation": {
      "value": "1",
      "label": "Tak"
    },
    "version_label": {
      "value": "330i Edition M Sport Shadow",
      "label": "330i Edition M Sport Shadow"
    },
    "autorenew": {
      "value": "0",
      "label": "Nie"
    },
    "new_used": {
      "value": "used",
      "label": "Używany"
    },
    "with_vin": {
      "value": "1"
    }
  },
  "countryOrigin": "us",
  "bodyType": "sedan",
  "damaged": false,
  "leaseTransfer": false,
  "noAccident": true,
  "serviceRecord": true,
  "originalOwner": false,
  "registered": false,
  "newUsed": "used",
  "condition": "used",
  "conditionLabel": "Używany",
  "generation": "BMW Seria 3 Benzyna 2012-2019",
  "driveType": "rear-wheel",
  "driveTypeLabel": "Na tylne koła",
  "color": "gray",
  "colorLabel": "Szary",
  "doorCount": 5,
  "nrSeats": 5,
  "lat": 52.23614,
  "lng": 21.00817,
  "sellerInventoryUrl": "https://panamerica.otomoto.pl/inventory",
  "sellerAddress": "Witolda Pileckiego 59",
  "sellerWorkingHours": [
    {
      "day": "Mon",
      "openAt": "10:00",
      "closeAt": "18:00"
    },
    {
      "day": "Tue",
      "openAt": "10:00",
      "closeAt": "18:00"
    },
    {
      "day": "Wed",
      "openAt": "10:00",
      "closeAt": "18:00"
    },
    {
      "day": "Thu",
      "openAt": "10:00",
      "closeAt": "18:00"
    },
    {
      "day": "Fri",
      "openAt": "10:00",
      "closeAt": "18:00"
    },
    {
      "day": "Sat",
      "openAt": "10:00",
      "closeAt": "18:00"
    }
  ],
  "sellerBadges": [
    "Firma",
    "Bardzo sprawnie odpowiada",
    "Sprzedający na OTOMOTO od 2026"
  ],
  "sellerSinceYear": 2026,
  "sellerActiveAdCount": 89,
  "description": "BMW 330i xDrive 2020 | 2.0 Turbo | Automat | Napęd 4x4 / M Sport Shadow\nNa sprzedaż BMW 330i xDrive z 2020 roku. Model G20 wyposażony w dynamiczny silnik benzynowy 2.0 TwinPower Turbo o mocy 258 KM, automatyczną skrzynię biegów oraz inteligentny napęd na cztery koła xDrive.\nDane techniczne:\nRok produkcji: 2020\nSilnik: 2.0 benzyna TwinPower Turbo (B48)\nMoc: 258 KM\nSkrzynia biegów: automatyczna Steptronic\nNapęd: xDrive (4x4)\nGeneracja: G20\nBMW 330i zapewnia świetne osiągi, precyzyjne prowadzenie i wysoki komfort jazdy. Połączenie mocnego silnika benzynowego z napędem xDrive gwarantuje doskonałą przyczepność i pewność prowadzenia w każdych warunkach.\nMożliwość sprowadzenia samochodu z USA pod dom.\nKażdy samochód przed zakupem przechodzi szczegółową weryfikację techniczną oraz kontrolę historii pojazdu. Sprawdzany jest stan mechaniczny, lakier, elektronika oraz wyposażenie.\nDodatkowo oferujemy możliwość wykonania szczegółowej inspekcji pojazdu przed zakupem, przeprowadzanej przez naszego rzeczoznawcę z ponad 15-letnim doświadczeniem w ocenie samochodów.\nW ramach inspekcji otrzymujesz:\nokoło 1 godziny nagrań wideo prezentujących samochód z każdej strony,\nponad 100 zdjęć w wysokiej jakości,\npełny odczyt wszystkich sterowników za pomocą komputera diagnostycznego,\npomiar grubości lakieru na każdym elemencie nadwozia profesjonalnym miernikiem lakieru,\nkontrolę silnika, skrzyni biegów, zawieszenia, podwozia oraz układu hamulcowego,\nsprawdzenie szczelności silnika i skrzyni biegów pod kątem wycieków,\nweryfikację elementów konstrukcyjnych oraz ewentualnych napraw powypadkowych,\nszczegółowy raport z oględzin wraz z rekomendacją dotyczącą zakupu.\nDzięki takiej weryfikacji dokładnie wiesz, w jakim stanie znajduje się samochód jeszcze przed jego zakupem i możesz podjąć świadomą decyzję, eliminując ryzyko nieprzewidzianych kosztów po odbiorze pojazdu.\nImport samochodu może pozwolić zaoszczędzić nawet 20–30% względem podobnych egzemplarzy na rynku polskim.",
  "photos": [
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6InBrOHJlMzBwOWswcy1PVE9NT1RPUEwiLCJ3IjpbeyJmbiI6IndnNGducXA2eTFmLU9UT01PVE9QTCIsInMiOiIxNiIsImEiOiIwIiwicCI6IjEwLC0xMCJ9XX0.PuaacUqt_B1WSJSOVuOiel9xKMqpVLf3rAXFlZkU9NU/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6ImN0NDU4aGVnb2ltbi1PVE9NT1RPUEwiLCJ3IjpbeyJmbiI6IndnNGducXA2eTFmLU9UT01PVE9QTCIsInMiOiIxNiIsImEiOiIwIiwicCI6IjEwLC0xMCJ9XX0.qij5fhRdDn11GL3Z3U0sMLrU36vIseqju-xmXeQO5sU/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6InF4MjV0bnVwdmw3bzMtT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.lqwypuvjJaMrFo5DB1BKIV_LAcLA4Nc5avze9U-Vb-c/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6Im5tZW0yMHN5c2J1YzItT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.Jg814FlAL37kOApxOiZ-gH2ECEzO7OFojSr0c7AfmSw/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6InZpY2t6eGk1MjdjNC1PVE9NT1RPUEwiLCJ3IjpbeyJmbiI6IndnNGducXA2eTFmLU9UT01PVE9QTCIsInMiOiIxNiIsImEiOiIwIiwicCI6IjEwLC0xMCJ9XX0.XHBejhGXS5EL8MuLuurR6kyKQgYDzxe2JmBMLfGf3YY/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6IjFmM21yMmM1bWZidjMtT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.CbIw-fQoTSYy7RSAM_MnXPNt6dbQVGNJa4PjUv5oezs/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6ImhkaTRvMDR3MGp4cjMtT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.A1FWyDoRI1QBBoXoJdxw9fIewZE8JYvD2_bSpuSRstI/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6ImI4OG93bW13anRtODMtT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.Ajxt5dwgOE8wP8JhyHBFtC8MU13BIeHMDTfMu1_nOjU/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6IjRwOG96NTJ2NjR0NS1PVE9NT1RPUEwiLCJ3IjpbeyJmbiI6IndnNGducXA2eTFmLU9UT01PVE9QTCIsInMiOiIxNiIsImEiOiIwIiwicCI6IjEwLC0xMCJ9XX0.tYA3s7FMN7DKq9064LNJIQGxY2RY5W1SrN5g5qj3X58/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6ImY3dDN0c3J2dG9iYTItT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.JIPs9L7Hhs4bxxt9-vReVdPTI8QF3KreREBBZnjl9Pw/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6ImJrZTlrZnVkOWc0dDEtT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.BEXzHB77V5AXa-0550vIOXcbJMM3PKYHaHrjJhvF2c4/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6Im1lOXJieGE4dzlzdS1PVE9NT1RPUEwiLCJ3IjpbeyJmbiI6IndnNGducXA2eTFmLU9UT01PVE9QTCIsInMiOiIxNiIsImEiOiIwIiwicCI6IjEwLC0xMCJ9XX0.Y3035wZuhrA-GOsCJF20I39gFVygeT7rwra57AwDYbc/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6ImRsZ3NoM2NoazhhMzMtT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.eblkeisRrzk9fUskPng-WVNhKtvCAB5aaEH9v39VGC4/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6IndlcDJxY2R5OXdrNzMtT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.Sh-7CPTtFAke1WjvLEAbAdD1iMIenCCK_5ktMOHBnYs/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6Im4zaGc1bnRpZTkxdTEtT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.qr3Qcy62E9MKRSisQKsLlHkOh2Olls0efdJEW3smTIw/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6ImZmOGdqdjhyaXp6YTMtT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.2H2itTNo96lISCksYJ_Hf9uQSBTKwINFu4Es4MDcCAk/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6ImpoaHZlejhyaXdwdjEtT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.EDdqVk_9WS0aUM21Py65zmGyaPGAOTqEmJEphFL9c78/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6InlybHRwenJ4eHRmaDItT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.VlvkKhiT-HFRw4oQdOidZcz4mbES24cqgPMSPf9VVL4/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6ImlxbnFpc3N4aGUwZTEtT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.6k6vzM_GkaYqWcxvXLQnebyDpSrcoMvBjm_KpcjJ8Rw/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6IjAwMWdmbjE0M2poaDEtT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.00CnRz_Yu5V01m0p2nAK6JmfP4-wfQctfUVycmHPjX0/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6InN1aXI5Y3pveDhoYy1PVE9NT1RPUEwiLCJ3IjpbeyJmbiI6IndnNGducXA2eTFmLU9UT01PVE9QTCIsInMiOiIxNiIsImEiOiIwIiwicCI6IjEwLC0xMCJ9XX0._awz-JwCvAUajNv5DAznrfP7-4fQJwdJDm-tCPOz0aU/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6Im5lbDdpbGpodWo0aDItT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.L7kNuYVfeSU4EA8MmQJYtxZPWljICy1gPIk_7JknbCg/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6Im1rcTE0NjV4OHhxNDEtT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.K780_jvBfed1h0wsqF8ASnpQTawCiivmBVSK_jHZe1A/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6IjJjMzdoZjk3dHZmai1PVE9NT1RPUEwiLCJ3IjpbeyJmbiI6IndnNGducXA2eTFmLU9UT01PVE9QTCIsInMiOiIxNiIsImEiOiIwIiwicCI6IjEwLC0xMCJ9XX0.WCarCgmnGSvug1JMG8ck0W-llVshcw9pN1TOHjqdce8/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6IjJhZTJ4Njc4MHN6ajMtT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.9XKWqJ_BYd4d6Y70aT49M7AKWamm9BDbgGvU2qNd4tg/image"
  ],
  "imageCount": 25,
  "equipment": [
    "Interfejs Bluetooth",
    "Radio",
    "Zestaw głośnomówiący",
    "Gniazdo USB",
    "Dostęp do internetu",
    "Klimatyzacja automatyczna: dwustrefowa",
    "Tapicerka skórzana",
    "Podgrzewany fotel kierowcy",
    "Podgrzewany fotel pasażera",
    "Sportowe fotele - przód",
    "Podłokietniki - przód",
    "Podłokietniki - tył",
    "Kierownica skórzana",
    "Kierownica sportowa",
    "Kierownica ze sterowaniem radia",
    "Kierownica wielofunkcyjna",
    "Dźwignia zmiany biegów wykończona skórą",
    "Cyfrowy kluczyk",
    "Uruchamianie silnika bez użycia kluczyków",
    "Czujnik deszczu",
    "Elektryczne szyby przednie",
    "Elektryczne szyby tylne",
    "Tempomat",
    "Lampy przednie w technologii LED",
    "Kontrola odległości z tyłu (przy parkowaniu)",
    "Podgrzewane lusterka boczne",
    "Ogranicznik prędkości",
    "Kontrola trakcji",
    "Czujnik zmierzchu",
    "Światła do jazdy dziennej diodowe LED",
    "Lampy przeciwmgielne w technologii LED",
    "Lampy tylne w technologii LED",
    "Oświetlenie wnętrza LED",
    "System Start/Stop",
    "Filtr cząstek stałych",
    "Zawieszenie sportowe",
    "ABS",
    "ESP",
    "System wspomagania hamowania",
    "System powiadamiania o wypadku",
    "Poduszka powietrzna kierowcy",
    "Poduszka powietrzna pasażera",
    "Kurtyny powietrzne - przód",
    "Boczne poduszki powietrzne - przód",
    "Kurtyny powietrzne - tył",
    "Isofix (punkty mocowania fotelika dziecięcego)"
  ],
  "equipmentGroups": [
    {
      "label": "Audio i multimedia",
      "values": [
        {
          "label": "Interfejs Bluetooth",
          "key": "bluetooth_interface"
        },
        {
          "label": "Radio",
          "key": "radio"
        },
        {
          "label": "Zestaw głośnomówiący",
          "key": "hands_free_system"
        },
        {
          "label": "Gniazdo USB",
          "key": "usb_in"
        },
        {
          "label": "Dostęp do internetu",
          "key": "internet_access"
        }
      ],
      "key": "audio_and_computing"
    },
    {
      "label": "Komfort i dodatki",
      "values": [
        {
          "label": "Klimatyzacja automatyczna: dwustrefowa",
          "key": "air_conditioning_type"
        },
        {
          "label": "Tapicerka skórzana",
          "key": "upholstery_type"
        },
        {
          "label": "Podgrzewany fotel kierowcy",
          "key": "heated_seat_driver"
        },
        {
          "label": "Podgrzewany fotel pasażera",
          "key": "heated_seat_passenger"
        },
        {
          "label": "Sportowe fotele - przód",
          "key": "sport_seats_front"
        },
        {
          "label": "Podłokietniki - przód",
          "key": "armrest_front"
        },
        {
          "label": "Podłokietniki - tył",
          "key": "armrest_rear"
        },
        {
          "label": "Kierownica skórzana",
          "key": "leather_steering_wheel"
        },
        {
          "label": "Kierownica sportowa",
          "key": "sports_steering_wheel"
        },
        {
          "label": "Kierownica ze sterowaniem radia",
          "key": "steering_wheel_with_radio_operation"
        },
        {
          "label": "Kierownica wielofunkcyjna",
          "key": "multi_functional_steering_wheel"
        },
        {
          "label": "Dźwignia zmiany biegów wykończona skórą",
          "key": "leather_gearshifterswitch"
        },
        {
          "label": "Cyfrowy kluczyk",
          "key": "digital_key"
        },
        {
          "label": "Uruchamianie silnika bez użycia kluczyków",
          "key": "keyless_engine_start"
        },
        {
          "label": "Czujnik deszczu",
          "key": "rain_sensor"
        },
        {
          "label": "Elektryczne szyby przednie",
          "key": "power_windows_front"
        },
        {
          "label": "Elektryczne szyby tylne",
          "key": "power_windows_rear"
        }
      ],
      "key": "comfort_and_addons"
    },
    {
      "label": "Systemy wspomagania kierowcy",
      "values": [
        {
          "label": "Tempomat",
          "key": "cruisecontrol_type"
        },
        {
          "label": "Lampy przednie w technologii LED",
          "key": "headlight_lamp_type"
        },
        {
          "label": "Kontrola odległości z tyłu (przy parkowaniu)",
          "key": "park_distance_control_rear"
        },
        {
          "label": "Podgrzewane lusterka boczne",
          "key": "door_mirrors_heated"
        },
        {
          "label": "Ogranicznik prędkości",
          "key": "speed_limiter"
        },
        {
          "label": "Kontrola trakcji",
          "key": "traction_control"
        },
        {
          "label": "Czujnik zmierzchu",
          "key": "automatic_dimlight_activation"
        },
        {
          "label": "Światła do jazdy dziennej diodowe LED",
          "key": "led_daytime_running_lights"
        },
        {
          "label": "Lampy przeciwmgielne w technologii LED",
          "key": "led_fog_lamps"
        },
        {
          "label": "Lampy tylne w technologii LED",
          "key": "led_rear_lights"
        },
        {
          "label": "Oświetlenie wnętrza LED",
          "key": "led_interior_lighting"
        },
        {
          "label": "System Start/Stop",
          "key": "startstop_system"
        }
      ],
      "key": "electronics_and_driver_assistance"
    },
    {
      "label": "Osiągi i tuning",
      "values": [
        {
          "label": "Filtr cząstek stałych",
          "key": "particle_filter"
        },
        {
          "label": "Zawieszenie sportowe",
          "key": "sport_suspension"
        }
      ],
      "key": "performance_and_tuning"
    },
    {
      "label": "Bezpieczeństwo",
      "values": [
        {
          "label": "ABS",
          "key": "antilock_brake_system"
        },
        {
          "label": "ESP",
          "key": "esp"
        },
        {
          "label": "System wspomagania hamowania",
          "key": "power_assisted_brakes"
        },
        {
          "label": "System powiadamiania o wypadku",
          "key": "automatic_emergency_call"
        },
        {
          "label": "Poduszka powietrzna kierowcy",
          "key": "driver_airbag"
        },
        {
          "label": "Poduszka powietrzna pasażera",
          "key": "passenger_airbag"
        },
        {
          "label": "Kurtyny powietrzne - przód",
          "key": "head_airbags_front"
        },
        {
          "label": "Boczne poduszki powietrzne - przód",
          "key": "side_airbag_driver_and_passenger"
        },
        {
          "label": "Kurtyny powietrzne - tył",
          "key": "head_airbags_rear"
        },
        {
          "label": "Isofix (punkty mocowania fotelika dziecięcego)",
          "key": "child_seat_fixation"
        }
      ],
      "key": "security"
    }
  ],
  "details": [
    {
      "key": "make",
      "label": "Marka pojazdu",
      "value": "BMW",
      "group": "basic_information"
    },
    {
      "key": "model",
      "label": "Model pojazdu",
      "value": "Seria 3",
      "group": "basic_information"
    },
    {
      "key": "version",
      "label": "Wersja",
      "value": "330i Edition M Sport Shadow",
      "group": "basic_information"
    },
    {
      "key": "color",
      "label": "Kolor",
      "value": "Szary",
      "group": "basic_information"
    },
    {
      "key": "door_count",
      "label": "Liczba drzwi",
      "value": "5",
      "group": "basic_information"
    },
    {
      "key": "nr_seats",
      "label": "Liczba miejsc",
      "value": "5",
      "group": "basic_information"
    },
    {
      "key": "year",
      "label": "Rok produkcji",
      "value": "2020",
      "group": "basic_information"
    },
    {
      "key": "generation",
      "label": "Generacja",
      "value": "G20/G21 (2019-)",
      "group": "basic_information"
    },
    {
      "key": "fuel_type",
      "label": "Rodzaj paliwa",
      "value": "Benzyna",
      "group": "technical_specs"
    },
    {
      "key": "engine_capacity",
      "label": "Pojemność skokowa",
      "value": "1 998 cm3",
      "group": "technical_specs"
    },
    {
      "key": "engine_power",
      "label": "Moc",
      "value": "252 KM",
      "group": "technical_specs"
    },
    {
      "key": "body_type",
      "label": "Typ nadwozia",
      "value": "Sedan",
      "group": "technical_specs"
    },
    {
      "key": "gearbox",
      "label": "Skrzynia biegów",
      "value": "Automatyczna",
      "group": "technical_specs"
    },
    {
      "key": "transmission",
      "label": "Napęd",
      "value": "Na tylne koła",
      "group": "technical_specs"
    },
    {
      "key": "co2_emissions",
      "label": "Emisja CO2",
      "value": "142 g/km",
      "group": "technical_specs"
    },
    {
      "key": "urban_consumption",
      "label": "Spalanie W Mieście",
      "value": "7.3 l/100km",
      "group": "technical_specs"
    },
    {
      "key": "extra_urban_consumption",
      "label": "Spalanie Poza Miastem",
      "value": "5.5 l/100km",
      "group": "technical_specs"
    },
    {
      "key": "country_origin",
      "label": "Kraj pochodzenia",
      "value": "Stany Zjednoczone",
      "group": "condition_history"
    },
    {
      "key": "mileage",
      "label": "Przebieg",
      "value": "31 500 km",
      "group": "condition_history"
    },
    {
      "key": "new_used",
      "label": "Stan",
      "value": "Używany",
      "group": "condition_history"
    },
    {
      "key": "no_accident",
      "label": "Bezwypadkowy",
      "value": "Tak",
      "group": "condition_history"
    },
    {
      "key": "has_registration",
      "label": "Ma numer rejestracyjny",
      "value": "Tak",
      "group": "condition_history"
    },
    {
      "key": "service_record",
      "label": "Serwisowany w ASO",
      "value": "Tak",
      "group": "condition_history"
    }
  ],
  "detailsGroups": [
    {
      "key": "basic_information",
      "label": "Podstawowe"
    },
    {
      "key": "technical_specs",
      "label": "Specyfikacja"
    },
    {
      "key": "condition_history",
      "label": "Stan i historia"
    },
    {
      "key": "financial_information",
      "label": "Finansowanie"
    },
    {
      "key": "ev_specs",
      "label": "ev_specs"
    }
  ],
  "updatedAt": "2026-11-09T15:33:16Z",
  "originalCreatedAt": "2026-11-02T14:04:41Z",
  "status": "ACTIVE",
  "scrapedAt": "2026-11-21T00:57:57Z",
  "marketMedianPrice": 74500,
  "priceVsMarketPct": -20.81,
  "marketCohortCount": 9,
  "marketOutlier": false,
  "belowMarket": true,
  "daysListed": 11,
  "firstSeen": "2026-11-17T16:12:38Z",
  "daysTracked": 3,
  "previousPrice": 59000,
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
