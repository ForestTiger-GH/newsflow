# Weather intelligence for agricultural commodity monitoring

**Project:** `newsflow`  
**Status:** research note  
**Scope:** public weather and agrometeorological sources for market-oriented monitoring of agricultural commodities  
**Primary objective:** detect meaningful weather changes affecting major agricultural production regions as early as practicable through GitHub Actions and a selective LLM layer.

---

## 1. Executive conclusion

Weather monitoring for agricultural markets fits `newsflow` very well, but the correct object of observation is not simply a daily weather bulletin.

The more useful object is:

> **a versioned stream of changing forecasts, warnings, forecast discussions and agrometeorological assessments.**

For market purposes, the most valuable event is often not that a forecast exists, but that it changed:

- expected rainfall decreased or increased;
- frost risk appeared, disappeared or intensified;
- a heat episode moved into a sensitive crop-development window;
- the affected area expanded or contracted;
- confidence increased or decreased;
- forecast models converged or diverged;
- timing shifted relative to sowing, flowering, pollination, harvesting or drying.

This suggests a four-speed architecture:

```text
MINUTES / HOURS
warnings / forecast discussions / short-range forecasts
        ↓
detect intraday changes in expectations
        ↓
DAY
daily forecast / daily agricultural weather
        ↓
WEEK
agrometeorological + crop/weather assessment
        ↓
10 DAYS / MONTH
soil moisture / drought / crop condition / seasonal context
```

GitHub Actions is a good fit for the first three layers, provided it is treated as a frequent observation system rather than ultra-low-latency trading infrastructure.

---

## 2. Core conceptual shift: forecast revision as a first-class event

A conventional news collector focuses on:

```text
new publication appeared
```

For weather intelligence, this is insufficient.

A more useful event is:

```text
forecast revision
```

Example:

```text
previous forecast:
Mato Grosso
scattered rain
20–40 mm
Friday–Saturday
```

Later:

```text
new forecast:
Mato Grosso
mostly dry
rain limited to southern areas
below 10 mm
```

The market-relevant event is:

```text
RAIN EXPECTATION DOWN

20–40 mm → <10 mm
coverage: regional → southern areas only
timing: Friday–Saturday → mostly absent
```

Other important transitions include:

```text
frost:
none → possible → likely

heat:
35°C → 39–41°C

rain:
isolated → widespread

rain timing:
before pollination → during pollination

soil moisture:
adequate → short / deficient

confidence:
low → moderate → high
```

Therefore, `diff` between forecast issues should be treated as a first-class object.

---

## 3. Why daily bulletins alone are too slow

Daily and weekly agricultural weather reports remain valuable, but they are better used for confirmation, interpretation and medium-term context.

The fastest useful layer is composed of:

- warnings;
- short-range forecasts;
- forecast discussions;
- structured forecast APIs;
- continuously updated forecast pages;
- regional meteorological discussions.

The slower layers provide:

- crop-weather interpretation;
- soil moisture;
- drought development;
- crop-condition confirmation;
- regional agricultural context;
- international comparison.

The strongest system combines both.

---

## 4. Strong source classes

### 4.1 United States

The United States has one of the best infrastructures for this use case.

#### NWS Area Forecast Discussion

The National Weather Service publishes Area Forecast Discussions (AFD), which explain forecast reasoning, uncertainty, timing and model disagreement.

Official description:

https://www.weather.gov/btv/product_descriptions

AFDs are especially valuable because they expose the human reasoning behind a forecast rather than merely presenting temperatures and precipitation totals.

They can reveal:

- increasing confidence in a dry or wet scenario;
- model disagreement;
- changes in timing;
- expanding or shrinking hazard areas;
- synoptic explanations;
- forecast uncertainty.

NWS also provides an official API with forecasts, hourly forecasts, raw forecast grids, observations and alerts:

https://www.weather.gov/documentation/services-web-api

This allows a fast U.S. layer focused on agricultural regions such as:

- Corn Belt;
- Great Plains;
- Delta;
- Texas;
- Pacific Northwest;
- California;
- Southeast.

#### NOAA CPC

NOAA Climate Prediction Center publishes daily 6–10 day and 8–14 day forecast discussions.

Example:

https://www.cpc.ncep.noaa.gov/products/predictions/6-10_day/fxus06.html

