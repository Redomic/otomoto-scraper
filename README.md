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

The badges and address are left out when the seller publishes none, and private sellers stay unnamed.

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

## Sample output
Real rows from a recent run: [`sample.json`](sample.json) · [`sample.csv`](sample.csv)

```json
{
  "id": "6151758170",
  "url": "https://www.otomoto.pl/osobowe/oferta/bmw-seria-3-ID6Ik9jY.html",
  "title": "BMW Seria 3 320i xDrive M Sport",
  "priceAmount": 53600,
  "priceCurrency": "PLN",
  "priceIsGross": true,
  "make": "BMW",
  "makeSlug": "bmw",
  "model": "Seria 3",
  "modelSlug": "seria-3",
  "version": "320i xDrive M Sport",
  "year": 2018,
  "mileageKm": 64846,
  "fuelType": "petrol",
  "gearbox": "automatic",
  "engineCapacityCm3": 1998,
  "enginePowerHp": 184,
  "city": "Warszawa",
  "region": "Mazowieckie",
  "thumbnailUrl": "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6ImI1Z2RtM3p6b3drcTEtT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.sbNdI0QEwDyjwsiIfS-F-kwxu9yYNtvLLZGjJ7qqyD4/image",
  "sellerType": "business",
  "sellerId": "16260395",
  "sellerName": "WINCARS",
  "sellerWebsite": "https://wincars.pl/",
  "cepikVerified": false,
  "createdAt": "2026-11-18T15:40:48Z",
  "shortDescription": "Idealny stan | Gwarancja | Mały przebieg",
  "promotionServices": [
    "topads",
    "export_olx"
  ],
  "mainFeatures": [
    "2018",
    "64 846 km",
    "1 998 cm3",
    "Benzyna"
  ],
  "attributes": {
    "air_condition_rear": {
      "value": "1",
      "label": "Tak"
    },
    "damaged": {
      "value": "0",
      "label": "Nie"
    },
    "esp": {
      "value": "1",
      "label": "Tak"
    },
    "apple_carplay": {
      "value": "1",
      "label": "Tak"
    },
    "adaptive_light": {
      "value": "1",
      "label": "Tak"
    },
    "city_emergency_brake_assist": {
      "value": "1",
      "label": "Tak"
    },
    "door_mirrors_heated": {
      "value": "1",
      "label": "Tak"
    },
    "electric_parking_brake": {
      "value": "1",
      "label": "Tak"
    },
    "heated_seat_passenger": {
      "value": "1",
      "label": "Tak"
    },
    "internet_access": {
      "value": "1",
      "label": "Tak"
    },
    "intersection_assist": {
      "value": "1",
      "label": "Tak"
    },
    "daytime_running_lights": {
      "value": "1",
      "label": "Tak"
    },
    "cruisecontrol_type": {
      "value": "adaptive-cruise-control",
      "label": "Tempomat adaptacyjny ACC"
    },
    "knee_airbag_passenger": {
      "value": "1",
      "label": "Tak"
    },
    "navigation_system": {
      "value": "1",
      "label": "Tak"
    },
    "park_distance_control_front": {
      "value": "1",
      "label": "Tak"
    },
    "power_assisted_brakes": {
      "value": "1",
      "label": "Tak"
    },
    "rain_sensor": {
      "value": "1",
      "label": "Tak"
    },
    "brake_assist": {
      "value": "1",
      "label": "Tak"
    },
    "gearbox": {
      "value": "automatic",
      "label": "Automatyczna"
    },
    "no_accident": {
      "value": "1",
      "label": "Tak"
    },
    "rear_view_camera": {
      "value": "1",
      "label": "Tak"
    },
    "sports_steering_wheel": {
      "value": "1",
      "label": "Tak"
    },
    "top_remote_controlled": {
      "value": "1",
      "label": "Tak"
    },
    "usb_in": {
      "value": "1",
      "label": "Tak"
    },
    "360_view_camera": {
      "value": "1",
      "label": "Tak e"
    },
    "front_seat_with_massage": {
      "value": "1",
      "label": "Tak"
    },
    "leather_steering_wheel": {
      "value": "1",
      "label": "Tak"
    },
    "park_distance_control_rear": {
      "value": "1",
      "label": "Tak"
    },
    "transmission": {
      "value": "all-wheel-permanent",
      "label": "4x4 (stały)"
    },
    "central_airbag_driver_and_passenger": {
      "value": "1",
      "label": "Tak"
    },
    "front_airbags_for_rear_seats": {
      "value": "1",
      "label": "Tak"
    },
    "led_fog_lamps": {
      "value": "1",
      "label": "Tak"
    },
    "particle_filter": {
      "value": "1",
      "label": "Tak"
    },
    "rhd": {
      "value": "0",
      "label": "Po lewej"
    },
    "traction_control": {
      "value": "1",
      "label": "Tak"
    },
    "traffic_sign_recognition": {
      "value": "1",
      "label": "Tak"
    },
    "door_mirror_electrically_adjustable_in_general": {
      "value": "1",
      "label": "Tak"
    },
    "fog_lamps": {
      "value": "1",
      "label": "Tak"
    },
    "rear_pre_crash_system": {
      "value": "1",
      "label": "Tak"
    },
    "towbar": {
      "value": "1",
      "label": "Tak"
    },
    "independent_vehicle_heater": {
      "value": "1",
      "label": "Tak"
    },
    "original_owner": {
      "value": "1",
      "label": "Tak"
    },
    "steering_wheel_electrically_adjustable": {
      "value": "1",
      "label": "Tak"
    },
    "ventilated_rear_seat": {
      "value": "1",
      "label": "Tak"
    },
    "active_lane_control_assistant": {
      "value": "1",
      "label": "Tak"
    },
    "driver_seat_electrically_adjustable": {
      "value": "1",
      "label": "Tak"
    },
    "dynamic_directional_lights": {
      "value": "1",
      "label": "Tak"
    },
    "service_record": {
      "value": "1",
      "label": "Tak"
    },
    "tinted_rear_windows": {
      "value": "1",
      "label": "Tak"
    },
    "urban_consumption": {
      "value": "7.5",
      "label": "7.5 l/100km"
    },
    "catalog_urn": {
      "value": "orn:otomotopl:catalog:eurotax-116175",
      "label": "orn:otomotopl:catalog:eurotax-116175"
    },
    "ceramic_composite_brakes": {
      "value": "1",
      "label": "Tak"
    },
    "collision_warning_system": {
      "value": "1",
      "label": "Tak"
    },
    "limited_slip_differential_in_general": {
      "value": "1",
      "label": "Tak"
    },
    "seat_belt_airbag_rear": {
      "value": "1",
      "label": "Tak"
    },
    "colour_type": {
      "value": "metallic",
      "label": "Metalik"
    },
    "hill_descent_control": {
      "value": "1",
      "label": "Tak"
    },
    "is_imported_car": {
      "value": "1",
      "label": "Tak"
    },
    "leather_gearshifterswitch": {
      "value": "1",
      "label": "Tak"
    },
    "side_airbag_driver_and_passenger": {
      "value": "1",
      "label": "Tak"
    },
    "side_airbags_rear": {
      "value": "1",
      "label": "Tak"
    },
    "windscreen_heating": {
      "value": "1",
      "label": "Tak"
    },
    "wireless_device_charging": {
      "value": "1",
      "label": "Tak"
    },
    "active_driver_conditioning_monitoring": {
      "value": "1",
      "label": "Tak"
    },
    "air_conditioning_type": {
      "value": "4-or-more-zone-automatic-climate-control",
      "label": "Klimatyzacja automatyczna: co najmniej 4 strefy"
    },
    "engine_capacity": {
      "value": "1998",
      "label": "1 998 cm3"
    },
    "follow_me_home": {
      "value": "1",
      "label": "Tak"
    },
    "door_count": {
      "value": "5",
      "label": "5"
    },
    "hands_free_system": {
      "value": "1",
      "label": "Tak"
    },
    "mileage": {
      "value": "64846",
      "label": "64 846 km"
    },
    "ventilated_front_seat": {
      "value": "1",
      "label": "Tak"
    },
    "year": {
      "value": "2018",
      "label": "2018"
    },
    "has_registration": {
      "value": "1",
      "label": "Tak"
    },
    "hill_holder": {
      "value": "1",
      "label": "Tak"
    },
    "lane_control_assistant": {
      "value": "1",
      "label": "Tak"
    },
    "passenger_airbag": {
      "value": "1",
      "label": "Tak"
    },
    "radio": {
      "value": "1",
      "label": "Tak"
    },
    "voice_control_for_vehicle_functions": {
      "value": "1",
      "label": "Tak"
    },
    "active_speed_sign_recognition": {
      "value": "1",
      "label": "Tak"
    },
    "autonomous_parking_system": {
      "value": "1",
      "label": "Tak"
    },
    "distance_control": {
      "value": "1",
      "label": "Tak"
    },
    "multi_functional_steering_wheel": {
      "value": "1",
      "label": "Tak"
    },
    "side_airbag_driver": {
      "value": "1",
      "label": "Tak"
    },
    "steering_wheel_with_gear_switches": {
      "value": "1",
      "label": "Tak"
    },
    "automatic_dimlight_activation": {
      "value": "1",
      "label": "Tak"
    },
    "automatic_emergency_call": {
      "value": "1",
      "label": "Tak"
    },
    "has_vin": {
      "value": "1",
      "label": "Tak"
    },
    "passenger_seat_electrically_adjustable": {
      "value": "1",
      "label": "Tak"
    },
    "rear_cross_traffic_alert": {
      "value": "1",
      "label": "Tak"
    },
    "alloy_wheels_type": {
      "value": "light-alloy-wheels-19",
      "label": "Felgi aluminiowe 19"
    },
    "automatic_heating_control": {
      "value": "1",
      "label": "Tak"
    },
    "lumbar_adjust_passenger_electric": {
      "value": "1",
      "label": "Tak"
    },
    "pre_crash_sound_system": {
      "value": "1",
      "label": "Tak"
    },
    "pre_crash_system": {
      "value": "1",
      "label": "Tak"
    },
    "sunblind_type": {
      "value": "sunblind-rear-side-door-electric",
      "label": "Rolety na bocznych szybach opuszczane elektrycznie"
    },
    "acoustic_vehicle_alerting_system": {
      "value": "1",
      "label": "Tak"
    },
    "adjustable_suspension": {
      "value": "1",
      "label": "Tak"
    },
    "extra_urban_consumption": {
      "value": "5.3",
      "label": "5.3 l/100km"
    },
    "fuel_type": {
      "value": "petrol",
      "label": "Benzyna"
    },
    "led_rear_lights": {
      "value": "1",
      "label": "Tak"
    },
    "registered": {
      "value": "1",
      "label": "Tak"
    },
    "air_suspension": {
      "value": "1",
      "label": "Tak"
    },
    "armrest_front": {
      "value": "1",
      "label": "Tak"
    },
    "body_type": {
      "value": "sedan",
      "label": "Sedan"
    },
    "comfort_suspension": {
      "value": "1",
      "label": "Tak"
    },
    "head_up_display": {
      "value": "1",
      "label": "Tak"
    },
    "historical_vehicle": {
      "value": "0",
      "label": "Nie"
    },
    "keyless_go": {
      "value": "1",
      "label": "Tak"
    },
    "pedestrian_emergency_brake_assist": {
      "value": "1",
      "label": "Tak"
    },
    "active_emergency_brake_assist": {
      "value": "1",
      "label": "Tak"
    },
    "basic_autonomous_driving": {
      "value": "1",
      "label": "Tak"
    },
    "child_seat_fixation": {
      "value": "1",
      "label": "Tak"
    },
    "curve_trace_assistant": {
      "value": "1",
      "label": "Tak"
    },
    "driver_conditioning_monitoring": {
      "value": "1",
      "label": "Tak"
    },
    "keyless_entry": {
      "value": "1",
      "label": "Tak"
    },
    "led_interior_lighting": {
      "value": "1",
      "label": "Tak"
    },
    "distribution_of_braking_force_electronically": {
      "value": "1",
      "label": "Tak"
    },
    "memory_seat": {
      "value": "1",
      "label": "Tak"
    },
    "side_pre_crash_system": {
      "value": "1",
      "label": "Tak"
    },
    "touchscreen_monitor": {
      "value": "1",
      "label": "Tak"
    },
    "new_used": {
      "value": "used",
      "label": "Używany"
    },
    "armrest_rear": {
      "value": "1",
      "label": "Tak"
    },
    "automated_high_beam_assist": {
      "value": "1",
      "label": "Tak"
    },
    "blind_spot_warning": {
      "value": "1",
      "label": "Tak"
    },
    "bluetooth_interface": {
      "value": "1",
      "label": "Tak"
    },
    "led_daytime_running_lights": {
      "value": "1",
      "label": "Tak"
    },
    "roll_over_protection_system": {
      "value": "1",
      "label": "Tak"
    },
    "steering_wheel_heated": {
      "value": "1",
      "label": "Tak"
    },
    "android_auto": {
      "value": "1",
      "label": "Tak"
    },
    "digital_key": {
      "value": "1",
      "label": "Tak"
    },
    "pause_recommendation_warning": {
      "value": "1",
      "label": "Tak"
    },
    "rear_seat_with_massage": {
      "value": "1",
      "label": "Tak"
    },
    "steering_wheel_with_radio_operation": {
      "value": "1",
      "label": "Tak"
    },
    "traffic_jam_assist": {
      "value": "1",
      "label": "Tak"
    },
    "upholstery_type": {
      "value": "leather-upholstery",
      "label": "Tapicerka skórzana"
    },
    "electronic_controlled_suspension": {
      "value": "1",
      "label": "Tak"
    },
    "headlight_lamp_type": {
      "value": "led-front-dim-light",
      "label": "Lampy przednie w technologii LED"
    },
    "hydro_pneumatic_suspension": {
      "value": "1",
      "label": "Tak"
    },
    "rear_seat_heating": {
      "value": "1",
      "label": "Tak"
    },
    "sport_seats_front": {
      "value": "1",
      "label": "Tak"
    },
    "startstop_system": {
      "value": "1",
      "label": "Tak"
    },
    "windscreenwiper_other": {
      "value": "1",
      "label": "Tak"
    },
    "approval_for_goods": {
      "value": "0",
      "label": "Nie"
    },
    "headlamp_wash": {
      "value": "1",
      "label": "Tak"
    },
    "make": {
      "value": "bmw",
      "label": "BMW"
    },
    "model": {
      "value": "seria-3",
      "label": "Seria 3"
    },
    "power_windows_rear": {
      "value": "1",
      "label": "Tak"
    },
    "tyre_pressure_control": {
      "value": "1",
      "label": "Tak"
    },
    "color": {
      "value": "white",
      "label": "Biały"
    },
    "generation": {
      "value": "gen-f30-2012",
      "label": "F30/F31 (2012-2020)"
    },
    "head_airbags_front": {
      "value": "1",
      "label": "Tak"
    },
    "park_assistant": {
      "value": "1",
      "label": "Tak"
    },
    "runflat_tyres": {
      "value": "1",
      "label": "Tak"
    },
    "soundsystem": {
      "value": "1",
      "label": "Tak"
    },
    "version": {
      "value": "ver-320i-xdrive-m-sport",
      "label": "320i xDrive M Sport"
    },
    "door_mirror_camera": {
      "value": "1",
      "label": "Tak"
    },
    "cornering_headlights": {
      "value": "1",
      "label": "Tak"
    },
    "keyless_engine_start": {
      "value": "1",
      "label": "Tak"
    },
    "top_electrically_operated": {
      "value": "1",
      "label": "Tak"
    },
    "version_label": {
      "value": "320i xDrive M Sport",
      "label": "320i xDrive M Sport"
    },
    "active_lane_change_assistant": {
      "value": "1",
      "label": "Tak"
    },
    "door_mirrors_electrically_foldable": {
      "value": "1",
      "label": "Tak"
    },
    "driver_airbag": {
      "value": "1",
      "label": "Tak"
    },
    "engine_power": {
      "value": "184",
      "label": "184 KM"
    },
    "head_airbags_rear": {
      "value": "1",
      "label": "Tak"
    },
    "power_steering": {
      "value": "1",
      "label": "Tak"
    },
    "selectable_central_differential_characteristics": {
      "value": "1",
      "label": "Tak"
    },
    "speed_limiter": {
      "value": "1",
      "label": "Tak"
    },
    "convertible_top_type": {
      "value": "textile-top",
      "label": "Dach materiałowy"
    },
    "sport_suspension": {
      "value": "1",
      "label": "Tak"
    },
    "sunroof": {
      "value": "glass-sunroof-fixed",
      "label": "Dach panoramiczny"
    },
    "antilock_brake_system": {
      "value": "1",
      "label": "Tak"
    },
    "heated_seat_driver": {
      "value": "1",
      "label": "Tak"
    },
    "knee_airbag_driver": {
      "value": "1",
      "label": "Tak"
    },
    "rear_transversal_curtain_airbag": {
      "value": "1",
      "label": "Tak"
    },
    "tyre_type": {
      "value": "all-season-tyres",
      "label": "Opony wielosezonowe"
    },
    "lumbar_adjust_driver_electric": {
      "value": "1",
      "label": "Tak"
    },
    "nr_seats": {
      "value": "5",
      "label": "5"
    },
    "power_windows_front": {
      "value": "1",
      "label": "Tak"
    },
    "tuning": {
      "value": "0",
      "label": "Nie"
    },
    "with_vin": {
      "value": "1"
    }
  },
  "bodyType": "sedan",
  "damaged": false,
  "leaseTransfer": false,
  "noAccident": true,
  "serviceRecord": true,
  "originalOwner": true,
  "registered": true,
  "newUsed": "used",
  "condition": "used",
  "conditionLabel": "Używany",
  "generation": "BMW Seria 3 Benzyna 2015-2019",
  "driveType": "4x4-permanent",
  "driveTypeLabel": "4x4 (stały)",
  "color": "white",
  "colorLabel": "Biały",
  "doorCount": 5,
  "nrSeats": 5,
  "colourType": "Metalik",
  "lat": 52.23614,
  "lng": 21.00817,
  "sellerInventoryUrl": "https://WINCARS.otomoto.pl/inventory",
  "sellerAddress": "UL. ALEJA SOLIDARNOŚCI 163 , U010",
  "sellerBadges": [
    "Firma",
    "Bardzo sprawnie odpowiada",
    "Sprzedający na OTOMOTO od 2024"
  ],
  "sellerSinceYear": 2024,
  "sellerActiveAdCount": 212,
  "description": "BMW 320i Edition M Sport Shadow\nZAPRASZAM DO KONTAKTU -Ewa\nSpecyfikacja techniczna:\nRok produkcji: 2018\nPrzebieg: 64 846 km\nSilnik: 2.0 benzyna\nSkrzynia biegów: automatyczna\nKolor: biały\nLiczba miejsc: 5\nNadwozie: sedan\nGeneracja: F30\nWyposażenie zewnętrzne:\n•⁠ ⁠Pakiet M Sport Shadow Line\n•⁠ ⁠Czarne felgi aluminiowe M\n•⁠ ⁠Reflektory LED\n•⁠ ⁠Światła do jazdy dziennej LED\n•⁠ ⁠Szyberdach\n•⁠ ⁠Czujniki parkowania\n•⁠ ⁠Elektrycznie regulowane i podgrzewane lusterka\n•⁠ ⁠Przyciemniane szyby tylne\nKomfort i wnętrze:\n•⁠ ⁠Tapicerka skórzana\n•⁠ ⁠Sportowe fotele [z pamięcią ustawień]\n•⁠ ⁠Automatyczna klimatyzacja\n•⁠ ⁠Tempomat\n•⁠ ⁠Niebieskie listwy ozdobne we wnętrzu\n•⁠ ⁠Wielofunkcyjna kierownica M ze skórzanym wieńcem\n•⁠ ⁠Elektryczne szyby przód i tył\n•⁠ ⁠Podłokietnik przedni\n•⁠ ⁠System ISOFIX\nMultimedia i bezpieczeństwo:\n•⁠ ⁠System multimedialny iDrive z nawigacją\n•⁠ ⁠Bluetooth\n•⁠ ⁠Komputer pokładowy\n•⁠ ⁠ABS, ESP, kontrola trakcji\n•⁠ ⁠Poduszki powietrzne przednie, boczne i kurtynowe\n•⁠ ⁠Dwa komplety kluczyków\nZAPRASZAM DO KONTAKTU -Ewa",
  "photos": [
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6ImI1Z2RtM3p6b3drcTEtT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.sbNdI0QEwDyjwsiIfS-F-kwxu9yYNtvLLZGjJ7qqyD4/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6Ijc2c3p3MGFpMnhrdDEtT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.OV8OM7e37eGWrIR4T362pZUtFrxUx7ApASZ3FJmpYLA/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6Imt6cGF4NmkxcW11NzItT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.gP7BiBQTRTZeN4MbXbhH4W2sksgN9fY8SoPUcULE_hw/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6IjRwZXBjc3V1bWRyMy1PVE9NT1RPUEwiLCJ3IjpbeyJmbiI6IndnNGducXA2eTFmLU9UT01PVE9QTCIsInMiOiIxNiIsImEiOiIwIiwicCI6IjEwLC0xMCJ9XX0.HuRNJ-WRlwV7-vHbabktBdbXL1_XG01x425oibF4VbU/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6ImR4Mzd1OTNrNnRzcDEtT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.7JmalSUedweLU3Rzxaw9b8-K8Iwot8hPyNDwuWmYIh0/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6IjEwbWE3Z2FsbGtrcy1PVE9NT1RPUEwiLCJ3IjpbeyJmbiI6IndnNGducXA2eTFmLU9UT01PVE9QTCIsInMiOiIxNiIsImEiOiIwIiwicCI6IjEwLC0xMCJ9XX0.VWjhPcvChu4cNmQxI3iwxQO1TutSDlrQ7YEOZPfvi2Y/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6ImxreTRuMG80OXRxcS1PVE9NT1RPUEwiLCJ3IjpbeyJmbiI6IndnNGducXA2eTFmLU9UT01PVE9QTCIsInMiOiIxNiIsImEiOiIwIiwicCI6IjEwLC0xMCJ9XX0.WaPcp0RRgLCHblg0TrEVBJpfJCIMynhuT54PxMpAMsU/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6ImIweHlxdTViN2loNS1PVE9NT1RPUEwiLCJ3IjpbeyJmbiI6IndnNGducXA2eTFmLU9UT01PVE9QTCIsInMiOiIxNiIsImEiOiIwIiwicCI6IjEwLC0xMCJ9XX0.KsvnmLHb6uxjJpupPRkLbh0QZM67BEZxA02f29qAnv8/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6ImNycGh4aG84ZjkxcjItT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.Oq8bjYptsPfgsQgGQ7fsJAGnqnVHaEk1FENsd6ENDGI/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6Ino2cnZzb2Yyc2R4bjItT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.72LLOaRYERLgjH01WqGGVmOVDrTSUIdkxTo1zQbZpZM/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6Ijc4a2NhdzJkaDY4MTItT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.xHr1aQRGRxoDKvOHKH-ymJayP_JcIir8Ub6FgpmSISE/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6IjM3Zjk3bW02c3Z3ZjItT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19.ZHFDnjc_koOhXcJXic7JdpsXX-BM3-e1RPMkB6gL1gI/image",
    "https://ireland.apollo.olxcdn.com/v1/files/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmbiI6InFrdGRnNjRyZzRyZzItT1RPTU9UT1BMIiwidyI6W3siZm4iOiJ3ZzRnbnFwNnkxZi1PVE9NT1RPUEwiLCJzIjoiMTYiLCJhIjoiMCIsInAiOiIxMCwtMTAifV19._AzL9kFr-zbi3ro59t_xY-VBTUoBaRQWtaAyamdrdBE/image"
  ],
  "imageCount": 13,
  "equipment": [
    "Apple CarPlay",
    "Android Auto",
    "Interfejs Bluetooth",
    "Radio",
    "Zestaw głośnomówiący",
    "Gniazdo USB",
    "Ładowanie bezprzewodowe urządzeń",
    "System nawigacji satelitarnej",
    "System nagłośnienia",
    "Wyświetlacz typu Head-Up",
    "Ekran dotykowy",
    "Sterowanie funkcjami pojazdu za pomocą głosu",
    "Dostęp do internetu",
    "Hak",
    "Klimatyzacja automatyczna: co najmniej 4 strefy",
    "Dach materiałowy",
    "Rolety na bocznych szybach opuszczane elektrycznie",
    "Dach panoramiczny",
    "Tapicerka skórzana",
    "Klimatyzacja dla pasażerów z tyłu",
    "Elektrycznie ustawiany fotel kierowcy",
    "Elektrycznie ustawiany fotel pasażera",
    "Podgrzewany fotel kierowcy",
    "Podgrzewany fotel pasażera",
    "Regul. elektr. podparcia lędźwiowego - kierowca",
    "Regul. elektr. podparcia lędźwiowego - pasażer",
    "Fotele przednie wentylowane",
    "Fotele przednie z funkcje masażu",
    "Siedzenie z pamięcią ustawienia",
    "Sportowe fotele - przód",
    "Ogrzewane siedzenia tylne",
    "Fotele tylne wentylowane",
    "Fotele tylne z funkcje masażu",
    "Podłokietniki - przód",
    "Podłokietniki - tył",
    "Kierownica skórzana",
    "Kierownica sportowa",
    "Kierownica ze sterowaniem radia",
    "Kolumna kierownicy regulowana elektrycznie",
    "Kierownica wielofunkcyjna",
    "Kierownica ogrzewana",
    "Zmiana biegów w kierownicy",
    "Dźwignia zmiany biegów wykończona skórą",
    "Cyfrowy kluczyk",
    "Keyless entry",
    "Keyless Go",
    "Uruchamianie silnika bez użycia kluczyków",
    "Ogrzewanie postojowe",
    "Automatyczna kontrola ogrzewania",
    "Czujnik deszczu",
    "Podgrzewana przednia szyba",
    "Elektryczne szyby przednie",
    "Elektryczne szyby tylne",
    "Przyciemniane tylne szyby",
    "Wycieraczki",
    "Dach otwierany elektrycznie",
    "Dach otwierany elektrycznie pilotem",
    "Tempomat adaptacyjny ACC",
    "Lampy przednie w technologii LED",
    "Kontrola odległości z przodu (przy parkowaniu)",
    "Kontrola odległości z tyłu (przy parkowaniu)",
    "Asystent parkowania (Park Assistant)",
    "Niezależny system parkowania",
    "Kamera panoramiczna 360",
    "Kamera parkowania tył",
    "Lusterka boczne ustawiane elektrycznie",
    "Podgrzewane lusterka boczne",
    "Lusterka boczne składane elektrycznie",
    "Kamera w lusterku bocznym",
    "Asystent (czujnik) martwego pola",
    "Aktywny asystent zmiany pasa ruchu",
    "Kontrola zmiany pasa ruchu (Lane assist)",
    "Kontrola odległości od poprzedzającego pojazdu",
    "Ogranicznik prędkości",
    "Asystent hamowania (Brake Assist)",
    "Asystent pokonywania zakrętów",
    "Kontrola trakcji",
    "Automatyczna kontrola zjazdu ze stoku",
    "Wspomaganie ruszania pod górę (Hill Holder)",
    "Aktywne rozpoznawanie znaków ograniczenia prędkości",
    "System rozpoznawania znaków drogowych",
    "Asystent zapobiegania kolizjom na skrzyzowaniu",
    "Autonomiczny system kierowania",
    "Asystent świateł drogowych",
    "Oświetlenie adaptacyjne",
    "Dynamiczne światła doświetlające zakręty",
    "Czujnik zmierzchu",
    "Spryskiwacze reflektorów",
    "Lampy doświetlające zakręt",
    "Światła do jazdy dziennej",
    "Światła do jazdy dziennej diodowe LED",
    "Lampy przeciwmgielne",
    "Lampy przeciwmgielne w technologii LED",
    "Lampy tylne w technologii LED",
    "Oświetlenie drogi do domu",
    "Oświetlenie wnętrza LED",
    "System Start/Stop",
    "Elektroniczna kontrola ciśnienia w oponach",
    "Elektryczny hamulec postojowy",
    "Wspomaganie kierownicy",
    "Blokada mechanizmu różnicowego",
    "Regulowany dyferencjał centralny",
    "Asystent jazdy w korku",
    "Filtr cząstek stałych",
    "Felgi aluminiowe 19",
    "Opony wielosezonowe",
    "Opony runflat",
    "Zawieszenie komfortowe",
    "Elektroniczna regul. charakterystyki zawieszenia",
    "Zawieszenie sportowe",
    "Zawieszenie regulowane",
    "Zawieszenie pneumatyczne",
    "Zawieszenie hydropneumatyczne",
    "Hamulce z kompozytów ceramicznych",
    "ABS",
    "ESP",
    "Elektroniczny system rozdziału siły hamowania",
    "System wspomagania hamowania",
    "Asystent hamowania awaryjnego w mieście",
    "System hamowania awaryjnego dla ochrony pieszych",
    "Aktywny asystent hamowania awaryjnego",
    "System ostrzegający o możliwej kolizji",
    "System ochrony słuchu w przypadku kolizji",
    "System chroniący przed kolizja, tylny",
    "System chroniący przed kolizja, boczny",
    "Alarm ruchu poprzecznego z tyłu pojazdu",
    "System emitujący dźwięk silnika (AVAS)",
    "System wykrywania zmęczenie kierowcy",
    "System rekomendacji przerw podczas trasy",
    "Aktywny system monitorowania kondycji kierowcy",
    "System minimalizujący skutki kolizji",
    "System powiadamiania o wypadku",
    "Poduszka powietrzna kierowcy",
    "Poduszka powietrzna pasażera",
    "Poduszka kolan kierowcy",
    "Poduszka kolan pasażera",
    "Kurtyny powietrzne - przód",
    "Poduszka powietrzna centralna",
    "Poduszki powietrzne czołowe - tył",
    "Boczna poduszka powietrzna kierowcy",
    "Boczne poduszki powietrzne - przód",
    "Kurtyny powietrzne - tył",
    "Boczne poduszki powietrzne - tył",
    "Poduszka powietrzna pasów bezpieczeństwa",
    "Kurtyna powietrzna z tyłu, poprzeczna",
    "Isofix (punkty mocowania fotelika dziecięcego)",
    "System zabezpieczający podczas dachowania",
    "Asystent utrzymania pasa ruchu"
  ],
  "equipmentGroups": [
    {
      "label": "Audio i multimedia",
      "values": [
        {
          "label": "Apple CarPlay",
          "key": "apple_carplay"
        },
        {
          "label": "Android Auto",
          "key": "android_auto"
        },
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
          "label": "Ładowanie bezprzewodowe urządzeń",
          "key": "wireless_device_charging"
        },
        {
          "label": "System nawigacji satelitarnej",
          "key": "navigation_system"
        },
        {
          "label": "System nagłośnienia",
          "key": "soundsystem"
        },
        {
          "label": "Wyświetlacz typu Head-Up",
          "key": "head_up_display"
        },
        {
          "label": "Ekran dotykowy",
          "key": "touchscreen_monitor"
        },
        {
          "label": "Sterowanie funkcjami pojazdu za pomocą głosu",
          "key": "voice_control_for_vehicle_functions"
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
          "label": "Hak",
          "key": "towbar"
        },
        {
          "label": "Klimatyzacja automatyczna: co najmniej 4 strefy",
          "key": "air_conditioning_type"
        },
        {
          "label": "Dach materiałowy",
          "key": "convertible_top_type"
        },
        {
          "label": "Rolety na bocznych szybach opuszczane elektrycznie",
          "key": "sunblind_type"
        },
        {
          "label": "Dach panoramiczny",
          "key": "sunroof"
        },
        {
          "label": "Tapicerka skórzana",
          "key": "upholstery_type"
        },
        {
          "label": "Klimatyzacja dla pasażerów z tyłu",
          "key": "air_condition_rear"
        },
        {
          "label": "Elektrycznie ustawiany fotel kierowcy",
          "key": "driver_seat_electrically_adjustable"
        },
        {
          "label": "Elektrycznie ustawiany fotel pasażera",
          "key": "passenger_seat_electrically_adjustable"
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
          "label": "Regul. elektr. podparcia lędźwiowego - kierowca",
          "key": "lumbar_adjust_driver_electric"
        },
        {
          "label": "Regul. elektr. podparcia lędźwiowego - pasażer",
          "key": "lumbar_adjust_passenger_electric"
        },
        {
          "label": "Fotele przednie wentylowane",
          "key": "ventilated_front_seat"
        },
        {
          "label": "Fotele przednie z funkcje masażu",
          "key": "front_seat_with_massage"
        },
        {
          "label": "Siedzenie z pamięcią ustawienia",
          "key": "memory_seat"
        },
        {
          "label": "Sportowe fotele - przód",
          "key": "sport_seats_front"
        },
        {
          "label": "Ogrzewane siedzenia tylne",
          "key": "rear_seat_heating"
        },
        {
          "label": "Fotele tylne wentylowane",
          "key": "ventilated_rear_seat"
        },
        {
          "label": "Fotele tylne z funkcje masażu",
          "key": "rear_seat_with_massage"
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
          "label": "Kolumna kierownicy regulowana elektrycznie",
          "key": "steering_wheel_electrically_adjustable"
        },
        {
          "label": "Kierownica wielofunkcyjna",
          "key": "multi_functional_steering_wheel"
        },
        {
          "label": "Kierownica ogrzewana",
          "key": "steering_wheel_heated"
        },
        {
          "label": "Zmiana biegów w kierownicy",
          "key": "steering_wheel_with_gear_switches"
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
          "label": "Keyless entry",
          "key": "keyless_entry"
        },
        {
          "label": "Keyless Go",
          "key": "keyless_go"
        },
        {
          "label": "Uruchamianie silnika bez użycia kluczyków",
          "key": "keyless_engine_start"
        },
        {
          "label": "Ogrzewanie postojowe",
          "key": "independent_vehicle_heater"
        },
        {
          "label": "Automatyczna kontrola ogrzewania",
          "key": "automatic_heating_control"
        },
        {
          "label": "Czujnik deszczu",
          "key": "rain_sensor"
        },
        {
          "label": "Podgrzewana przednia szyba",
          "key": "windscreen_heating"
        },
        {
          "label": "Elektryczne szyby przednie",
          "key": "power_windows_front"
        },
        {
          "label": "Elektryczne szyby tylne",
          "key": "power_windows_rear"
        },
        {
          "label": "Przyciemniane tylne szyby",
          "key": "tinted_rear_windows"
        },
        {
          "label": "Wycieraczki",
          "key": "windscreenwiper_other"
        },
        {
          "label": "Dach otwierany elektrycznie",
          "key": "top_electrically_operated"
        },
        {
          "label": "Dach otwierany elektrycznie pilotem",
          "key": "top_remote_controlled"
        }
      ],
      "key": "comfort_and_addons"
    },
    {
      "label": "Systemy wspomagania kierowcy",
      "values": [
        {
          "label": "Tempomat adaptacyjny ACC",
          "key": "cruisecontrol_type"
        },
        {
          "label": "Lampy przednie w technologii LED",
          "key": "headlight_lamp_type"
        },
        {
          "label": "Kontrola odległości z przodu (przy parkowaniu)",
          "key": "park_distance_control_front"
        },
        {
          "label": "Kontrola odległości z tyłu (przy parkowaniu)",
          "key": "park_distance_control_rear"
        },
        {
          "label": "Asystent parkowania (Park Assistant)",
          "key": "park_assistant"
        },
        {
          "label": "Niezależny system parkowania",
          "key": "autonomous_parking_system"
        },
        {
          "label": "Kamera panoramiczna 360",
          "key": "360_view_camera"
        },
        {
          "label": "Kamera parkowania tył",
          "key": "rear_view_camera"
        },
        {
          "label": "Lusterka boczne ustawiane elektrycznie",
          "key": "door_mirror_electrically_adjustable_in_general"
        },
        {
          "label": "Podgrzewane lusterka boczne",
          "key": "door_mirrors_heated"
        },
        {
          "label": "Lusterka boczne składane elektrycznie",
          "key": "door_mirrors_electrically_foldable"
        },
        {
          "label": "Kamera w lusterku bocznym",
          "key": "door_mirror_camera"
        },
        {
          "label": "Asystent (czujnik) martwego pola",
          "key": "blind_spot_warning"
        },
        {
          "label": "Aktywny asystent zmiany pasa ruchu",
          "key": "active_lane_change_assistant"
        },
        {
          "label": "Kontrola zmiany pasa ruchu (Lane assist)",
          "key": "lane_control_assistant"
        },
        {
          "label": "Kontrola odległości od poprzedzającego pojazdu",
          "key": "distance_control"
        },
        {
          "label": "Ogranicznik prędkości",
          "key": "speed_limiter"
        },
        {
          "label": "Asystent hamowania (Brake Assist)",
          "key": "brake_assist"
        },
        {
          "label": "Asystent pokonywania zakrętów",
          "key": "curve_trace_assistant"
        },
        {
          "label": "Kontrola trakcji",
          "key": "traction_control"
        },
        {
          "label": "Automatyczna kontrola zjazdu ze stoku",
          "key": "hill_descent_control"
        },
        {
          "label": "Wspomaganie ruszania pod górę (Hill Holder)",
          "key": "hill_holder"
        },
        {
          "label": "Aktywne rozpoznawanie znaków ograniczenia prędkości",
          "key": "active_speed_sign_recognition"
        },
        {
          "label": "System rozpoznawania znaków drogowych",
          "key": "traffic_sign_recognition"
        },
        {
          "label": "Asystent zapobiegania kolizjom na skrzyzowaniu",
          "key": "intersection_assist"
        },
        {
          "label": "Autonomiczny system kierowania",
          "key": "basic_autonomous_driving"
        },
        {
          "label": "Asystent świateł drogowych",
          "key": "automated_high_beam_assist"
        },
        {
          "label": "Oświetlenie adaptacyjne",
          "key": "adaptive_light"
        },
        {
          "label": "Dynamiczne światła doświetlające zakręty",
          "key": "dynamic_directional_lights"
        },
        {
          "label": "Czujnik zmierzchu",
          "key": "automatic_dimlight_activation"
        },
        {
          "label": "Spryskiwacze reflektorów",
          "key": "headlamp_wash"
        },
        {
          "label": "Lampy doświetlające zakręt",
          "key": "cornering_headlights"
        },
        {
          "label": "Światła do jazdy dziennej",
          "key": "daytime_running_lights"
        },
        {
          "label": "Światła do jazdy dziennej diodowe LED",
          "key": "led_daytime_running_lights"
        },
        {
          "label": "Lampy przeciwmgielne",
          "key": "fog_lamps"
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
          "label": "Oświetlenie drogi do domu",
          "key": "follow_me_home"
        },
        {
          "label": "Oświetlenie wnętrza LED",
          "key": "led_interior_lighting"
        },
        {
          "label": "System Start/Stop",
          "key": "startstop_system"
        },
        {
          "label": "Elektroniczna kontrola ciśnienia w oponach",
          "key": "tyre_pressure_control"
        },
        {
          "label": "Elektryczny hamulec postojowy",
          "key": "electric_parking_brake"
        },
        {
          "label": "Wspomaganie kierownicy",
          "key": "power_steering"
        },
        {
          "label": "Blokada mechanizmu różnicowego",
          "key": "limited_slip_differential_in_general"
        },
        {
          "label": "Regulowany dyferencjał centralny",
          "key": "selectable_central_differential_characteristics"
        },
        {
          "label": "Asystent jazdy w korku",
          "key": "traffic_jam_assist"
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
          "label": "Felgi aluminiowe 19",
          "key": "alloy_wheels_type"
        },
        {
          "label": "Opony wielosezonowe",
          "key": "tyre_type"
        },
        {
          "label": "Opony runflat",
          "key": "runflat_tyres"
        },
        {
          "label": "Zawieszenie komfortowe",
          "key": "comfort_suspension"
        },
        {
          "label": "Elektroniczna regul. charakterystyki zawieszenia",
          "key": "electronic_controlled_suspension"
        },
        {
          "label": "Zawieszenie sportowe",
          "key": "sport_suspension"
        },
        {
          "label": "Zawieszenie regulowane",
          "key": "adjustable_suspension"
        },
        {
          "label": "Zawieszenie pneumatyczne",
          "key": "air_suspension"
        },
        {
          "label": "Zawieszenie hydropneumatyczne",
          "key": "hydro_pneumatic_suspension"
        },
        {
          "label": "Hamulce z kompozytów ceramicznych",
          "key": "ceramic_composite_brakes"
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
          "label": "Elektroniczny system rozdziału siły hamowania",
          "key": "distribution_of_braking_force_electronically"
        },
        {
          "label": "System wspomagania hamowania",
          "key": "power_assisted_brakes"
        },
        {
          "label": "Asystent hamowania awaryjnego w mieście",
          "key": "city_emergency_brake_assist"
        },
        {
          "label": "System hamowania awaryjnego dla ochrony pieszych",
          "key": "pedestrian_emergency_brake_assist"
        },
        {
          "label": "Aktywny asystent hamowania awaryjnego",
          "key": "active_emergency_brake_assist"
        },
        {
          "label": "System ostrzegający o możliwej kolizji",
          "key": "collision_warning_system"
        },
        {
          "label": "System ochrony słuchu w przypadku kolizji",
          "key": "pre_crash_sound_system"
        },
        {
          "label": "System chroniący przed kolizja, tylny",
          "key": "rear_pre_crash_system"
        },
        {
          "label": "System chroniący przed kolizja, boczny",
          "key": "side_pre_crash_system"
        },
        {
          "label": "Alarm ruchu poprzecznego z tyłu pojazdu",
          "key": "rear_cross_traffic_alert"
        },
        {
          "label": "System emitujący dźwięk silnika (AVAS)",
          "key": "acoustic_vehicle_alerting_system"
        },
        {
          "label": "System wykrywania zmęczenie kierowcy",
          "key": "driver_conditioning_monitoring"
        },
        {
          "label": "System rekomendacji przerw podczas trasy",
          "key": "pause_recommendation_warning"
        },
        {
          "label": "Aktywny system monitorowania kondycji kierowcy",
          "key": "active_driver_conditioning_monitoring"
        },
        {
          "label": "System minimalizujący skutki kolizji",
          "key": "pre_crash_system"
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
          "label": "Poduszka kolan kierowcy",
          "key": "knee_airbag_driver"
        },
        {
          "label": "Poduszka kolan pasażera",
          "key": "knee_airbag_passenger"
        },
        {
          "label": "Kurtyny powietrzne - przód",
          "key": "head_airbags_front"
        },
        {
          "label": "Poduszka powietrzna centralna",
          "key": "central_airbag_driver_and_passenger"
        },
        {
          "label": "Poduszki powietrzne czołowe - tył",
          "key": "front_airbags_for_rear_seats"
        },
        {
          "label": "Boczna poduszka powietrzna kierowcy",
          "key": "side_airbag_driver"
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
          "label": "Boczne poduszki powietrzne - tył",
          "key": "side_airbags_rear"
        },
        {
          "label": "Poduszka powietrzna pasów bezpieczeństwa",
          "key": "seat_belt_airbag_rear"
        },
        {
          "label": "Kurtyna powietrzna z tyłu, poprzeczna",
          "key": "rear_transversal_curtain_airbag"
        },
        {
          "label": "Isofix (punkty mocowania fotelika dziecięcego)",
          "key": "child_seat_fixation"
        },
        {
          "label": "System zabezpieczający podczas dachowania",
          "key": "roll_over_protection_system"
        },
        {
          "label": "Asystent utrzymania pasa ruchu",
          "key": "active_lane_control_assistant"
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
      "value": "320i xDrive M Sport",
      "group": "basic_information"
    },
    {
      "key": "color",
      "label": "Kolor",
      "value": "Biały",
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
      "value": "2018",
      "group": "basic_information"
    },
    {
      "key": "generation",
      "label": "Generacja",
      "value": "F30/F31 (2012-2020)",
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
      "value": "184 KM",
      "group": "technical_specs"
    },
    {
      "key": "body_type",
      "label": "Typ nadwozia",
      "value": "Sedan",
      "group": "technical_specs"
    },
    {
      "key": "colour_type",
      "label": "Rodzaj koloru",
      "value": "Metalik",
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
      "value": "4x4 (stały)",
      "group": "technical_specs"
    },
    {
      "key": "urban_consumption",
      "label": "Spalanie W Mieście",
      "value": "7.5 l/100km",
      "group": "technical_specs"
    },
    {
      "key": "extra_urban_consumption",
      "label": "Spalanie Poza Miastem",
      "value": "5.3 l/100km",
      "group": "technical_specs"
    },
    {
      "key": "mileage",
      "label": "Przebieg",
      "value": "64 846 km",
      "group": "condition_history"
    },
    {
      "key": "new_used",
      "label": "Stan",
      "value": "Używany",
      "group": "condition_history"
    },
    {
      "key": "registered",
      "label": "Zarejestrowany w Polsce",
      "value": "Tak",
      "group": "condition_history"
    },
    {
      "key": "original_owner",
      "label": "Pierwszy właściciel (od nowości)",
      "value": "Tak",
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
  "updatedAt": "2026-11-18T16:09:57Z",
  "originalCreatedAt": "2026-11-18T15:40:29Z",
  "scrapedAt": "2026-11-18T21:29:55Z",
  "distanceKm": 0.767,
  "marketMedianPrice": 46999,
  "priceVsMarketPct": 14.04,
  "marketCohortCount": 7,
  "marketOutlier": false,
  "belowMarket": false,
  "daysListed": 0,
  "firstSeen": "2026-11-18T18:40:51Z",
  "daysTracked": 0,
  "previousPrice": 53600,
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
