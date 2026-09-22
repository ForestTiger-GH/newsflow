# Мониторинг курсов валют крупных банков: исследование для `newsflow`

**Дата исследования:** 2026-09-23  
**Статус:** Research / epoch-bound material  
**Репозиторий назначения:** `ForestTiger-GH/newsflow`  
**Предлагаемый route:** `_mw/epochs/001-foundation/research/01-base-ideas/bank-fx-rate-monitoring-research.md`  
**Не является:** контрактом Tabularium, поддерживаемой Product truth или автоматически принимаемым evidence-layer.

---

## 1. Вопрос исследования

Цель — понять, можно ли построить в `newsflow` устойчивый автоматический мониторинг официально публикуемых курсов покупки/продажи иностранной валюты крупными российскими банками без опоры на PDF и без необходимости ежедневно отдавать каждую страницу LLM.

Практический вопрос формулируется так:

> можно ли регулярно получать банковские валютные котировки как машиночитаемое наблюдение непосредственно из публичной web-поверхности банка, сохраняя provenance и семантику котировки, а LLM использовать лишь как исключение?

Исследование продолжает две уже выполненные работы `newsflow`:

- `bank-rate-bearing-html-research.md` — мониторинг процентных ставок через rate-bearing HTML;
- `github-actions-agent-gateway.md` — разделение постоянного детерминированного наблюдения и агентной/LLM-интерпретации.

В отличие от процентных ставок, валютные котировки в целом выглядят **значительно более пригодными для полной или почти полной детерминированной автоматизации**.

---

# 2. Краткий вывод

## 2.1 Основной результат

Для банковских валютных курсов **не следует переносить один-в-один архитектуру мониторинга процентных ставок**.

Для ставок разумная схема была:

```text
rate-bearing HTML
→ newsflow
→ LLM semantic extraction
→ structured observations
```

Для валют сильнее другая лестница:

```text
1. first-party JSON / XML / documented API
        ↓ если нет
2. stable first-party HTML table
        ↓ если нет
3. generic Playwright + network interception
        ↓ если endpoint не удаётся стабильно воспроизвести
4. rendered DOM snapshot + deterministic extraction
        ↓ только при semantic drift / ambiguity
5. LLM fallback
```

То есть LLM должен стать **не штатным парсером каждой ежедневной котировки, а исключением**.

## 2.2 Почему валюты проще ставок

Курс валюты обычно представлен значительно более регулярной структурой:

```text
currency pair
buy
sell
nominal
channel
amount tier
location / office
updated_at
```

В отличие от депозитных или кредитных ставок, не требуется каждый раз интерпретировать сложную комбинацию срока, сегмента клиента, новых денег, промоусловия, способа выплаты процентов и других продуктовых признаков.

Главная сложность валют не в извлечении числа. Главная сложность — **не потерять контекст конкретной котировки**.

## 2.3 Реалистичность полной автоматизации

Вывод исследования:

```text
полная автоматизация для значительной части банков — реалистична;
почти полная автоматизация для широкого universe — реалистична;
полностью универсальный zero-maintenance collector для всех банков — не доказан.
```

Наиболее вероятный production-профиль:

```text
80–95% штатных runs:
    deterministic HTTP/API/HTML

редкие failures / schema drift:
    browser diagnostics

редкие semantic changes:
    LLM / human review
```

Процент здесь — архитектурная оценка, а не измеренный coverage текущего bank universe. Точный показатель можно получить только после отдельного onboarding-аудита endpoint’ов каждого банка.

---

# 3. Что именно означает «официальный курс банка»

Это критически важно. У банка почти никогда нет одного универсального значения вида:

```text
USD buy = X
USD sell = Y
```

Один банк может одновременно публиковать разные цены для:

- наличного обмена;
- безналичного обмена;
- мобильного приложения;
- интернет-банка;
- операции по банковской карте;
- банкомата;
- конкретного отделения;
- конкретного города;
- разных сумм операции;
- премиального сегмента;
- специальных подписок;
- разных типов/годов выпуска банкнот;
- индивидуальной котировки.

Это подтверждается текущими официальными страницами банков.

### Газпромбанк

Официальная страница прямо говорит, что на сайте показывается лучший курс в городе, а курсы разных офисов могут отличаться. Для обмена свыше 1 000 единиц иностранной валюты и для Premium/Private существуют отдельные условия; онлайн-канал также выделяется отдельно.

Source: https://www.gazprombank.ru/personal/courses/

### Уралсиб

Публичная страница различает кассу и онлайн, а опубликованная котировка может быть привязана к региону и порогу суммы — например, к сделкам от эквивалента 10 000 USD.

Source: https://uralsib.ru/kursy-valyut

### Совкомбанк

Для получения актуального наличного курса требуется выбрать конкретный офис; банк отдельно предупреждает, что курс действует на указанное время и финальная цена определяется в момент операции.

Source: https://sovcombank.ru/apply/currency/city-moskva/

### ПСБ

Публичная web-поверхность одновременно показывает наличный и интернет-банковский курс, а банк указывает, что котировки могут меняться в течение дня.

Source: https://www.psbank.ru/personal/rates

### Т-Банк

Публичная страница показывает котировки для разных клиентов и операций и публикует timestamp вида «действительны на HH:MM по Москве».