These discussions are valuable because they reference ensemble guidance, model agreement/disagreement and forecast confidence.

#### USDA

USDA publishes:

- Daily U.S. Agricultural Weather Highlights;
- Weekly Weather and Crop Bulletin.

USDA describes these products as agricultural-weather information relevant to production and market volatility.

Weather resources:

https://www.rma.usda.gov/tools-reports/weather-resources

Weekly Weather and Crop Bulletin catalogue:

https://esmis.nal.usda.gov/publication/weekly-weather-and-crop-bulletin

The main limitation is format: some of the most valuable USDA products are PDF-heavy.

Recommended role:

```text
FAST:
NWS AFD + NWS alerts/API

DAILY:
NOAA CPC 6–10 / 8–14 discussion

AG-MARKET CONFIRMATION:
USDA Daily Agricultural Weather Highlights

WEEKLY GLOBAL CONFIRMATION:
USDA Weekly Weather & Crop Bulletin
```

---

### 4.2 Brazil

Brazil is highly suitable for HTML-based collection.

INMET publishes HTML weather articles containing:

- regional forecasts;
- rainfall amounts;
- frost risk;
- heat;
- cold fronts;
- cyclones;
- short-range regional outlooks;
- weekly forecasts;
- occasional agricultural interpretation.

Example weekly forecast:

https://portal.inmet.gov.br/noticias/inmet-divulga-previs%C3%A3o-para-semana-de-14-a-21-de-setembro-de-2026

News catalogue:

https://portal.inmet.gov.br/noticias/noticias

This makes INMET an excellent source for:

- soybeans;
- corn;
- coffee;
- sugar;
- cotton.

Suggested collector pattern:

```text
INMET news index
        ↓
poll every 15–30 minutes
        ↓
new URL?
        ↓
title classifier
        ↓
weather / forecast / frost / heat / rain / agroclimate
        ↓
fetch HTML
```

---

### 4.3 Argentina

Bolsa de Comercio de Rosario / GEA is particularly valuable because it already combines weather and agricultural interpretation.

GEA climate indicators:

https://www.bcr.com.ar/es/mercados/gea/seguimiento-de-cultivos/indicadores-climaticos

Typical material combines:

- next-week forecast;
- soil moisture;
- crop condition;
- agricultural region;
- campaign phase;
- corn / soybean / wheat implications.

This is valuable because the source itself often performs the weather-to-agriculture bridge.

A strong two-layer design is possible:

```text
daily:
observed precipitation

weekly:
forecast + soil moisture + crop interpretation
```

For grain markets, BCR/GEA should be considered a high-priority source.

---

### 4.4 Russia

Hydrometeorological Centre of Russia regularly publishes:

> Обзор атмосферных процессов и наиболее важных явлений погоды

Main news route:

https://meteoinfo.ru/novosti

These materials are suitable for HTML collection and contain:

- regions;
- precipitation;
- temperature;
- wind;
- frost;
- atmospheric processes.

They are relevant for:

- wheat;
- sunflower;
- barley;
- corn.

Priority areas include southern European Russia, Volga region and Central Russia.

---

### 4.5 Ghana — cocoa

Ghana Meteorological Agency provides a particularly useful fast layer.

24 Hour Impact-Based Forecast:

https://www.meteo.gov.gh/product/24-hour-impact-based-forecast/

Forecasts are issued in intraday segments such as:

- Morning;
- Afternoon;
- Evening;
- Night.

They contain text for zones such as:

- forested areas;
- transition sector;
- coast;
- northern sector.

For cocoa, this creates a useful fast-weather layer.

GMet also publishes seasonal and sub-seasonal forecasts:

https://www.meteo.gov.gh/product/seasonal-and-sub-seasonal-forecast/

A possible architecture:

```text
GMet intraday
        ↓
forest-zone rain / thunderstorm changes
        ↓
sub-seasonal weekly confirmation
        ↓
dekadal agrometeorological confirmation
```

Important semantic boundary:

```text
source says:
"Forest Zone"

our bridge says:
"relevant to cocoa-producing areas"
```

That market bridge must remain distinct from the source observation.

---

## 5. Broader source matrix

