# Bank Rate-Bearing HTML Research

## Status

Research note for `newsflow`.

This document evaluates a specific architectural hypothesis:

> Can `newsflow` obtain sufficiently broad coverage of bank deposit and loan rates by regularly saving official HTML pages that already contain published rates, without building source-specific parsers for each bank?

The intended downstream chain is:

```text
official bank website
→ rate-bearing HTML page
→ newsflow snapshot/history
→ LLM extraction
→ structured observations
→ Tabularium / documented bridge
→ daily time series
```

The key question is not whether banks publish rates somewhere. The key question is whether there are enough official **rate-bearing web pages** to make HTML-first observation a practical primary strategy.

---

# 1. Executive conclusion

Yes.

Rate-bearing HTML is sufficiently common across the current banking universe already represented in Tabularium to justify an **HTML-first architecture** for bank-rate monitoring.

For deposits and savings accounts, the approach appears especially strong. For consumer lending it is also useful, but published web rates are more often represented as ranges, full-cost-of-credit ranges, conditional rates, or individualized pricing envelopes.

Across the current Tabularium banking universe, useful official web pages containing numerical rate information were identified for the large majority of banks. In many cases the page contains substantially more than a promotional “up to X%” headline: full term structures, amount-by-term matrices, ordinary versus promotional rates, effective dates, or archived tariff versions.

The strongest source classes found include:

- full HTML tariff matrices;
- current-state product pages with explicit numeric rates;
- dated HTML tariff documents;
- HTML archives of prior versions;
- dated HTML announcements of rate changes.

The main unresolved technical distinction is:

```text
rate exists on public web page
≠
rate is present in initial HTTP response HTML
```

For some banks, JavaScript may render the rate only after page load. This does not invalidate the architecture. A generic browser collector such as Playwright can serialize the final rendered DOM with `page.content()` and store an HTML representation that already contains the rates, without any bank-specific parsing logic.

Therefore the central collector design should be:

```text
known rate-bearing URL
→ fetch / render
→ save HTML
→ timestamp + provenance + hash
→ newsflow
```

The collector itself should not know how to interpret the rate.

---

# 2. What counts as a rate-bearing page

A useful source for this project is not merely any page mentioning a bank product.

A **rate surface** is an official public web page whose rendered representation contains bank pricing information relevant to a financial product.

A practical classification is:

| Class | Description | Usefulness |
|---|---|---|
| A — tariff HTML | Full or near-full matrix such as term × amount × rate, sometimes with effective date and version history | Ideal |
| B — state HTML | Current numerical rates or ranges plus core conditions, while some detail may remain in calculator/PDF | Very strong |
| C — event HTML | Dated publication stating that rates changed from a specified date | Useful secondary evidence |
| D — document index | HTML page that only links to PDF/RTF tariff files | Insufficient by itself for HTML-only monitoring |

The target architecture should prioritize A and B pages for continuous observation.

C pages are useful for confirmation and historical reconstruction.

D pages remain useful as fallback discovery surfaces for formal tariff documents.

---

# 3. Why this matters architecturally

The conventional approach would be to build a separate parser for every bank:

```text
bank A:
CSS selector A1
CSS selector A2

bank B:
XPath B1
XPath B2

bank C:
JavaScript API reverse engineering
...
```

This does not scale well.

Frontend structures change, class names change, components move, sites migrate between frameworks, and product layouts are redesigned.

The proposed approach separates **retrieval** from **interpretation**.

The collector knows only:

```yaml
bank: otp-bank
surface: retail-deposit-horizon
url: https://...
representation: rendered_html
```

It does not know:

```text
where the rate is located
which <div> contains the term
which CSS class means “promotional rate”
which column contains the amount bucket
```

The entire rendered HTML is stored as evidence.

The LLM then interprets that HTML later.

This makes the expensive semantic component flexible while keeping collection deterministic and source-faithful.

---

# 4. Current Tabularium banking universe

The current banking contour already represented in Tabularium includes at least the following rate-relevant entities:

1. Sber
2. VTB
3. Gazprombank
4. Alfa-Bank
5. Russian Agricultural Bank (RSHB)
6. PSB
7. Moscow Credit Bank (MKB)
8. Bank DOM.RF
9. T-Bank
10. Sovcombank
11. OTP Bank
12. MTS Bank
13. Ak Bars Bank
14. Bank Saint Petersburg
15. Uralsib
16. VBRR
17. MSP Bank
18. Ozon Bank
19. Yandex Bank