Source: https://www.tbank.ru/about/exchange/

Следовательно, ключевой semantic rule:

```text
same currency code ≠ same rate construct
```

Иначе можно построить технически идеальный, но экономически неверный временной ряд.

---

# 4. Правильный объект наблюдения

Минимальная структурированная валютная котировка должна быть ближе к следующему объекту:

```yaml
bank: gazprombank
source_url: https://...
retrieved_at: 2026-09-23T12:31:14+03:00
source_updated_at: 2026-09-23T12:29:00+03:00

from_currency: USD
to_currency: RUB
nominal: 1

bank_buy: 84.10
bank_sell: 86.40

channel: cash_office
location_scope: city
city: Moscow
office_id: null

amount_lower_limit: 0
amount_upper_limit: null
client_segment: standard
banknote_class: null

representation_id: sha256:...
collector_version: ...
mapping_version: ...
status: OBSERVED
```

Это не предлагается как окончательная schema `newsflow`. Это исследовательская иллюстрация того, какие измерения нельзя потерять.

## 4.1 Обязательные смысловые измерения

Минимально нужно различать:

```text
bank
currency_from
currency_to
nominal
bank_buy
bank_sell
channel
geography / office
amount tier
client segment
source_updated_at
retrieved_at
```

По необходимости добавляются:

```text
banknote series / quality
premium flag
subscription / package
operation type
valid_from
valid_to
cross-rate
```

## 4.2 Особая осторожность с buy / sell

Нельзя нормализовать направление только по имени поля `buy` / `sell`.

Разные API и интерфейсы могут описывать направление с точки зрения:

- банка;
- клиента;
- исходной валюты;
- целевой валюты.

Например, сторонний открытый код для ВТБ уже содержит ручную перестановку полей `BankSellAt` / `BankBuyAt` при приведении к пользовательской модели. Это хороший пример того, почему первоначальное semantic binding должно быть проверено по видимой официальной странице, а не угадано по JSON key.

---

# 5. Дневной ряд и внутридневные изменения — это разные вещи

Пользовательская формулировка «ежедневный курс банка» создаёт риск ложной точности.

Многие банки прямо говорят, что курс меняется **в течение дня**. Следовательно, одно наблюдение за календарный день — это не «курс банка за день», а:

```text
курс, наблюдавшийся в конкретный момент времени T
```

## 5.1 Минимальный режим

Если нужен именно один daily snapshot:

```text
fixed observation time
например 12:30 Moscow
```

и всегда сохранять:

```text
retrieved_at
source_updated_at (если банк его публикует)
```

Сравнивать банки в разные часы нельзя без явной оговорки.

## 5.2 Более сильный режим

Для реального анализа поведения банков лучше:

```text
poll hourly during the day
→ store only changed source states / quote events
→ later derive daily views
```

Тогда можно получить:

```text
first observed quote of day
last observed quote of day
intraday min/max
number of repricings
exact first_observed_at of each change
spread dynamics
```

`newsflow` при этом хранит наблюдения/representations, а дневной OHLC-подобный ряд является downstream derivation.

## 5.3 GitHub Actions подходит, но не как биржевая low-latency инфраструктура

GitHub Actions поддерживает cron с минимальным интервалом 5 минут, однако GitHub прямо предупреждает, что scheduled jobs могут задерживаться в периоды высокой нагрузки, особенно в начале часа, а при высокой нагрузке отдельные runs могут быть отброшены.

Поэтому Actions подходит для:

```text
daily / hourly / half-hourly observation
```

но не следует трактовать его как гарантированный tick collector.

GitHub documentation:

- https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax
- https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows

---

# 6. Источники: практическая лестница

## Tier A — документированный официальный API/feed

Лучший вариант, если он реально пригоден эксплуатационно.

Плюсы:

- explicit machine-readable contract;
- меньше зависимости от DOM;
- timestamp и dimensions часто уже структурированы;
- дешёвая регулярная загрузка.

Но слово «API» само по себе не гарантирует удобство.

### Газпромбанк: реальный пример

Газпромбанк публично предлагает «Открытый API для получения курсов банка».

Однако подключение требует:

- изучения инструкции;
- подачи заявки;
- данных организации и ИНН;
- мобильного подтверждения;
- указания IP-адресов, с которых будут выполняться вызовы.

Source: https://www.gazprombank.ru/personal/page/openapi/

Это значит, что такой API может быть сильным source contract, но неудобен для обычных GitHub-hosted runners.

GitHub сам указывает, что стандартные hosted runners используют большой динамический набор IP-диапазонов и не рекомендует allowlisting этих диапазонов; для статического egress рекомендуются larger runners со static IP или self-hosted runner.

Source: https://docs.github.com/en/actions/reference/runners/github-hosted-runners

Следовательно:

```text
best semantic API ≠ best operational collector
```

## Tier B — first-party JSON/XML endpoint, которым пользуется публичная страница

Это, вероятно, **главный practical sweet spot**.

Многие современные сайты работают так:

```text
HTML shell
→ browser JS
→ GET /api/...rates...
→ JSON
→ render table
```

Если endpoint можно вызвать напрямую без пользовательской авторизации, ежедневный сбор превращается в обычный HTTP GET.