| Market / region | Source | Approximate speed | Machine readability | Main use |
|---|---|---:|---|---|
| United States | NWS AFD / alerts | several times daily / event-driven | very high | fast weather intelligence |
| United States | NOAA CPC 6–10 / 8–14 | daily | HTML | forecast revision and confidence |
| U.S. / global | USDA Daily / Weekly | daily / weekly | PDF + HTML index | agricultural interpretation |
| Brazil | INMET | daily + weekly | HTML | soy, corn, coffee, sugar, cotton |
| Argentina | BCR GEA | daily + weekly | HTML | soy, corn, wheat |
| Russia | Hydrometcenter | every few days | HTML | grain, sunflower |
| Ghana | GMet | several times daily | HTML | cocoa |
| Ukraine | Ukrhydrometcenter | dekadal + other | HTML | precipitation / soil moisture |
| Kazakhstan | Kazhydromet | dekadal | HTML | wheat, barley, sunflower |
| India | IMD | fast warnings + slower agromet | HTML index + PDF | cotton, sugar, rice, wheat |
| Thailand | TMD | multi-day / weekly | HTML + PDF | sugar, rubber, rice |
| Indonesia | BMKG | dekadal | HTML | palm oil, rubber, coffee |
| Australia | Bureau of Meteorology | weekly / medium-range | HTML | wheat, canola |
| Canada | ECCC | hourly / daily | HTML + structured data | Prairie weather |
| Canada | AAFC | daily to monthly | HTML / maps | agroclimate and drought |
| EU | JRC MARS | monthly | HTML / PDF / data | crop-impact confirmation |
| Vietnam | NCHMF | daily → dekadal / monthly | HTML | coffee potential |

---

## 6. Important gaps

Several globally important agricultural regions are less straightforward.

### China

China is essential for:

- cotton;
- corn;
- soybeans;
- wheat.

CMA/NMC provides many products, but a stable public textual agrometeorological series suitable for easy automation still requires dedicated discovery.

### Côte d'Ivoire

Critical for cocoa, but public SODEXAM routes appear less straightforward than Ghana GMet.

### Colombia

Important for coffee. IDEAM publishes climate and agroclimatic material, but much of it is slower or PDF-heavy.

### Malaysia

Relevant for palm oil and rubber. Public agrometeorological products exist, but stable routing requires further work.

### Vietnam

NCHMF appears promising for coffee, especially the Central Highlands, but source routing should be tested separately.

These areas should form a second research wave rather than delaying an initial system.

---

## 7. Correct logical abstraction: commodity × production region

The monitoring system should not be conceptually organized only by country.

A better routing layer is:

```text
corn
  US Corn Belt
  Brazil
  Argentina

soy
  United States
  Brazil
  Argentina

wheat
  US Plains
  Canada Prairies
  Russia
  Ukraine
  Kazakhstan
  European Union
  Australia

coffee
  Brazil
  Vietnam
  Colombia

cocoa
  Côte d'Ivoire
  Ghana

sugar
  Brazil
  India
  Thailand

cotton
  United States
  Brazil
  India
  China

palm_oil
  Indonesia
  Malaysia

canola
  Canada
  Australia
  European Union
```

This is primarily a logical routing structure.

A source such as one INMET article may be relevant simultaneously to:

- coffee;
- sugar;
- soybeans;
- corn.

Therefore, the same source material should not be physically duplicated by commodity.

---

## 8. Recommended pipeline

```text
                    PUBLIC WEB
                        │
          ┌─────────────┼─────────────┐
          │             │             │
        HTML           API        RSS / index
          │             │             │
          └─────────────┬─────────────┘
                        ▼
                  DISCOVERY POLL
                        │
                  changed / new?
                  │           │
                 no          yes
                  │           │
                 stop         ▼
                           FETCH
                             │
                ┌────────────┴────────────┐
                ▼                         ▼
            raw snapshot             main text
                │                         │
                └────────────┬────────────┘
                             ▼
                     compare previous
                             │
                             ▼
                       semantic diff
                             │
                        LLM ONLY HERE
                             │
                             ▼
                       WEATHER EVENT
                             │
                             ▼
                       market router
                             │
          ┌──────────────────┼──────────────────┐
          ▼                  ▼                  ▼
         soy               wheat              cocoa
          │                  │                  │
          └──────────────────┴──────────────────┘
                             ▼
                        alert / digest
```

The LLM should not be used for polling, HTTP retrieval, hashing, timestamp parsing or deduplication.

Those operations should remain deterministic.