`T-Technologies` should not be treated as a separate retail-rate source. For rate monitoring it should resolve to T-Bank through an explicit entity bridge.

---

# 5. Bank-by-bank preliminary census

## 5.1 Sber

### Preliminary status

```text
Deposits/savings: technically unresolved
Loans: technically unresolved
```

Official product pages and a dedicated archive of interest rates exist and are indexed publicly.

However, automated direct retrieval of the official site is not consistently reliable in the current test environment.

This should not be classified as `NO_DATA`.

Correct status:

```text
DATA_EXISTS
FETCH_MECHANISM_UNRESOLVED
```

Likely next step:

```text
generic Playwright render
```

The key question is whether the final rendered DOM contains the full rate information without requiring authenticated interaction.

---

## 5.2 VTB

### Deposits and savings

Strong current-state HTML.

The official web surface exposes numerical rates for deposits and savings products directly on the page.

### Loans

Strong current-state HTML.

Consumer loan pages expose:

- full-cost-of-credit range;
- interest-rate range;
- differences between pricing with and without optional services.

### Classification

```text
Deposits: B
Loans: B
```

Very suitable for continuous HTML snapshotting.

---

## 5.3 Gazprombank

### Deposits and savings

Official HTML pages expose explicit rates and product conditions.

### Loans

Consumer credit pages contain full-cost-of-credit ranges and conditional rate ranges in web-readable form.

Formal tariff documents also exist separately.

### Classification

```text
Deposits: A/B
Loans: B
```

This is a good example of a bank where HTML can be the daily monitoring layer while formal PDF tariffs remain a verification/backfill layer.

---

## 5.4 Alfa-Bank

### Deposits and savings

Official pages expose numerical values for several products and calculator states.

Alfa-Bank also provides dated HTML announcements of tariff changes.

### Loans

Official pages expose:

- full-cost-of-credit range;
- interest-rate range;
- explicit statement that the final rate is determined individually.

### Classification

```text
Deposits: A/B
Loans: B
```

The observation must preserve the published range rather than manufacture a single “Alfa loan rate”.

---

## 5.5 Russian Agricultural Bank (RSHB)

### Deposits and savings

One of the strongest sources.

Official pages expose current numerical rates, product terms, and explicit effective dates.

The bank also maintains a rich historical archive of dated rate revisions.

### Loans

Rate information exists, but formal detail may sometimes be concentrated in attached documents rather than the web page itself.

### Classification

```text
Deposits: A
Loans: B/D depending on product
```

RSHB is a strong candidate for historical backfill.

---

## 5.6 PSB

### Deposits and savings

Strong HTML source.

Official pages contain actual rate tables rather than only headline maximums.

### Loans

The bank also publishes dated HTML announcements of lending-rate changes.

### Classification

```text
Deposits: A
Loans: B/C
```

Useful both for current-state monitoring and event-based historical reconstruction.

---

## 5.7 Moscow Credit Bank (MKB)

### Deposits and savings

Important finding: some MKB `/doc/...` resources are web-readable documents rather than only binary PDF files.

These pages can contain:

- product name;
- amount ranges;
- terms;
- numerical rate matrices;
- effective dates.

### Classification

```text
Deposits: A
Loans: B/C/D depending on product
```

This makes MKB substantially more attractive for HTML-first monitoring than a simple “PDF archive only” classification would suggest.

---

## 5.8 Bank DOM.RF

### Deposits and savings

Current HTML product pages expose:

- headline rates;
- conditions;
- bonuses/premiums;
- some account pricing.

Detailed matrices may still require calculator/document fallback.

### Loans

The bank publishes dated HTML announcements of rate changes with explicit numeric values.

### Classification

```text
Deposits: B
Loans: B/C
```

Useful web sensor, though not always a complete tariff surface.

---

## 5.9 T-Bank

### Deposits and savings

Strong web-readable product and tariff surfaces.

### Loans

Consumer credit pages expose explicit:

- full-cost-of-credit ranges;
- interest-rate ranges.

### Classification

```text
Deposits: A/B
Loans: B
```

A natural candidate for the first continuous collector set.

---

## 5.10 Sovcombank

One of the strongest HTML sources discovered.

### Deposits

Official pages can expose a matrix such as:

```text
term
×
base rate
×
enhanced rate
```

with explicit validity dates.

### Loans

Official pages may expose product-level tables with:

- product;
- full-cost-of-credit range;
- annual interest rate;
- amount;
- term.

### Classification

```text
Deposits: A
Loans: A
```

Excellent candidate for end-to-end testing.

---

## 5.11 OTP Bank