### Обнаруженные кандидатные endpoint’ы

В ходе desk research по открытым GitHub-проектам обнаружены first-party URL, которые сторонние программы использовали для получения банковских курсов:

#### Альфа-Банк

```text
https://alfabank.ru/api/v1/scrooge/currencies/alfa-rates
```

Сторонние реализации передают параметры `currencyCode`, `rateType`, `clientType`, дату и получают структуру с `lastActualRate`, `buy`, `sell` и timestamp.

Discovery references:

- https://github.com/dukei/any-balance-providers/blob/master/providers/ab-exchange-alfabank/main.js
- https://github.com/Erquilenne/currency_store/blob/master/pkg/parser/parser.go

#### ВТБ

```text
https://www.vtb.ru/api/currency-exchange/table-info
```

В открытом стороннем коде response описан полями типа:

```text
FromCurrency
ToCurrency
StartDate
BankSellAt
BankBuyAt
```

Discovery reference:

- https://github.com/DevNulPavel/rust_examples/blob/master/my/097_currencies_request/currency_lib/src/banks/vtb.rs

#### Россельхозбанк

```text
https://www.rshb.ru/api/v1/rates
```

Сторонний код ожидает записи вида:

```text
currencyPair
buyRate
sellRate
lastUpdatedAt
```

Discovery reference:

- https://github.com/drGOD/rshb_unionpay_converter/blob/main/app.py

### Важное ограничение provenance

Эти endpoint’ы **не должны считаться подтверждёнными source contracts только потому, что встречаются в стороннем GitHub-коде**.

Правильная цепочка:

```text
third-party code
→ discovery candidate
→ direct first-party replay
→ compare with official visible page
→ confirm semantic mapping
→ only then admit as collector profile
```

В текущем исследовательском окружении прямой вызов этих API URL не удалось проверить сетевым инструментом, поэтому здесь они намеренно помечены как **CANDIDATE_ENDPOINT**, а не VERIFIED_API.

## Tier C — обычный first-party HTML

Для ряда банков это уже достаточно.

### ПСБ

Текущая публичная страница отдаёт web-readable таблицу:

- наличные;
- безналичные;
- интернет-банк;
- покупка;
- продажа.

Source: https://www.psbank.ru/personal/rates

### ОТП Банк

Публичная страница отдаёт в HTML полноценную таблицу для безналичного и наличного обмена, включая несколько валют, кросс-курс и дополнительные условия.

Source: https://www.otpbank.ru/retail/currency/

### МТС Банк

Публичная страница покупки/продажи наличной валюты содержит сами значения USD/EUR и explicit timestamp обновления.

Source: https://www.mtsbank.ru/factory/pokupka-valyuty/

### Уралсиб

Публичная страница отдаёт значения в HTML вместе с регионом, каналом и amount condition.

Source: https://uralsib.ru/kursy-valyut

Для таких банков LLM для извлечения пары чисел экономически и технически не оправдан.

## Tier D — rendered page / network discovery

У ряда страниц видимая пользователю котировка существует, но текстовый crawler видит только заголовки или пустые места.

Характерные кандидаты:

- ВТБ;
- Т-Банк;
- отдельные поверхности Альфа-Банка;
- Банк Санкт-Петербург;
- офисные/географические интерфейсы Совкомбанка и Газпромбанка.

Здесь нужен не bank-specific parser, а generic onboarding browser:

```text
Playwright
→ load canonical page
→ record all network responses
→ inspect JSON/XML candidates
→ replay candidate directly
```

Если удаётся воспроизвести endpoint — production collector возвращается к Tier B.

Если нет — сохраняется rendered DOM и применяется generic deterministic extraction либо, в крайнем случае, LLM.

## Tier E — LLM fallback

LLM нужен, когда:

- поменялась структура источника;
- появились новые rate tiers;
- неясна семантика полей;
- visible DOM содержит сложную таблицу без стабильной машинной структуры;
- нужно сопоставить новую representation со старым construct;
- обнаружен конфликт между API и видимой страницей.

LLM **не должен** вызываться только потому, что наступил новый день.

---

# 7. Почему network interception сильнее, чем написание 20 scraper’ов

В ставочном исследовании ключевая идея была сохранить rate-bearing HTML, не кодируя десятки CSS selectors.

Для валют можно пойти ещё на шаг дальше.

## 7.1 Generic endpoint discovery

При onboarding нового банка можно один раз выполнить:

```text
1. открыть официальную страницу в Playwright;
2. подписаться на response events;
3. сохранить URL / method / status / content-type;
4. отметить JSON, XML и маленькие HTML responses;
5. искать ISO currency codes: USD, EUR, CNY, RUB...;
6. искать повторяющиеся numeric pairs;
7. искать timestamp-поля;
8. изменить UI control: cash ↔ online, city, amount;
9. посмотреть, какой request изменился;
10. replay endpoint без браузера;
11. сопоставить response с visible values;
12. зафиксировать source profile.
```

Это можно автоматизировать в значительной степени без LLM.

## 7.2 Scoring кандидатов

Пример эвристики:

```text
+ first-party host
+ application/json / xml
+ 2+ ISO 4217 codes
+ repeated objects
+ two numeric rate-like values
+ updatedAt / date / time
+ response triggered by rate page
+ values change when user switches channel/city/amount
```

Высокий score → endpoint candidate.

## 7.3 Почему всё равно нужна первоначальная semantic verification

Автоматика не должна самостоятельно решить, что:

```text
buyRate = bank buys USD from client
```

только по имени поля.

Первоначальный mapping должен подтверждаться сравнением:

```text
API response
↔ exact values and labels visible on official page
```

После этого ежедневный run может быть полностью детерминированным.

---

# 8. Исторический Yandex YML: полезная находка, но не production dependency

Яндекс ранее документировал специальный валютный YML/XML feed для банков.

Схема включала:

```text
From
To
AmountLowerLimit
Nominal
BuyPrice
SellPrice
UpdateAt
Offices
City
Address
```

То есть сама структура практически совпадает с тем, что нужно `newsflow` для валютных котировок.

Архивная документация:

https://yandex.ru/support/webmaster/ru/search-appearance/07052025/banks

Текущая документация Яндекса сообщает, что отображение дополнительной информации об обмене валют приостановлено:

https://www.yandex.ru/support/webmaster/ru/search-appearance/banks

Следовательно, важный вывод двоякий:

1. банки исторически имели стимул формировать машиночитаемые feed’ы с курсами, amount tiers и офисами;
2. нельзя предполагать, что такие feed’ы сегодня публичны и поддерживаются.

В desk research не был найден надёжный текущий публичный YML-feed конкретного крупного банка, который можно было бы немедленно взять как универсальную основу.

Поэтому YML следует использовать как **discovery hint**:

```text
robots.txt
sitemap
page source
network requests
common filenames: feed.xml, currency.yml, rates.xml ...
```

но не как обязательный слой архитектуры.

---

# 9. Банк-за-банком: предварительная карта пригодности

Статусы ниже отражают именно это исследование на 2026-09-23, а не постоянный контракт источника.

Условные классы:

```text
A — сильная детерминированная public surface уже видна
B — public surface подтверждена, вероятен прямой deterministic collector
C — нужна network/browser discovery
D — публичная котировка не подтверждена / auth-only / scope unclear
N/A — retail FX surface может быть экономически несущественна для данного банка
```

| Банк | Наблюдаемая official surface | Предварительный класс | Комментарий |
|---|---|---:|---|
| Сбер | известна публичная страница `sberbank.ru/ru/quotes/currencies`, но прямой fetch в текущем окружении timeout | C | onboarding через Playwright/network capture |
| ВТБ | официальная страница курсов и конвертер; values динамические | C → B | найден candidate JSON endpoint `api/currency-exchange/table-info` |
| Газпромбанк | официальная страница + формальный Open API | A/C | API требует onboarding и IP allowlist; public page отдельно имеет office/location semantics |
| Альфа-Банк | публичная currency surface; найден first-party JSON candidate | C → B | `api/v1/scrooge/currencies/alfa-rates`, требует direct verification |
| Россельхозбанк | официальная страница операций с наличной валютой и безналичного обмена | B/C | найден candidate `rshb.ru/api/v1/rates`; проверить coverage currencies/channels |
| ПСБ | полноценные наличные и интернет-банковские котировки в web-readable HTML | A | хороший HTTP-first pilot |
| МКБ | существование публичных курсов подтверждается внешними наблюдателями, но stable first-party rate surface в desk research не закреплена | C/D | отдельный Playwright discovery; не использовать aggregator как evidence |
| Банк ДОМ.РФ | подтверждён обмен в ДБО, но сильная открытая retail FX surface не найдена | D | возможно auth-only / низкий public value |
| Т-Банк | официальная timestamped public page | B/C | сильный кандидат на network endpoint discovery |
| Совкомбанк | official city/office exchange pages | B/C | geography/office — часть construct, а не metadata decoration |
| ОТП Банк | богатая web-readable таблица | A | HTTP-first; проверить city-specific variation |
| МТС Банк | web-readable USD/EUR + explicit updated time | A | очень простой baseline pilot для cash rates |
| Ак Барс Банк | операции и текущие курсы существуют, но desk research не закрепил простой official machine surface | C | Playwright + network discovery |
| Банк Санкт-Петербург | official exchange page, каналы cash/card/online | B/C | browser/network discovery для numeric payload |
| Уралсиб | web-readable values + channel/region/amount semantics | A | сильный deterministic pilot |
| ВБРР | обмен валют существует, но stable first-party public rate surface не закреплена | C/D | discovery отдельно; агрегаторы не являются source authority |
| МСП Банк | массовый retail FX не является очевидной основной surface | D/N/A | исследовать только если это входит в analytical universe |
| Ozon Банк | публичная массовая retail FX surface не подтверждена | D/N/A | не создавать искусственный coverage |
| Яндекс Банк | публичная массовая retail FX surface не подтверждена | D/N/A | не путать с исторической документацией Yandex Webmaster |

## 9.1 Важная трактовка таблицы

`C` не означает «банк нельзя автоматизировать».

Чаще это означает:

```text
FETCH_MECHANISM_NOT_YET_BOUND
```

а не:

```text
NO_MACHINE_READABLE_SOURCE
```