The LLM should be invoked only when:

```text
new meaningful text detected
```

or:

```text
main_content_sha changed
```

---

## 9. Fingerprinting

Weather pages frequently change for technical reasons unrelated to forecast content.

At minimum, store:

```text
raw_sha256
main_content_sha256
```

Optionally:

```text
link_set_sha256
```

Interpretation:

```text
raw HTML changed = yes
main content changed = no

→ ignore
```

versus:

```text
raw HTML changed = yes
main content changed = yes

→ inspect
```

Fixed URLs such as forecast discussions require body-hash comparison.

Index-based sources such as INMET or GMet require discovery of new publication URLs followed by targeted fetch.

---

## 10. LLM extraction contract

The LLM should not merely produce a narrative summary.

A more useful structure is:

```yaml
event_type: forecast_revision

source:
  publisher: INMET
  product: short_range_forecast

issue:
  published_at: ...
  observed_at: ...
  valid_from: ...
  valid_to: ...

geography:
  source_regions:
    - Paraná
    - Mato Grosso do Sul

weather:
  phenomena:
    - heavy_rain
    - frost
  direction:
    - precipitation_increase
    - temperature_decrease

change_vs_previous:
  type: intensification
  timing_shift: ...
  spatial_change: ...
  magnitude_change: ...

source_support:
  - exact source section / paragraph identity

uncertainty:
  source_confidence: UNKNOWN
  extraction_confidence: high
```

The market bridge should remain separate:

```yaml
market_bridge:
  commodities:
    - soybeans
    - corn
  production_regions:
    - ...
  crop_stage:
    - ...
  relevance_reason:
    - ...
```

This preserves the distinction between:

```text
source observation
```

and:

```text
market interpretation
```

---

## 11. Suggested forecast-change taxonomy

A compact taxonomy can include:

```text
NEW_HAZARD
HAZARD_REMOVED

INTENSIFICATION
WEAKENING

DRYER
WETTER
HOTTER
COLDER

TIMING_EARLIER
TIMING_LATER

AREA_EXPANSION
AREA_CONTRACTION

CONFIDENCE_UP
CONFIDENCE_DOWN

MODEL_CONVERGENCE
MODEL_DIVERGENCE
```

The last two categories are particularly useful when forecast discussions explicitly describe model disagreement and convergence.

Example:

```text
yesterday:
ECMWF and GEFS diverged

today:
both converge on a dry scenario
```

This may be more informative than the change in mean forecast alone.

---

## 12. GitHub Actions scheduling

A naïve design would poll the entire world every five minutes.

That is unnecessary.

A better design uses source-specific publication windows.

For example:

```text
CPC expected publication window

14:55
15:07
15:19
15:37
15:52
16:07
16:22
```

Avoid `:00` where possible because scheduled GitHub Actions may be delayed around the start of the hour.

GitHub Actions scheduled workflows documentation:

https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows

Troubleshooting scheduled workflows:

https://docs.github.com/en/actions/how-tos/troubleshoot-workflows

Over time, `newsflow` can learn actual publication timing.

Store:

```text
scheduled_release_time
actual_first_seen_time
```

Then optimize polling windows around observed release distributions.

---

## 13. Measure newsflow latency

For each publication or revision, store:

```text
source_published_at
newsflow_first_seen_at
newsflow_fetched_at
newsflow_event_created_at
```

Derived operational metrics:

```text
detection_latency
processing_latency
total_latency
```

Example:

```text
INMET
median detection latency = 8 min
p95 = 19 min

GMet
median detection latency = 11 min

CPC
median detection latency = 6 min
```

This allows `newsflow` to evaluate its own performance quantitatively.

---

## 14. PDF policy

The system should not be built around PDF.

However, completely excluding PDF would remove highly valuable sources such as:

- USDA Weekly Weather & Crop Bulletin;
- some IMD agrometeorological products;
- some Thai agricultural-weather publications;
- other national agricultural bulletins.

Recommended rule:

```text
HTML / API / RSS
    = preferred fast layer

PDF whose existence is discovered from stable HTML catalogue
    = allowed confirmation layer

PDF-only opaque scraping
    = low priority
```

This avoids making PDF extraction part of the core fast path.

---

## 15. Initial production-oriented source set

For maximum market value per unit of implementation effort:

### 1. U.S. grains