Another near-ideal HTML source.

### Deposits

Official pages contain genuine matrices such as:

```text
amount bucket
×
term
→
rate
```

This is substantially richer than a headline maximum.

### Loans

Official consumer-loan pages expose numerical rate and full-cost-of-credit ranges.

### Classification

```text
Deposits: A
Loans: A/B
```

Very strong candidate for LLM extraction experiments.

---

## 5.12 MTS Bank

### Deposits

Current HTML product catalogs expose numeric ranges and product-level pricing.

However, the formal tariff archive frequently uses PDF documents.

### Loans

Consumer-credit pages expose numerical rate and full-cost-of-credit ranges in HTML.

### Classification

```text
Deposits: B
Loans: B
```

MTS is especially useful because it also has a rich historical sequence of tariff documents, allowing HTML current-state monitoring plus document-based historical backfill.

---

## 5.13 Ak Bars Bank

### Preliminary status

The public site is more JavaScript-dependent.

Some official surfaces explicitly require JavaScript.

### Classification

```text
Deposits: unresolved raw HTML / likely rendered HTML
Loans: unresolved raw HTML / likely rendered HTML
```

The correct next experiment is not a custom parser.

It is:

```text
Playwright
→ page load
→ page.content()
→ inspect whether rates exist in rendered HTML
```

---

## 5.14 Bank Saint Petersburg

### Deposits

Strong finding: official web pages expose full term/rate matrices.

### Loans

Mortgage and other lending pages also expose numerical rates and full-cost-of-credit information.

### Classification

```text
Deposits: A
Loans: A/B
```

A strong HTML-first source.

---

## 5.15 Uralsib

### Deposits and savings

Official pages directly expose numerical rates for several deposits and savings products.

However, formal detailed tariff surfaces often remain in PDF.

### Loans

Mixed HTML/document model.

### Classification

```text
Deposits: B
Loans: B/D
```

Good daily sensor, with document fallback for full formal coverage.

---

## 5.16 VBRR

### Preliminary status

Official product pages exist and publicly indexed content indicates blocks such as:

```text
Interest rate (% per annum)
```

However, direct automated retrieval is not fully reliable in the current environment.

### Classification

```text
Deposits: likely B
Loans: unresolved
```

Needs a Playwright-level fetch test before final classification.

---

## 5.17 MSP Bank

MSP Bank differs from mass-market retail banks because much of its relevant rate offering is business-oriented.

### Deposits

Official HTML product pages expose:

- headline deposit yield;
- amount range;
- term range.

Some exact pricing may depend on a calculator or formal tariff document.

### Classification

```text
Deposits: B
Loans: product-specific
```

It remains relevant because the project goal is bank pricing behavior, not only mass retail.

---

## 5.18 Ozon Bank

A particularly strong finding.

### Deposits

The official product landing page exposes a detailed term structure directly in HTML across multiple maturities.

This means the HTML itself can function as a high-value daily rate surface.

### Classification

```text
Deposits: A
Loans: B/D depending on product
```

Ozon demonstrates that modern digital-bank landing pages can be more useful for machine observation than classic tariff-document archives.

---

## 5.19 Yandex Bank

Possibly the strongest example discovered.

### Fixed-term savings

The official tariff is a normal HTML page containing:

- effective date;
- full term/rate table;
- linked prior versions.

### Savings account

Similar structure:

- current numerical rates;
- explicit conditions;
- historical HTML versions.

### Classification

```text
Deposits/savings: A+
Loans: B depending on product
```

Yandex effectively provides a versioned historical HTML tariff corpus already suitable for immediate backfill.

This is strong evidence that HTML can itself be the canonical tariff representation rather than merely a marketing shell around PDFs.

---

# 6. Aggregate assessment

The central hypothesis survives the empirical test.

For the current Tabularium banking universe:

```text
rate-bearing HTML is common
```

and for deposits/savings it appears available for the overwhelming majority of relevant institutions.

A preliminary classification suggests:

- a large core of banks with full or near-full HTML tariff surfaces;
- several banks with useful current-state HTML plus document fallback;
- a small unresolved group where JavaScript/browser rendering must be tested.

The unresolved banks should not yet be interpreted as failures of the architecture.

They are primarily:

```text
FETCH_MECHANISM_UNRESOLVED
```

rather than:

```text
NO_RATE_BEARING_PAGE
```

---

# 7. Deposits versus loans

The HTML-first architecture is strongest for:

```text
fixed-term deposits
savings accounts
```

because banks frequently publish explicit product/term/rate structures.