Опыт со ставками и найденные candidate endpoint’ы показывают, что после одного browser/network onboarding многие `C` с высокой вероятностью превратятся в `B`.

---

# 10. Первичные страницы, проверенные в исследовании

Ниже — first-party public surfaces, полезные для дальнейшего onboarding.

## Газпромбанк

- Курсы: https://www.gazprombank.ru/personal/courses/
- Open API: https://www.gazprombank.ru/personal/page/openapi/

## ВТБ

- Курсы: https://www.vtb.ru/personal/platezhi-i-perevody/obmen-valjuty/
- Конвертер: https://www.vtb.ru/personal/platezhi-i-perevody/konverter/

## Россельхозбанк

- Операции с наличной иностранной валютой: https://www.rshb.ru/natural/currency-transactions

## ПСБ

- Курсы: https://www.psbank.ru/personal/rates
- USD: https://www.psbank.ru/personal/rates/usd
- CNY: https://www.psbank.ru/personal/rates/cny

## Т-Банк

- Курсы: https://www.tbank.ru/about/exchange/

## Совкомбанк

- Москва / выбор офиса: https://sovcombank.ru/apply/currency/city-moskva/

## ОТП Банк

- Курсы: https://www.otpbank.ru/retail/currency/

## МТС Банк

- Наличная валюта: https://www.mtsbank.ru/factory/pokupka-valyuty/

## Банк Санкт-Петербург

- Курсы: https://www.bspb.ru/finance/exchange

## Уралсиб

- Курсы: https://uralsib.ru/kursy-valyut

---

# 11. Что должен хранить `newsflow`

Исходя из текущего repo contract, `newsflow` не должен сразу превращаться в аналитическую базу валютных рядов.

Предпочтительная цепочка:

```text
SOURCE
official bank page / API
        ↓
REPRESENTATION
raw JSON / XML / HTML / rendered DOM
        ↓
OBSERVATION
retrieval event + timestamp + hash + source metadata
        ↓
EXTRACTION
normalized quote record
        ↓
DERIVATION
hourly/daily series, spreads, repricing events
        ↓
ANALYTICAL CLAIM
bank X widened spread, repriced earlier than bank Y, etc.
```

## 11.1 Raw evidence envelope

Даже если JSON маленький, желательно сохранять:

```text
source_url
request_method
query parameters / request body where relevant
retrieved_at
HTTP status
Content-Type
ETag
Last-Modified
response bytes / response text
SHA-256
collector version
```

Если страница требует браузера:

```text
fetch_mode = browser
final_url
rendered_html_hash
selected network response(s)
```

## 11.2 Не надо хранить ежедневные дубликаты bytes

Как и в rate-bearing HTML research, можно дедуплицировать representations.

Например:

```text
09:00 → representation A
10:00 → representation A
11:00 → representation B
12:00 → representation B
13:00 → representation C
```

Хранить уникальные payload’ы + ledger наблюдений.

Так сохраняется важное различие:

```text
same representation observed again
≠
no observation happened
```

---

# 12. Рекомендуемые статусы observation pipeline

Минимально полезны:

```text
OBSERVED_NEW_REPRESENTATION
OBSERVED_SAME_REPRESENTATION
FETCH_FAILED
HTTP_BLOCKED
AUTH_REQUIRED
SOURCE_UNAVAILABLE
STALE_SOURCE_TIMESTAMP
SCHEMA_DRIFT
SEMANTIC_DRIFT
PARSE_FAILED
UNKNOWN
```

Особенно важно:

```text
FETCH_FAILED ≠ unchanged rate
```

и:

```text
missing rate ≠ zero rate
```

---

# 13. Validation без подмены первоисточника

Для автоматического контроля можно применять несколько уровней.

## 13.1 Structural validation

```text
expected currencies present?
expected keys present?
rate values numeric?
nominal positive?
source_updated_at parsable?
content non-empty?
```

## 13.2 Freshness validation

Если source публикует timestamp:

```text
retrieved_at - source_updated_at
```

можно сравнивать с допустимым freshness window.

Слишком старый timestamp:

```text
STALE_SOURCE_TIMESTAMP
```

но значение не заменяется другим источником.

## 13.3 Economic anomaly checks

Допустимо использовать ЦБ/рынок **как detector**, например:

```text
rate differs by > X%
spread exploded
rate moved 20% in one observation
```

Но результат должен быть:

```text
ANOMALY_FLAG
```

а не «исправленное» значение.

## 13.4 Cross-representation check

Если доступны одновременно:

```text
bank JSON
bank visible HTML
```

можно периодически проверять совпадение.

При несовпадении:

```text
CONFLICT
```

и обе representations сохраняются.

---

# 14. Schema drift и semantic drift

Это разные виды поломки.

## Schema drift

Например:

```text
buyRate → buy
JSON nesting changed
endpoint path changed
```

Здесь часто достаточно deterministic adaptation.

## Semantic drift

Гораздо опаснее:

```text
единый наличный курс
→ разные amount tiers

городской курс
→ office-specific

standard client
→ standard + premium

USD cash
→ separate new / old banknotes
```

Автоматически продолжать старый ряд нельзя.

Правильное событие:

```text
SEMANTIC_DRIFT
```