- selected NWS AFD;
- NWS alerts;
- NWS structured API;
- NOAA CPC discussions.

### 2. Brazil

- INMET short-range weather articles;
- INMET weekly forecast;
- relevant frost / rain / heat / agroclimate articles.

### 3. Argentina

- BCR GEA climate indicators;
- daily precipitation;
- crop-weather assessments.

### 4. Russia

- Hydrometcenter atmospheric-process reviews.

### 5. Cocoa

- Ghana GMet intraday forecasts;
- sub-seasonal forecast;
- dekadal agrometeorological confirmation.

### Second wave

- Ukraine;
- Kazakhstan;
- Canada;
- India;
- Indonesia;
- Thailand;
- Australia;
- Vietnam;
- Côte d'Ivoire;
- Colombia;
- Malaysia;
- China.

### Later confirmation layer

- USDA Weekly Weather & Crop Bulletin;
- other high-value PDFs discovered through stable catalogues.

---

## 16. Workflow partitioning

Do not place every source in one giant workflow.

A better logical split is:

```text
weather-discovery-fast
weather-discovery-daily
weather-discovery-slow
```

or source-specific groups.

Advantages:

- a failing Indian site does not break INMET;
- different sources can use different schedules;
- retry policy can be source-specific;
- source adapters remain isolated;
- debugging becomes easier.

A fast intraday source and a monthly JRC bulletin should not share the same cadence.

---

## 17. Persistent state

State should include:

```text
last_release_id
last_published_at
last_url
raw_sha256
main_content_sha256
previous_release_id
```

Empty polls should not generate commits.

Commits should occur only for material state changes such as:

```text
NEW
REVISION
RECOVERY
SOURCE_CHANGE
```

This prevents repository history from being flooded with meaningless polling commits.

---

## 18. Longer-term architecture: machine forecast + human interpretation

Textual weather releases are powerful, but they are not the earliest possible signal.

Meteorologists often write forecast discussions after new numerical model runs already exist.

The longer-term architecture can therefore become:

```text
                 WEATHER SIGNAL

      MACHINE FORECAST          HUMAN FORECAST
             │                       │
     model/grid change             HTML text
             │                       │
             ▼                       ▼
      numerical diff             semantic diff
             │                       │
             └───────────┬───────────┘
                         ▼
                 convergence layer
                         │
                         ▼
                   market event
```

Example:

```text
18z forecast:
Iowa 7-day precipitation = 42 mm

00z:
27 mm

06z:
14 mm
```

This numerical change may be visible before a human forecast discussion states that confidence has increased in a drier pattern.

NWS structured forecast interfaces make a first experiment possible without a commercial data provider:

https://www.weather.gov/documentation/services-web-api

---

## 19. Recommended conceptual chain

The weather branch of `newsflow` should preserve the following distinction:

```text
official forecast
→ captured representation
→ forecast issue
→ revision / semantic delta
→ crop-region bridge
→ commodity relevance
→ market-facing event
```

The raw weather statement remains a source observation.

The mapping from that weather statement to:

- crop;
- production region;
- crop stage;
- expected agricultural significance;
- market relevance;

is a separate bridge or derived layer.

---

## 20. Final recommendation

The project is technically viable and strategically useful.

The first implementation should not attempt to create a universal global meteorological crawler.

It should begin with a compact high-value backbone:

```text
NWS / NOAA CPC
+ INMET
+ BCR GEA
+ Hydrometcenter
+ Ghana GMet
```

This already covers major weather-sensitive agricultural markets across:

- U.S. grains;
- Brazilian soy/corn/coffee/sugar;
- Argentine soy/corn/wheat;
- Russian grain;
- West African cocoa.

The core architectural principle should be:

> `newsflow` observes versioned public weather forecasts, detects new issues and revisions, computes changes relative to previous states, and only then routes those changes toward agricultural commodity markets.

The deterministic layer should perform:

- discovery;
- retrieval;
- hashing;
- timestamps;
- deduplication;
- version control;
- source-state management.

The LLM layer should be reserved for:

- semantic forecast diffs;
- extraction of changes in timing, intensity, area and confidence;
- structured normalization;
- crop-region routing assistance;
- interpretation that cannot be obtained deterministically.

That separation gives a much stronger system than a generic weather-news collector and creates a path toward a genuine market-oriented weather-intelligence layer.