Loan pricing is more semantically difficult.

A bank may publish:

```text
rate_min
rate_max
psk_min
psk_max
rate_with_service
rate_without_service
```

while the actual customer rate remains individualized.

Therefore the correct public observation is not:

```text
bank_loan_rate = 19.9%
```

but a pricing envelope such as:

```text
advertised_rate_min
advertised_rate_max
psk_min
psk_max
eligibility_conditions
optional_service_conditions
```

The HTML is still useful.

The observation model simply needs to preserve what the bank actually states.

---

# 8. Why headline rate pages are not enough

The project should distinguish:

```text
page contains a rate
```

from:

```text
page contains the full tariff surface
```

A marketing page saying:

```text
up to 14%
```

is useful as a sensor but insufficient to reconstruct all product pricing dimensions.

A strong rate surface may instead contain:

```text
product
term
amount band
base rate
promotional rate
client segment
new-money condition
channel
effective date
```

Both page types may be collected, but they must not be treated as semantically equivalent.

---

# 9. Raw HTML versus rendered HTML

This is the most important remaining technical question.

A public web page can expose rates to a user while the initial HTTP response contains only an application shell:

```text
initial HTML
→ JavaScript
→ API request
→ React/Vue render
→ visible rate
```

Therefore each candidate surface must be tested for two representations:

```text
raw_response_html
rendered_dom_html
```

The preferred collector logic is:

```text
1. ordinary HTTP fetch
2. inspect whether useful page content is present
3. if not, generic Playwright fallback
4. page.content()
5. save rendered HTML
```

This requires no bank-specific selectors.

The browser simply serializes what the bank rendered for the user.

This is fully consistent with the desired newsflow contract:

> the HTML stored in newsflow should already contain the rates.

---

# 10. What newsflow should store

The primary object should be a **rate-surface HTML representation**.

Conceptually:

```text
source_entity = otp-bank
surface_id = retail-deposit-horizon
surface_type = deposit_offer
source_url = ...
retrieved_at = ...
fetch_mode = http | browser
representation_type = raw_html | rendered_html
content_sha256 = ...
```

Then the actual HTML is stored as the observed representation.

`newsflow` does not need to understand the structure.

It needs to prove:

```text
at time T
the official source returned/rendered representation R
```

---

# 11. What newsflow should not do at collection time

The collector should not contain bank-specific business logic such as:

```text
CSS selector for rate
XPath for term
regex for amount
special case for promotional block
```

That is precisely the maintenance burden the architecture is intended to eliminate.

The collector should only know:

```text
where to fetch
how to fetch
when it was fetched
what bytes/DOM were obtained
whether retrieval succeeded
```

Interpretation belongs downstream.

---

# 12. LLM role

The LLM should receive HTML that already contains the relevant rate information.

Its task is semantic extraction.

Example:

```text
Read this official bank HTML.

Extract every explicitly published rate observation.

Preserve:
- exact product name;
- product type;
- term;
- amount range;
- rate;
- rate range;
- full-cost-of-credit range;
- promotional/base distinction;
- segment/eligibility;
- effective date;
- conditions.

Do not infer missing values.
Do not normalize a product into another product.
Do not calculate derived rates.
Use UNKNOWN when the source is ambiguous.
```

This is a far more appropriate use of LLMs than trying to use an LLM as a web crawler.

---

# 13. Historical daily series

The evidence layer does not need to store one duplicated HTML file for every calendar day.

The system can preserve unique representations plus an observation ledger.

Example:

```text
2026-09-21 → representation A
2026-09-22 → representation A
2026-09-23 → representation A
2026-09-24 → representation B
2026-09-25 → representation B
```

Then:

```text
representation A → extracted tariff state A
representation B → extracted tariff state B
```

A deterministic daily view can resolve:

```text
date
× bank
× product
× term
× condition
→ observed public rate
```

The daily state remains provable because each day has an explicit successful observation of the representation.

Important statuses must remain distinct:

```text
OBSERVED_SAME_REPRESENTATION
OBSERVED_NEW_REPRESENTATION
FETCH_FAILED
SOURCE_UNAVAILABLE
UNPARSED
UNKNOWN
```

Therefore:

```text
missing != unchanged
```

---

# 14. Historical backfill

Many banks already expose historical versions.

Two reconstruction mechanisms can therefore coexist.

## Historical official archive

```text
official archived tariff
→ effective_from
→ historical observation
```

## Prospective newsflow monitoring

```text
daily/frequent fetch
→ first_observed_at
→ representation history
```