и далее LLM/human review обновляет mapping version.

---

# 15. Роль LLM

LLM здесь всё ещё полезен, но его роль существенно уже, чем в мониторинге ставок.

## Не использовать LLM для

```text
каждый день прочитать JSON
вытащить USD buy/sell
переписать цифры в таблицу
```

Это ненужная стоимость и дополнительный источник ошибок.

## Использовать LLM для

```text
новая незнакомая страница
новый rate tier
неоднозначная семантика buy/sell
сопоставление payload с visible UI
schema/semantic drift diagnosis
обнаружение изменившихся условий
редкий DOM-only fallback
```

## Сильный operating pattern

```text
normal run:
    no LLM

collector failed:
    capture diagnostics

candidate new endpoint found:
    deterministic tests

semantics unclear:
    LLM review

material mapping changed:
    human/agent-approved new mapping_version
```

Это превращает LLM из постоянной инфраструктурной зависимости в **exception processor**.

---

# 16. GitHub Actions: целевой контур

Концептуально:

```text
schedule
  ↓
source registry
  ↓
HTTP collectors ──────────────┐
  ↓                           │ failure / drift
raw representation            ▼
  ↓                      diagnostic Playwright
hash / dedupe                 │
  ↓                           ▼
observation ledger       endpoint rediscovery
  ↓                           │
validated quote               ▼
  ↓                      LLM only if needed
change event
```

## 16.1 Не запускать cron ровно в `:00`

GitHub предупреждает о повышенной нагрузке в начале часа.

Вместо:

```cron
0 * * * *
```

лучше, например:

```cron
17 * * * *
```

или иной смещённый minute.

## 16.2 Static IP case

Для API с IP allowlist, как у Газпромбанка, стандартный shared hosted runner неудобен.

Варианты:

```text
A. self-hosted runner со статическим egress
B. larger GitHub runner со static IP
C. отдельный small proxy / serverless gateway со static egress
D. не использовать gated API, если публичная page/API surface даёт тот же construct
```

Нельзя усложнять инфраструктуру только ради статуса «официальный API», если публичный first-party endpoint надёжнее и source-faithful для нужной котировки.

---

# 17. Предлагаемый onboarding algorithm для каждого банка

## Phase 0 — define target constructs

До технической работы определить, что именно нужно получать:

```text
cash standard city-level?
online standard?
USD/EUR/CNY only?
all currencies?
amount tiers?
premium?
office-specific?
```

Иначе collector может успешно автоматизировать не тот курс.

## Phase 1 — official page discovery

Зафиксировать:

```text
canonical page URL
page purpose
public/authenticated
city dependence
channel dependence
amount dependence
published timestamp
```

## Phase 2 — plain HTTP test

```text
GET page
→ do numeric quotes exist in raw HTML?
```

Если да — Tier C.

## Phase 3 — browser network audit

Если нет:

```text
Playwright
→ capture responses
→ identify candidate rate payload
```

## Phase 4 — direct replay

Для каждого candidate endpoint:

```text
fresh HTTP session
no browser state
same headers only if necessary
repeat several times
```

Проверить:

```text
status stability
response schema
fresh timestamp
currency coverage
rate match with UI
```

## Phase 5 — UI perturbation

Изменить:

```text
cash/online
city
amount
currency
```

и проверить, какие request parameters/response blocks соответствуют каждой размерности.

## Phase 6 — semantic binding

Только после совпадения visible labels ↔ payload fields создать mapping.

## Phase 7 — contract tests

Например:

```text
USD and CNY present
buy/sell finite positive
updated_at not older than threshold
known channel id present
schema fingerprint allowed
```

## Phase 8 — production polling

Browser из штатного run исключается, если direct endpoint уже найден.

---

# 18. Рекомендуемый пилот

Не стоит сразу пытаться охватить 20 банков.

Нужен pilot, специально подобранный так, чтобы проверить разные source classes.

## 18.1 ПСБ — HTML baseline

Почему:

- public page;
- cash + online;
- значения видимы в текстовом HTML;
- минимум инфраструктуры.

Цель: доказать простой HTTP → parse → provenance.

## 18.2 МТС Банк — timestamped HTML baseline

Почему:

- compact page;
- USD/EUR;
- explicit update time.

Цель: проверить dual timestamp:

```text
retrieved_at
source_updated_at
```

## 18.3 Уралсиб — dimensionality test

Почему:

- region;
- amount threshold;
- cash/online.

Цель: доказать, что schema сохраняет construct, а не только цифры.

## 18.4 Альфа-Банк — hidden first-party JSON test

Почему:

- обнаружен конкретный candidate endpoint;
- response, судя по независимым open-source clients, уже содержит rate types и timestamps.

Цель: провести полноценную цепочку:

```text
third-party discovery
→ first-party direct verification
→ UI cross-check
→ endpoint admission
```

## 18.5 ВТБ или РСХБ — второй endpoint-discovery case

Почему:

- для обоих найдены concrete candidate endpoints;
- официальный web UX существует;
- можно проверить generic network/replay approach на другом технологическом стеке.

## 18.6 Газпромбанк — gated API / office complexity test

Отдельный optional pilot.

Цель:

- сравнить gated documented API и public surface;
- проверить, оправдывает ли static IP дополнительную инфраструктуру;
- проверить geography/office semantics.

---

# 19. Критерии успеха пилота

Pilot можно считать успешным, если для 5–6 разных банков получается:

```text
1. получить котировку без PDF;
2. получить её без LLM в normal run;
3. сохранять raw first-party representation;
4. воспроизводимо определить channel/location/amount semantics;
5. детектировать unchanged vs changed representation;
6. детектировать stale/schema drift;
7. восстановить provenance для каждого structured quote;
8. после первичного onboarding обходиться без browser на большинстве runs.
```

Особенно сильный критерий:

```text
LLM calls / successful observations << 1%
```

после стабилизации source profiles.

Это не обязательный числовой SLA, а желаемая архитектурная цель.

---

# 20. Что не следует делать

## 20.1 Не строить всё на Banki.ru / Brobank / агрегаторах

Они могут быть полезны как discovery/cross-check signal, но пользовательская задача — официальный курс банка.

Следовательно:

```text
aggregator ≠ evidence authority
```

## 20.2 Не писать сразу bank-specific CSS selectors для каждого сайта

Это создаёт тот же maintenance hell, которого `newsflow` пытается избежать.

Selectors допустимы как последний deterministic adapter, но не как первый universal design.

## 20.3 Не использовать LLM на каждый run

Это ухудшает:

- стоимость;
- воспроизводимость;
- latency;
- debugging;
- auditability.

## 20.4 Не считать retrieval time временем действия курса

Если банк публикует `updated_at`, это отдельное поле.

```text
source_updated_at ≠ retrieved_at
```

## 20.5 Не смешивать cash и online

Даже если значения совпали сегодня.

```text
same value today ≠ same construct
```

## 20.6 Не заполнять пропуски курсом ЦБ

Если банк не отдал значение:

```text
UNKNOWN / FETCH_FAILED
```

а не synthetic CBR substitute.

---

# 21. Возможная registry-конфигурация

Не как принятая schema, а как направление для pilot:

```yaml
source_id: psb-retail-fx
bank: psb
canonical_url: https://www.psbank.ru/personal/rates
fetch:
  mode: http
  expected_content_type: text/html
constructs:
  - channel: cash
  - channel: internet_bank
freshness:
  source_timestamp: optional
validation:
  currencies_required: [USD, EUR]
  schema_policy: monitored
fallback:
  browser: true
  llm: drift_only
```

Для API:

```yaml
source_id: alfa-retail-fx-candidate
bank: alfabank
status: onboarding_candidate
canonical_page: https://alfabank.ru/currency/
endpoint_candidate: https://alfabank.ru/api/v1/scrooge/currencies/alfa-rates
admission:
  direct_replay_verified: false
  ui_mapping_verified: false
```

Последние две строки критичны: discovery ещё не превращает endpoint в accepted source profile.

---

# 22. Daily view как downstream derivation

После накопления intraday observations можно детерминированно построить несколько daily views.

Например:

```text
DAILY_FIRST_OBSERVED
DAILY_LAST_OBSERVED
DAILY_NOON_SNAPSHOT
DAILY_INTRADAY_MIN_BUY
DAILY_INTRADAY_MAX_BUY
DAILY_MIN_SPREAD
DAILY_REPRICING_COUNT
```

Каждое такое значение должно иметь формулу/метод и разрешаться обратно до исходных quote observations.

Не следует называть `DAILY_LAST_OBSERVED` «официальным курсом банка за день» без явного определения.

---

# 23. Сравнение с мониторингом банковских ставок

## Процентные ставки

Типичная проблема:

```text
HTML already contains numbers
but semantic structure is complex
→ LLM useful routinely
```

## Валютные курсы

Типичная проблема:

```text
numbers are structurally simple
but may live behind JS/API
→ endpoint discovery useful
→ LLM mostly exceptional
```

Поэтому для валют оптимальная архитектура должна быть более «программной» и менее «агентной».

Это хороший пример того, что `newsflow` не должен иметь один универсальный extraction strategy для всех source families.

---

# 24. Итоговая рекомендуемая архитектура

```text
                         OFFICIAL BANK
                              │
                  canonical exchange page
                              │
                 ┌────────────┴────────────┐
                 │                         │
        documented API/feed        public web frontend
                 │                         │
                 │                 HTTP raw HTML test
                 │                         │
                 │              ┌──────────┴──────────┐
                 │              │                     │
                 │        rate-bearing HTML     JS application
                 │              │                     │
                 │              │               Playwright once
                 │              │                     │
                 │              │              network discovery
                 │              │                     │
                 └──────────────┴──────────────┬──────┘
                                               ▼
                                machine-readable representation
                                  JSON / XML / HTML payload
                                               │
                                               ▼
                                           NEWSFLOW
                                 retrieval time / hash / status
                                               │
                                      deterministic mapping
                                               │
                                               ▼
                                      FX quote observations
                                               │
                       ┌───────────────────────┴──────────────────────┐
                       │                                              │
                  normal state                                   drift/failure
                       │                                              │
                 no LLM call                              diagnostics / browser
                                                                      │
                                                            semantic ambiguity?
                                                                      │
                                                                LLM / review
                                               │
                                               ▼
                                     downstream daily derivation
                                               │
                                               ▼
                                          analytics
```