Where both exist, they can cross-validate one another.

Strong historical candidates include:

- Yandex Bank;
- RSHB;
- Alfa-Bank;
- MTS Bank;
- PSB;
- VTB;
- Ozon Bank;
- Sovcombank;
- others with dated tariff/document archives.

---

# 15. Main architectural conclusion

The project does **not** need to choose between:

```text
build 20 custom parsers
```

and:

```text
give everything to an LLM
```

A much stronger division of labor is available.

## Deterministic layer

```text
URL registry
HTTP/browser fetch
timestamps
status codes
provenance
HTML preservation
hashing
deduplication
change history
```

## LLM layer

```text
semantic interpretation
table understanding
product identification
condition extraction
rate extraction
ambiguity handling
```

This preserves source fidelity while avoiding source-specific scraping logic.

---

# 16. What is now sufficiently established

The following hypotheses are supported by the research.

## Hypothesis 1

> Banks do not expose enough numeric rate data in HTML, so the idea would only work for a few institutions.

Not supported.

Rate-bearing HTML is widespread across the current banking universe.

## Hypothesis 2

> Most pages only contain advertising headlines like “up to X%”.

Not supported.

Several banks expose full or near-full matrices directly in web pages.

Examples include source classes found at:

- OTP Bank;
- Sovcombank;
- Ozon Bank;
- Yandex Bank;
- PSB;
- MKB;
- Bank Saint Petersburg;
- others.

## Hypothesis 3

> Every bank will require a separate HTML parser.

Not supported.

A generic HTML/render collector plus downstream LLM interpretation appears viable.

## Hypothesis 4

> Every site can be collected with a simple HTTP GET.

Not yet established.

Some sources are likely JavaScript-dependent.

A generic browser-render fallback is required.

---

# 17. What must be researched next

The next research step should **not** be agent orchestration.

It should be an exhaustive **rate-surface registry audit**.

For every bank currently in the Tabularium universe, identify all relevant official pages for:

```text
term deposits
savings accounts
consumer loans
mortgages
auto loans
credit cards
business deposits where relevant
```

For each candidate URL, record:

```text
bank
surface_id
source_url
product_scope

rates_in_raw_html
rates_in_rendered_html

requires_javascript
requires_interaction
requires_authentication

contains_numeric_rate
contains_full_matrix
contains_effective_date
contains_previous_versions
contains_links_to_formal_tariffs

stable_url
fetch_status
recommended_fetch_mode
```

The empirical objective is to answer:

```text
What percentage of the required bank × product universe can be captured by:

1. plain HTTP HTML;
2. generic Playwright-rendered HTML;
3. document fallback only?
```

Only after this census should the production collector architecture be finalized.

---

# 18. Suggested decision criterion

The HTML-first architecture should be considered successful if the overwhelming majority of economically relevant rate surfaces can be obtained through:

```text
HTTP GET
or
generic browser rendering
```

without source-specific extraction code.

The current research strongly suggests this threshold is achievable, especially for retail deposits and savings accounts.

---

# 19. Target architecture

```text
                 OFFICIAL BANK WEBSITE
                          │
                    known rate URL
                          │
              ┌───────────┴───────────┐
              │                       │
          HTTP fetch             Playwright
              │                       │
              └───────────┬───────────┘
                          ▼
                 RATE-BEARING HTML
                          │
                          ▼
                      NEWSFLOW
                          │
                 timestamp / hash
                          │
                          ▼
                    LLM EXTRACTION
                          │
                          ▼
              STRUCTURED OBSERVATIONS
                          │
                          ▼
                 TABULARIUM / BRIDGE
                          │
                          ▼
             DAILY BANK-RATE TIME SERIES
```

The critical property is:

> `newsflow` receives HTML in which the bank's published rate information is already present.

The collector does not need to understand that HTML.

That separation is the core reason the architecture can scale from a handful of banks to dozens.

---

# 20. Final assessment

The HTML-first hypothesis is strong enough to continue.

The evidence found so far indicates that official rate-bearing web pages are not exceptional. They are a common publication surface across major Russian banks.

For deposits and savings accounts, they may be sufficient to become the primary daily observation mechanism.

For loans, the same mechanism remains highly useful but requires a richer observation model because public pricing is often published as ranges or conditional envelopes rather than a single rate.

The next decisive step is therefore:

> build a complete bank × product × URL rate-surface registry and experimentally classify every surface as raw-HTML, rendered-HTML, or document-fallback.

That registry will determine the real coverage ceiling of the architecture before any large-scale agent automation is built.