---

# 25. Final assessment

Исследование даёт существенно более оптимистичный ответ, чем в случае банковских процентных ставок.

### Что уже достаточно установлено

1. Крупные банки широко публикуют валютные котировки на нормальных web-поверхностях, а не только в PDF.
2. Несколько банков уже отдают полностью web-readable таблицы, которые можно собирать обычным HTTP.
3. Для нескольких динамических сайтов в открытом коде обнаружены конкретные first-party JSON endpoint candidates.
4. Газпромбанк официально предоставляет API для курсов, что дополнительно подтверждает машинную природу этого source class, хотя operational contract этого API неудобен для обычных shared GitHub runners.
5. Browser rendering нужен прежде всего как **инструмент onboarding и rediscovery**, а не обязательно как ежедневный runtime.
6. Главный риск — semantic dimensionality: cash/online/office/city/amount/client segment нельзя свести в одно поле `rate`.
7. LLM для этой задачи нужен главным образом как **drift/ambiguity processor**, а не как обязательный daily extraction layer.

### Практический вердикт

Для `newsflow` стоит разрабатывать валютный мониторинг как **deterministic-first source family**.

Лучший следующий шаг — не проектировать большую универсальную архитектуру и не писать 20 банковских scraper’ов. Нужно сделать bounded pilot на 5–6 банках разных классов и измерить:

```text
HTTP-only coverage
endpoint-discovery success rate
browser-needed share
LLM-needed share
source freshness
schema drift frequency
```

Если pilot подтвердит ожидаемую картину, production-модель может оказаться очень простой:

```text
hourly/daily GitHub Action
→ direct first-party payload
→ hash + provenance
→ deterministic quote extraction
→ commit only change/observation ledger
→ LLM only on exception
```

Для валют это выглядит не просто возможным, а **предпочтительным** по сравнению с LLM-first подходом.

---

# 26. Источники и discovery references

## First-party bank sources

- Газпромбанк — курсы: https://www.gazprombank.ru/personal/courses/
- Газпромбанк — Open API: https://www.gazprombank.ru/personal/page/openapi/
- ВТБ — обмен валюты: https://www.vtb.ru/personal/platezhi-i-perevody/obmen-valjuty/
- ВТБ — конвертер: https://www.vtb.ru/personal/platezhi-i-perevody/konverter/
- Россельхозбанк — наличная валюта: https://www.rshb.ru/natural/currency-transactions
- ПСБ — курсы: https://www.psbank.ru/personal/rates
- Т-Банк — курсы: https://www.tbank.ru/about/exchange/
- Совкомбанк — Москва: https://sovcombank.ru/apply/currency/city-moskva/
- ОТП Банк — курсы: https://www.otpbank.ru/retail/currency/
- МТС Банк — наличная валюта: https://www.mtsbank.ru/factory/pokupka-valyuty/
- Банк Санкт-Петербург — обмен: https://www.bspb.ru/finance/exchange
- Уралсиб — курсы: https://uralsib.ru/kursy-valyut

## Feed / workflow infrastructure

- Yandex Webmaster current banking services documentation: https://www.yandex.ru/support/webmaster/ru/search-appearance/banks
- Yandex historical banking services / currency-feed schema: https://yandex.ru/support/webmaster/ru/search-appearance/07052025/banks
- GitHub Actions workflow syntax: https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax
- GitHub Actions scheduled events: https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows
- GitHub-hosted runner networking/IP: https://docs.github.com/en/actions/reference/runners/github-hosted-runners

## Discovery-only third-party code for candidate first-party endpoints

These are not source authority. They are pointers for first-party verification.

- Alfa-Bank endpoint discovery: https://github.com/dukei/any-balance-providers/blob/master/providers/ab-exchange-alfabank/main.js
- Alfa-Bank independent endpoint discovery: https://github.com/Erquilenne/currency_store/blob/master/pkg/parser/parser.go
- VTB endpoint discovery: https://github.com/DevNulPavel/rust_examples/blob/master/my/097_currencies_request/currency_lib/src/banks/vtb.rs
- RSHB endpoint discovery: https://github.com/drGOD/rshb_unionpay_converter/blob/main/app.py

---

# 27. Research boundary / unresolved questions

Перед переводом идеи в production остаются bounded вопросы:

1. Прямо проверить candidate JSON endpoints Alfa / VTB / RSHB из GitHub Actions-like environment.
2. Для каждого endpoint подтвердить visible-page semantic mapping buy/sell/channel.
3. Провести network audit Т-Банка, Совкомбанка, БСПБ, МКБ, Ак Барса и ВБРР.
4. Решить target scope: только retail cash, только online, или параллельные constructs.
5. Определить sampling policy: daily snapshot либо intraday event monitoring.
6. Проверить robots/ToS/rate limits и разумную частоту запросов для каждой official source surface.
7. Измерить реальные WAF/anti-bot failures на GitHub-hosted runner.
8. Только после empirical pilot решать, нужен ли отдельный registry/schema в `newsflow`.

До этого момента исследование следует трактовать как достаточно сильное основание **для пилота**, но не как окончательно принятый Product contract.
