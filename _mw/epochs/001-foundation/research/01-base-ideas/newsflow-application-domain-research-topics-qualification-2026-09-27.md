# Newsflow — квалификация новых прикладных контуров

**Дата:** 2026-09-27  
**Метод:** MADARAII-07 — Research Topic Development  
**Репозиторий:** `ForestTiger-GH/newsflow`  
**Baseline:** `main`, актуальный корпус Epoch 001 / `01-base-ideas` на момент квалификации  
**Статус:** Research qualification result; темы не считаются приоритетом, Agenda admission или Commission

## 1. Purpose

Определить крупные **конкретные сферы применения** `newsflow`, которые еще не покрыты текущим research corpus и соответствуют его реальной операционной природе:

- unattended execution в GitHub Actions;
- предпочтение публичным HTML / RSS / Atom / JSON / XML / API / renderable web surfaces;
- PDF не должен быть обязательным production-carrier;
- результат должен становиться удобным входом для последующей LLM-обработки;
- простая загрузка готовых таблиц, рядов и bulk datasets относится прежде всего к Tabularium, а не к `newsflow`;
- `newsflow` особенно ценен там, где нужно обнаружить **новое событие или смысловое изменение**, сохранить provenance и превратить изменчивый web-source в адресуемый event/representation.

Decision consumer: дальнейшее исследование и выбор конкретных application contours для развития `newsflow`.

## 2. Existing coverage: что уже не является пробелом

Текущий corpus уже содержит самостоятельную работу по следующим прикладным направлениям:

1. банковские процентные ставки;
2. банковские валютные курсы;
3. физические потоки товарных рынков;
4. погодные сигналы для аграрных commodities;
5. публичные заявления конкретных лиц;
6. общая архитектура GitHub Actions как сенсорного слоя;
7. автономный LLM/gateway contour и zero-cost cloud execution.

Следовательно, новый Topic должен давать новый **consumer job / event class / source family**, а не просто еще один пример того же механизма.

## 3. Граница между newsflow и Tabularium

Используется следующий operational test.

```text
Если источник уже публикует нормализованный ряд / таблицу / bulk dataset
и главная задача — сохранить значения,
→ преимущественно Tabularium.

Если источник публикует страницу, релиз, уведомление, policy/help article,
существенный факт, решение, изменение условий, статус инцидента,
рейтинговое обоснование или иной событийный текст,
а главная задача — обнаружить изменение, понять тип события,
извлечь смысл и дать LLM компактный доказуемый вход,
→ естественная зона newsflow.
```

Гибрид возможен:

```text
structured primary payload
→ deterministic observation
→ semantic event generation
→ newsflow event
→ downstream LLM

reported numeric table itself
→ Tabularium
```

## 4. Математический / алгебраический brainstorm

Чтобы не ограничиться ассоциативным списком идей, application-space рассмотрен как декартово произведение:

```text
A = D × E × S × J
```

где:

- `D` — domain / экономический объект;
- `E` — event primitive;
- `S` — source morphology;
- `J` — downstream consumer job.

### 4.1 Domain families D

1. банки и финансовые сервисы;
2. публичные эмитенты и крупные компании;
3. регуляторы и правовые изменения;
4. государственная поддержка и госспрос;
5. маркетплейсы и цифровые платформы;
6. кредитный риск и рейтинговые агентства;
7. рынок труда и организационные сигналы;
8. цифровые продукты и технологические сервисы;
9. операционная устойчивость и инциденты;
10. международные ограничения и trade controls;
11. строительство / недвижимость / project finance;
12. корпоративные инвестиционные и производственные проекты.

### 4.2 Event primitives E

1. `TERMS_CHANGE` — изменение цены, комиссии, лимита, правила или условия;
2. `RULE_CHANGE` — новое обязательство, запрет, исключение, effective date;
3. `RATING_ACTION` — присвоение, повышение, понижение, outlook/watch;
4. `FORMAL_DISCLOSURE` — существенный факт / корпоративное действие;
5. `OPERATING_CHANGE` — запуск, остановка, производство, guidance, capex;
6. `FUNDING_CHANGE` — размещение, погашение, refinance, dividend, buyback;
7. `RELATION_CHANGE` — собственник, руководство, контроль, партнерство;
8. `OPPORTUNITY` — субсидия, грант, закупка, award, support programme;
9. `INCIDENT` — outage, disruption, cyber/security event;
10. `PRODUCT_RELEASE` — запуск, feature, API/service capability;
11. `ENFORCEMENT` — предписание, штраф, дело, ограничение;
12. `STATEMENT` — заявление / обещание / публичная позиция.

### 4.3 Source morphologies S

1. index + HTML detail page;
2. RSS / Atom;
3. first-party JSON / XML;
4. help / tariff / policy HTML with revision;
5. status API / status page;
6. JS-rendered first-party page.

Такой генератор дает 12 × 12 × 6 = **864 потенциальных source-event cells** до учета consumer job. Это не означает 864 отдельных research topics. Это способ системно проверить пропуски.

### 4.4 Hard filters

Кандидат сохраняется только если одновременно выполняются условия:

```text
public or lawfully retrievable
× recurring/change-bearing
× GitHub-Actions-compatible
× PDF-independent baseline
× semantic delta is material
× LLM-ready output has value
× not merely table ingestion
× not already covered
```

### 4.5 Diagnostic value function

Для brainstorm использовалась не как priority rule, а как фильтр:

```text
Q = V + F + M + P + N - T - A
```

где:

- `V` — decision/use value;
- `F` — unattended fetch feasibility;
- `M` — semantic gain from extraction/diff;
- `P` — quality of primary/public source routes;
- `N` — novelty vs existing newsflow coverage;
- `T` — Tabularium overlap penalty;
- `A` — access / anti-bot / PDF / licensing penalty.

Числовой score не установлен как priority. MADARAII-07 требует отделять value signals от admission и Commission.

## 5. Accounted candidate universe

| Candidate | Disposition | Reason / route |
| --- | --- | --- |
| Банковские ставки | ALREADY COVERED | отдельное исследование существует |
| Банковский FX | ALREADY COVERED | отдельное исследование существует |
| Commodity physical flows | ALREADY COVERED | отдельное исследование существует |
| Ag-weather | ALREADY COVERED | отдельное исследование существует |
| Публичные заявления | ALREADY COVERED | отдельное исследование существует |
| Generic news crawler | MERGED / TOO BROAD | application value возникает только после event/domain binding |
| Regulatory obligations / effective dates | RESEARCH → RT-01 | самостоятельный прикладной контур |
| Санкции / export controls / trade restrictions | RESEARCH → RT-02 | самостоятельная международная risk/event family |
| Существенные факты / corporate actions | RESEARCH → RT-03 | формальный issuer-event stream |
| Рейтинговые действия и rationale | RESEARCH → RT-04 | отдельный risk-intelligence stream |
| Непроцентные банковские тарифы и product terms | RESEARCH → RT-05 | конкурентная pricing/terms intelligence |
| Экономика маркетплейсов и правила продавцов | RESEARCH → RT-06 | высокочастотные terms/rules changes |
| Corporate operating guidance / capex / project events | RESEARCH → RT-07 | семантические операционные события компаний |
| Регуляторные enforcement actions | RESEARCH → RT-08 | supervisory / antitrust / consumer-protection signal |
| Госпрограммы, субсидии, закупки и awards как opportunities | RESEARCH → RT-09 | поток деловых возможностей и спроса |
| Hiring / organizational capability signals | RESEARCH → RT-10 | ранний индикатор изменения capability mix |
| Digital product / API / feature releases | RESEARCH → RT-11 | стратегическая продуктовая динамика |
| Operational / cyber / service incidents | RESEARCH → RT-12 | operational resilience / dependency signal |
| Raw market quotes / candles / order book | TABULARIUM / DATA PIPELINE | structured numeric data, LLM обычно не нужен |
| Макростатистика ЦБ / Росстата как готовые ряды | TABULARIUM | основная ценность — reported observations |
| Полные sanctions lists как CSV/XML | TABULARIUM for raw list; RT-02 for semantic changes | список и change-event имеют разные jobs |
| Warning list ЦБ как JSON/XML | TABULARIUM / RETRIEVAL | готовая структурированная сущность; semantic enforcement отдельно RT-08 |
| Branch / ATM / office directories | TABULARIUM unless event semantics | каталог сам по себе — structured current state |
| Disclosure numeric tables | TABULARIUM | reported rows; event meaning — RT-03 |
| Raw procurement rows | TABULARIUM / RETRIEVAL | если задача только загрузить реестр |
| Procurement requirement / award semantics | MERGED → RT-09 | нужен semantic event |
| Real-estate project open data | TABULARIUM | structured project registry |
| Девелоперские launch/delay/financing events | MERGED → RT-07 | event intelligence |
| M&A / ownership / management changes | MERGED → RT-03 / RT-07 | formal disclosure + operating relation event |
| Debt placements / refinancing / buybacks / dividends | MERGED → RT-03 | формальная issuer-event semantics |
| State aid / grants | MERGED → RT-09 | opportunity semantics |
| App-store releases | MERGED → RT-11 | один из carrier classes |
| Statuspage incidents | MERGED → RT-12 | один из source classes |
| Generic cybersecurity vulnerability feeds | DEFER / DOMAIN-SPECIFIC | легко собираются, но consumer для newsflow пока слишком широк; reopen при заданном target universe |
| Court litigation monitoring | DEFER | высокий access/anti-bot/representation risk; сильная utility, слабая current source contract |
| Bankruptcy / Fedresurs monitoring | DEFER | требует отдельного source/access/legal audit; не квалифицируется автоматически |
| Social sentiment / reviews | DEFER | высокий noise/licensing/identity risk; secondary signal |
| Search trends / web traffic | DEFER | provider dependence и ограниченная provenance |
| Broad press monitoring | DISCOVERY LAYER | secondary discovery, не самостоятельный application contour без event taxonomy |
| Выбор общего event schema | DESIGN | не Research Topic |
| Выбор LLM provider | DESIGN / existing architecture research | уже изучалось |
| Конкретный URL registry audit | CURRENT_STATE_STUDY | object-specific source reconstruction, не новый research topic |
| Выбор cadence | DESIGN / IMPLEMENTATION | определяется после source audit |
| PDF extraction infrastructure | OUTSIDE CURRENT APPLICATION QUALIFICATION | отдельная engineering concern; user constraint — PDF не baseline |

# 6. Qualified research briefs

## RT-01 — Регуляторные обязанности, правила и effective-date intelligence

### Problem

Для банков, компаний и рынков materially важны не только новые документы, а переходы вида:

```text
новое правило
→ кого касается
→ что именно меняется
→ с какой даты
→ переходный период
→ исключения
→ требуемое действие
```

Эта семантика часто опубликована в HTML-релизе, карточке проекта, пояснении регулятора или government page и плохо сводится к простому архивированию документа.

### Research question

Можно ли построить устойчивый Actions-first pipeline, который регулярно превращает изменения официальных web-публикаций регуляторов в provenance-aware regulatory event records для последующей LLM-аналитики?

### Scope

- Банк России;
- regulation.gov.ru;
- Правительство;
- Минфин / Минэкономразвития / профильные ведомства;
- ФАС и иные регуляторы только там, где publication surface устойчив;
- effective dates, transition periods, scope, obligations, exceptions, affected entities.

### Non-goals

- юридическое заключение;
- автоматическое определение обязательности для конкретного банка;
- bulk ingestion нормативных актов;
- PDF-first legal archive.

### Assumptions under test

- official HTML/RSS/index surfaces покрывают достаточно material changes;
- semantic diff ценнее простого new-document alert;
- LLM может извлекать operational meaning при сохранении evidence spans.

### Source classes

- regulation.gov.ru project pages / RSS;
- cbr.ru press / regulation / supervisory pages;
- government.ru decisions and news;
- ministry HTML publications;
- official legal publication metadata where usable without PDF.

### Alternatives / strongest counterarguments

- обычная подписка на новости может быть достаточна;
- полноценная legal database может быть надежнее;
- классификация applicability требует юриста и не должна автоматизироваться.

### Failure modes

- effective date скрыта в attachment;
- draft принят за действующее правило;
- amendment не связан с parent rule;
- summary регулятора упрощает юридический текст;
- LLM превращает interpretation в fact.

### Applicability and cost

Высокая применимость для banking / strategy / compliance intelligence. Средняя source heterogeneity. PDF penalty потенциально высокий, поэтому production baseline должен уметь сохранять event даже если deeper legal text остается attachment-only.

### Evidence strategy

Проверить несколько независимых source families и восстановить lifecycle одного regulatory change от проекта до вступления в силу.

### Stopping / reopen

Остановить исследование, когда понятны устойчивые carriers, lifecycle states, extraction boundary и proportion of materially useful changes available without PDF. Reopen при изменении portal contracts или появлении reliable legal API.

---

## RT-02 — Санкции, экспортный контроль и trade-restriction change intelligence

### Problem

Санкционные и торговые ограничения меняются как через списки лиц, так и через режимы, сектора, товары, лицензии, exemptions, ownership/control rules и даты вступления в силу. Простого diff CSV недостаточно для экономической интерпретации.

### Research question

Как newsflow может отделять raw list changes от semantic rule changes и формировать доказуемые events по санкциям / export controls / trade restrictions без PDF dependence?

### Scope

- UK / EU / US и иные materially relevant official sources;
- Russia-related restrictions;
- designations, delistings, amendments;
- sectoral restrictions;
- export/import controls;
- licenses / exemptions / sunset and effective dates;
- Russian countermeasures only при наличии first-party web surfaces.

### Non-goals

- sanctions legal opinion;
- automatic ownership/control determination;
- complete KYC screening system;
- хранение всех list rows вместо Tabularium.

### Assumptions under test

- machine-readable list formats + HTML change notices можно связать;
- event semantics materially improves downstream research;
- change observation возможен полностью server-side.

### Source classes

- sanctions lists in XML/CSV/HTML;
- official update histories;
- official notices / guidance HTML;
- government/ministry trade restriction pages.

### Alternatives / counterarguments

- коммерческие sanctions databases дают готовый продукт;
- raw structured lists могут быть достаточны для screening;
- semantic rule interpretation слишком specialist-heavy.

### Failure modes

- list diff без rule context;
- correction принят за new sanction;
- name matching creates false entity linkage;
- regime update действует только для узкой юрисдикции;
- third-country ownership/control semantics incorrectly generalized.

### Applicability and cost

Очень высокая для банков, внешней торговли, commodities и issuer analysis. Source access generally favorable; semantic/legal risk high.

### Evidence strategy

Связать list change с official notice и протестировать несколько event types: add, vary, revoke, sectoral rule amendment.

### Stopping / reopen

Research sufficient when raw-vs-semantic boundary, source precedence, jurisdiction fields and specialist escalation are explicit.

---

## RT-03 — Существенные факты, corporate actions и формальный issuer-event graph

### Problem

Системы раскрытия публикуют тысячи формально структурированных, но текстовых сообщений: дивиденды, сделки, решения органов управления, эмиссии, погашения, реорганизации, ratings, собрания, раскрытия по ценным бумагам. Для LLM ценен event graph, а не еще одна копия страницы.

### Research question

Можно ли надежно превратить first-party / authorized disclosure pages в единый issuer-event stream с точной типизацией, entity/security binding и evidence spans?

### Scope

- Интерфакс e-disclosure;
- ПРАЙМ disclosure;
- НРД corporate actions;
- MOEX/issuer official surfaces;
- dividends, meetings, material transactions, issuance, redemption, buybacks, restructurings, governance changes, M&A-related facts.

### Non-goals

- перенос отчетных таблиц;
- автоматическая valuation interpretation;
- duplicate storage of every attachment;
- замена formal disclosure owner.

### Assumptions under test

- HTML bodies несут достаточно event meaning;
- тип раскрытия можно использовать как deterministic prior;
- LLM полезен для subject/object/date/amount/condition extraction;
- duplicate disclosure across systems можно reconcile.

### Source classes

- formal disclosure HTML;
- corporate action indexes;
- issuer press/IR corroboration;
- machine-readable metadata where available.

### Alternatives / counterarguments

- формальные disclosure types уже являются достаточной классификацией;
- bulk database provider может быть удобнее;
- слишком большой volume создаст noise.

### Failure modes

- duplicate mirrors;
- correction/revision lost;
- one disclosure contains several events;
- securities IDs misbound;
- event date, publication date and record date conflated.

### Applicability and cost

Очень высокая для equity, debt, bank and corporate analytics. High volume, but strong source structure makes deterministic prefiltering realistic.

### Evidence strategy

Sample across banks, corporates and bond issuers; include amendment/correction and multi-security cases.

### Stopping / reopen

Stop when event taxonomy, duplicate/revision model and source contracts are empirically bounded.

---

## RT-04 — Рейтинговые действия и rating-rationale intelligence

### Problem

Рейтинг как строка — малоинформативен. Ценность находится в rationale: факторы, support assumptions, outlook, watch, triggers, metrics and qualitative constraints. Агентства публикуют это в HTML-релизах.

### Research question

Может ли newsflow формировать comparable rating-action records, сохраняя distinction между action, agency rationale, rating factors и downstream analytical inference?

### Scope

- АКРА;
- Эксперт РА;
- НКР;
- другие materially relevant agencies после source audit;
- issuer and issue ratings;
- upgrade/downgrade/affirmation/withdrawal;
- outlook / watch;
- positive/negative factors and triggers;
- release revisions.

### Non-goals

- собственный кредитный рейтинг;
- пересчет score;
- копирование full rating reports PDF;
- утверждение, что agency rationale является objective truth.

### Assumptions under test

- HTML releases достаточно богаты;
- common semantic frame возможен без потери agency-specific meaning;
- revision tracking materially useful.

### Source classes

- agency release indexes;
- individual HTML rating releases;
- methodology notices as context;
- issuer disclosure as corroboration.

### Alternatives / counterarguments

- достаточно хранить rating level history;
- rationale слишком agency-specific;
- paid data vendors already normalize actions.

### Failure modes

- issuer vs issue rating conflation;
- national vs international scale mixing;
- support factor treated as guarantee;
- revised release overwrites original wording;
- withdrawal reason misread as deterioration.

### Applicability and cost

Очень высокая для банков и кредитного анализа. Actions fit strong; semantic extraction strong; PDF dependency low for releases.

### Evidence strategy

Golden corpus from upgrade, downgrade, affirmation, watch, withdrawal and methodology-change cases.

### Stopping / reopen

Stop after proving cross-agency semantic common core and explicit agency-specific extensions.

---

## RT-05 — Непроцентные банковские тарифы, комиссии, лимиты и product-condition intelligence

### Problem

Текущий rate monitoring покрывает процентные ставки и FX, но конкурентная экономика банков также меняется через:

- комиссии;
- cashback / бонусные правила;
- лимиты переводов;
- стоимость обслуживания;
- premium subscriptions;
- acquiring;
- brokerage fees;
- SME packages;
- card/product eligibility;
- grace / free-service conditions.

Это часто живет в HTML-help/product pages и меняется без отдельного press release.

### Research question

Можно ли расширить competitive monitoring до generic bank-product terms, сохраняя semantic diff условий, а не пытаясь парсить все формальные PDF-тарифы?

### Scope

Retail + SME + brokerage products на публичных страницах крупнейших банков.

### Non-goals

- exhaustive tariff-book ingestion;
- PDF tariff parsing as baseline;
- персональные offers;
- calculation of customer-specific effective price.

### Assumptions under test

- material public terms достаточно представлены в HTML;
- semantic diff выявляет изменения лучше raw hash;
- one generic extraction contract can cover several product classes.

### Source classes

- product pages;
- help pages;
- tariff summaries;
- public calculators/endpoints;
- bank news only as secondary confirmation.

### Alternatives / counterarguments

- formal PDF tariff remains legally controlling;
- headline public page may omit exceptions;
- source heterogeneity much higher than rates.

### Failure modes

- promo vs permanent terms;
- old cached pages;
- conditional fee omitted;
- user segment mismatch;
- formal controlling document disagrees with web summary.

### Applicability and cost

Высокая для конкурентного банковского анализа. Source census cost medium/high; LLM gain high.

### Evidence strategy

Pilot on 5–6 banks and 4 product families with explicit comparison to controlling tariff links.

### Stopping / reopen

Stop when HTML coverage ceiling and representation hierarchy are measured.

---

## RT-06 — Экономика маркетплейсов и правила продавцов / партнеров

### Problem

Ozon, Wildberries, Yandex Market и другие платформы регулярно меняют:

- комиссии;
- логистические тарифы;
- storage / fulfillment;
- payout schedules;
- penalties;
- seller quality rules;
- subsidies / discounts;
- promotions;
- advertising mechanics;
- partner/PVZ economics.

Это materially влияет на продавцов, unit economics, конкуренцию банковских/финансовых экосистем и сектор электронной торговли.

### Research question

Может ли newsflow поддерживать исторически доказуемую карту изменений platform economics и partner rules из публичных help / seller pages?

### Scope

- Ozon;
- Wildberries;
- Yandex Market;
- позже Avito / delivery / service platforms;
- seller-facing and partner-facing public terms.

### Non-goals

- scraping individual product prices;
- seller-account private data;
- copying downloadable category tables when those rows лучше принадлежат Tabularium;
- legal interpretation of contracts.

### Assumptions under test

- help pages provide effective dates and revision markers often enough;
- semantic change detection can separate editorial updates from economic changes;
- cross-platform normalized concepts are possible.

### Source classes

- seller help;
- tariffs pages;
- public APIs/calculators;
- legal terms HTML;
- news pages.

### Alternatives / counterarguments

- category-level tables слишком большие;
- private cabinet may contain controlling rates;
- public pages can lag actual personalized terms.

### Failure modes

- temporary campaign mistaken for base tariff;
- model FBY/FBS/DBS conflation;
- seller category mismatch;
- page rewrite creates false mass change.

### Applicability and cost

Высокая для platform/e-commerce analysis. Actions fit strong for Yandex/WB examples; cross-platform source audit required.

### Evidence strategy

Construct same-concept comparison for commission, logistics, storage, payout and penalties.

### Stopping / reopen

Stop when common event semantics and platform-specific exceptions are established.

---

## RT-07 — Corporate operating guidance, capex и project-event intelligence

### Problem

Многие компании раскрывают важные изменения через press/IR HTML:

- production guidance;
- sales/GMV/traffic operational updates;
- capacity launch;
- plant/field/project start or delay;
- capex programme;
- new contract / partnership;
- asset shutdown;
- investment project financing;
- commissioning milestones.

Это не всегда формальный substantial fact и не сводится к statement monitoring.

### Research question

Можно ли построить sector-aware, но общим event contract, который превращает corporate press/IR flow в LLM-ready operational event timeline?

### Scope

- банки/финтех;
- нефтегаз;
- металлы;
- retail/e-commerce;
- developers;
- transport;
- selected strategic companies.

### Non-goals

- earnings statement table ingestion;
- automatic forecast;
- valuation;
- generic media monitoring.

### Assumptions under test

- operational event primitives повторяются между sectors;
- entity + asset/project + metric + horizon можно reliably extract;
- first-party press/IR coverage materially useful between reporting dates.

### Source classes

- IR press releases;
- corporate news;
- operational-results HTML;
- project pages;
- partner/government corroboration.

### Alternatives / counterarguments

- sector-specific schemas may be unavoidable;
- PR language creates selection bias;
- formal reporting provides more reliable data albeit slower.

### Failure modes

- target vs actual confused;
- project phase misclassified;
- company PR exaggeration treated as outcome;
- duplicate announcement across subsidiaries.

### Applicability and cost

Очень высокая для issuer and sector monitoring; medium source heterogeneity; strong downstream LLM value.

### Evidence strategy

Cross-sector pilot with same event taxonomy and explicit REPORTED_ACTUAL / GUIDANCE / PLAN distinction.

### Stopping / reopen

Stop when common core and required sector extensions are empirically known.

---

## RT-08 — Enforcement, supervisory, antitrust и consumer-protection actions

### Problem

Regulators publish actions that materially change risk view of firms:

- prescriptions;
- fines;
- warnings;
- antitrust cases;
- consumer-protection violations;
- license restrictions;
- illegal-market findings;
- compliance remediation status.

Raw lists may be structured, but event explanations and resolution lifecycle are text-heavy.

### Research question

Как отделить structured registry data от semantically rich enforcement events и построить defensible lifecycle: opened → ordered → appealed/remediated → closed?

### Scope

- Банк России;
- ФАС;
- other official regulators selectively;
- named financial institutions / platforms / large companies.

### Non-goals

- guilt assessment;
- legal advice;
- court-outcome prediction;
- full litigation system.

### Assumptions under test

- official web pages provide event history;
- entity binding possible;
- action lifecycle gives useful early-warning / governance signal.

### Source classes

- regulator enforcement HTML;
- case/decision indexes;
- warning/notice pages;
- structured lists only as supporting metadata.

### Alternatives / counterarguments

- many cases are low materiality;
- legal lifecycle can be incomplete online;
- court appeal may break source continuity.

### Failure modes

- allegation vs final finding conflated;
- paid fine vs resolved violation conflated;
- same bank name entity mismatch;
- duplicate regulator and court event.

### Applicability and cost

Высокая для bank risk and platform regulation. Source availability medium; specialist boundary high.

### Evidence strategy

Trace complete lifecycle for several enforcement event types.

### Stopping / reopen

Stop when action-state model, materiality filter and legal-language boundary are defined.

---

## RT-09 — Государственная поддержка, гранты, закупки и contract-award opportunity intelligence

### Problem

Для бизнеса существенны не все госновости, а события вида:

```text
открылась программа / закупка / конкурс
→ кто может участвовать
→ объем / лимит
→ предмет
→ регион / отрасль
→ deadline
→ условия
→ award / contract outcome
```

Raw tender registries относятся к structured data, но opportunity semantics и изменения условий могут требовать LLM.

### Research question

Где проходит полезная граница между Tabularium-style registry ingestion и newsflow semantic opportunity events для банковского/корпоративного/инвестиционного анализа?

### Scope

- grants / subsidies / concessional finance;
- SME support;
- procurement notices / awards where text semantics adds value;
- project concessions / infrastructure tenders;
- major corporate/state-company awards.

### Non-goals

- зеркало всего ЕИС;
- scoring заявки;
- automatic tender participation;
- PDF tender document extraction as baseline.

### Assumptions under test

- high-value opportunity subset can be selected deterministically;
- HTML/API metadata + short text sufficient for early event;
- semantic normalization produces useful company/sector demand signals.

### Source classes

- government programme pages;
- development institutions;
- procurement metadata;
- corporate/state-company tender pages;
- award announcements.

### Alternatives / counterarguments

- raw registry may be sufficient;
- access scale/terms may make GitHub Actions impractical;
- tender documents often PDF-heavy.

### Failure modes

- amendment changes deadline/amount;
- cancellation missed;
- framework procurement mistaken for actual spend;
- award amount vs maximum price conflated.

### Applicability and cost

Potentially very high, but source and volume risk high. Qualification retained because consumer job is distinct; substantive research must prove bounded source strategy.

### Evidence strategy

Pilot only selected high-value buyers/programmes, not whole-market crawl.

### Stopping / reopen

Stop when a bounded opportunity universe with acceptable PDF/access dependence is demonstrated or rejected.

---

## RT-10 — Strategic hiring и organizational capability signals

### Problem

Vacancies can reveal strategic direction before financial reporting:

- build-out of AI/data teams;
- risk/compliance hiring;
- regional expansion;
- new product unit;
- sales channel build-out;
- technology migration;
- mass hiring / hiring freeze signals.

Raw vacancy rows are structured data, but strategic meaning lives in descriptions, clusters and changes in capability mix.

### Research question

Можно ли законно и устойчиво использовать public career surfaces для company-level capability-change events без превращения newsflow в vacancy warehouse?

### Scope

- target banks and major corporates;
- first-party career sites preferred;
- public vacancy APIs only where terms permit analytical use;
- semantic role clusters, location, function, seniority, project hints.

### Non-goals

- resume/person monitoring;
- labor-market census;
- full HH dataset mirroring;
- employee-level inference.

### Assumptions under test

- first-party career surfaces sufficiently observable;
- cluster changes are meaningful after controlling reposts;
- LLM classification adds value.

### Source classes

- corporate career pages;
- ATS APIs;
- selected vacancy APIs subject to permitted use;
- company news for corroboration.

### Alternatives / counterarguments

- vacancies are noisy and tactical;
- third-party API terms may prohibit intended analytics;
- duplicate/reposted roles distort trend.

### Failure modes

- one recruiter campaign creates false strategic signal;
- closed vacancy interpreted as cancellation;
- remote geography misbound;
- employer subsidiaries double-counted.

### Applicability and cost

Medium/high research value. Technical feasibility medium; source/licensing risk material.

### Evidence strategy

Start with first-party career portals and compare against one permitted API source.

### Stopping / reopen

Stop when signal-to-noise and permitted source contract are established.

---

## RT-11 — Digital product, API и feature-release intelligence

### Problem

Banks, fintechs, marketplaces and technology companies frequently change products through web announcements and documentation:

- new payment function;
- digital ruble support;
- API capability;
- merchant tools;
- AI assistant;
- mobile feature;
- product launch/retirement;
- integration;
- availability by customer segment.

Это стратегически важно и обычно не отражается в отчетности как отдельный structured dataset.

### Research question

Можно ли формировать reproducible product-capability timeline по target companies, связывая release/announcement/documentation change с конкретной customer capability?

### Scope

- banking digital products;
- fintech/payment products;
- marketplace seller/buyer tools;
- APIs and developer docs where business-relevant;
- public product retirements and migrations.

### Non-goals

- software dependency monitoring in general;
- app binary reverse engineering;
- feature-quality assessment;
- user-review sentiment.

### Assumptions under test

- release semantics recoverable from public HTML;
- capability taxonomy can be compact;
- docs diff can expose material rollout even without press release.

### Source classes

- product news;
- help center;
- developer docs;
- release notes;
- public app descriptions as optional source.

### Alternatives / counterarguments

- many rollouts are gradual/A-B tested;
- marketing announcement may precede actual availability;
- docs change may be editorial.

### Failure modes

- announced != available;
- pilot != full rollout;
- segment-specific availability lost;
- rename treated as new feature.

### Applicability and cost

Высокая для digital banking / ecosystem competitive intelligence.

### Evidence strategy

Track one capability across 5 banks/platforms from announcement to general availability.

### Stopping / reopen

Stop when availability-state semantics and source precedence are defined.

---

## RT-12 — Operational resilience: service incidents, outages и cyber events

### Problem

Для digital banks, exchanges, cloud providers, payment systems and platforms outages materially affect operational risk and customer experience. Structured status APIs exist in some ecosystems, while others publish incident notices or postmortems in HTML.

### Research question

Можно ли поддерживать cross-provider incident timeline with start/impact/components/resolution/postmortem semantics и отделять first-party confirmed incident от secondary outage reports?

### Scope

- banking/payment infrastructure where official sources exist;
- exchanges;
- cloud/platform dependencies;
- statuspage-compatible services;
- official security incidents and postmortems.

### Non-goals

- generic cyber threat intelligence;
- exploit instructions;
- social-media outage rumor aggregation;
- SLA adjudication.

### Assumptions under test

- enough target entities expose official status/incident surfaces;
- incident lifecycle can be normalized;
- dependency relation materially improves bank/platform risk analysis.

### Source classes

- status API;
- status page RSS/JSON;
- official incident notices;
- postmortems;
- security advisories.

### Alternatives / counterarguments

- Russian financial institutions may lack public status pages;
- secondary outage aggregators have better coverage but weaker provenance;
- rare events may not justify separate pipeline.

### Failure modes

- maintenance vs incident;
- partial vs full outage;
- upstream dependency causes duplicate incidents;
- resolved status before full customer recovery.

### Applicability and cost

Potentially high, but source coverage uncertainty material. This uncertainty itself justifies bounded research.

### Evidence strategy

Census 20–30 target entities and providers, then reconstruct several incident lifecycles.

### Stopping / reopen

Stop if first-party coverage is too sparse; route secondary monitoring only after explicit source-quality decision.

# 7. Relations and fan-in

```text
RT-01 regulation
 ├─> RT-08 enforcement
 └─> RT-09 support/opportunity

RT-02 sanctions/trade
 └─> can affect RT-07 corporate operations

RT-03 issuer events
 ├─> formal side of RT-07 operating events
 └─> corroborates RT-04 ratings

RT-05 bank terms
 └─> parallel to RT-06 platform economics

RT-07 corporate operations
 ├─> can consume RT-10 hiring as leading signal
 └─> can consume RT-11 product releases

RT-12 incidents
 └─> operational-risk evidence for RT-07 / RT-11
```

Это relations, а не stages. Один Topic не должен поглощать другой только из-за common source.

# 8. External feasibility anchors checked during qualification

Qualification did not perform full substantive research, но проверила, что несколько ключевых source families реально существуют в web-friendly форме.

## Regulation

- `regulation.gov.ru` exposes project pages and visibly advertises Open Data, email subscription and RSS subscription.
- Bank of Russia publishes HTML press/regulatory/supervisory pages.

## Enforcement

- Bank of Russia publishes a dedicated HTML stream of prescriptions to credit institutions for consumer-protection violations.
- Bank of Russia also exposes its warning list as JSON/XML/API; this is an example where raw registry belongs to structured-data handling while semantic action monitoring is a separate job.

## Ratings

- ACRA maintains a large HTML rating-release index with sector/date/type filtering.
- Expert RA publishes rating actions as detailed HTML releases.
- NCR publishes a press-release index with rating actions.

## Corporate disclosures

- e-disclosure and PRIME disclosure pages expose substantial-fact messages in HTML.
- NSDdata publishes corporate-action/news items in an index/detail model.

## Marketplace economics

- Yandex Market public seller help contains current tariffs, payout rules and effective-date changes.
- Wildberries seller portal publishes tariff help/news pages with updated dates.

## Sanctions

- UK Sanctions List is available in HTML, XML, CSV, TXT and other formats, with explicit update history.
- Official sanctions guidance/notices are also available as HTML.

## Hiring

- HeadHunter exposes vacancy search/details through API/OpenAPI, but its terms materially constrain permitted use. Therefore RT-10 must prefer first-party career surfaces or separately verify permitted analytical use.

## Incidents

- Statuspage-based services expose machine-readable incident endpoints with incident lifecycle states. Coverage of the actual Russian bank/platform target universe remains UNKNOWN and must be measured.

# 9. Value / cost signals without priority

| Topic | Decision value | Actions fit | LLM semantic gain | PDF dependence risk | Access / rights risk | Expected source heterogeneity |
| --- | --- | --- | --- | --- | --- | --- |
| RT-01 Regulation | High | High | High | Medium | Low | High |
| RT-02 Sanctions/trade | High | High | High | Low/Medium | Low | Medium |
| RT-03 Issuer events | High | High | High | Medium | Medium | Medium |
| RT-04 Ratings | High | High | High | Low | Low | Low/Medium |
| RT-05 Bank terms | High | Medium/High | High | Medium | Low | High |
| RT-06 Marketplace economics | High | High | High | Low/Medium | Low/Medium | Medium |
| RT-07 Corporate operations | High | High | High | Medium | Low | High |
| RT-08 Enforcement | High | High | High | Medium | Low | Medium |
| RT-09 Opportunities/procurement | High | Medium | High | High | Medium | High |
| RT-10 Hiring | Medium/High | Medium | High | Low | High | High |
| RT-11 Product releases | High | High | High | Low | Low | High |
| RT-12 Incidents | Medium/High | High where source exists | Medium/High | Low | Low | Medium |

Это не ranking и не Agenda admission.

# 10. Strongest cross-topic conclusion

Наиболее крупный незакрытый слой `newsflow` — не «еще больше новостей», а **event intelligence for changing terms, obligations, formal actions and operating states**.

Текущий corpus уже хорошо охватывает:

```text
price-like signals:
rates, FX

physical signals:
commodity flow, weather

speech signals:
public statements
```

Главный remaining frontier:

```text
RULES
FORMAL EVENTS
RISK ASSESSMENTS
BUSINESS TERMS
CORPORATE OPERATIONS
OPPORTUNITIES
CAPABILITY CHANGES
INCIDENTS
```

Именно эти классы особенно хорошо соответствуют `newsflow`, потому что:

1. источник часто HTML-first;
2. source changes recurring;
3. raw table ingestion недостаточен;
4. event detection можно делать детерминированно;
5. LLM нужна для semantic extraction, а не для web discovery;
6. результат естественно хранить как compact provenance-aware event records;
7. такой event corpus дает downstream LLM гораздо более сильный вход, чем набор сырых ссылок.

# 11. Suggested next commissioning choices

MADARAII-07 не устанавливает priority. Для последующего Commission целесообразно выбирать topic, где одновременно есть высокий practical payoff и быстрый empirical source test.

Самые очевидные кандидаты для первого substantive research contour:

- RT-04 Ratings;
- RT-03 Issuer events;
- RT-01 Regulation;
- RT-05 Bank non-rate terms;
- RT-06 Marketplace economics;
- RT-07 Corporate operating events.

RT-09, RT-10 и RT-12 требуют более жесткого source/access census до product decision.

# 12. Verification against MADARAII-07

- Current `newsflow` Research corpus was inspected before declaring gaps.
- The candidate universe was generated from domain × event × source morphology rather than free-form ideation only.
- Existing themes were explicitly removed from the gap set.
- Ready tables / series were routed toward Tabularium where semantic event processing adds little value.
- Design questions (schema, provider, cadence) were not misclassified as Research Topics.
- Every material candidate has a disposition.
- Qualified topics are application-specific and independently executable.
- Each brief includes problem, question, scope, non-goals, assumptions, source classes, alternatives/counterarguments, failure modes, applicability/cost, evidence strategy and stop/reopen.
- Priority, Agenda admission and Commission remain separate.
- Specialist legal/compliance boundaries remain explicit.
- PDF is treated as optional/fallback evidence, not a required production carrier.

# 13. Reopen triggers

Revisit this qualification if:

- `newsflow` establishes a maintained Product owner that materially narrows its mission;
- a qualified Topic is researched and found to collapse into an existing contour;
- a major public source family becomes PDF-only or auth-only;
- GitHub Actions access constraints change materially;
- a new high-value consumer job appears that is absent from this candidate universe;
- Tabularium expands its role from source-faithful tables into semantic event ownership, changing the boundary.

# 14. External source anchors

- Federal portal for draft regulations: https://regulation.gov.ru/
- Bank of Russia: https://www.cbr.ru/
- ACRA rating releases: https://www.acra-ratings.ru/press-releases/
- Expert RA: https://raexpert.ru/
- NCR rating releases: https://www.ratings.ru/ratings/press-releases/
- Interfax disclosure: https://www.e-disclosure.ru/
- PRIME disclosure: https://disclosure.1prime.ru/
- NSDdata corporate actions/news: https://nsddata.ru/ru/news
- Yandex Market seller help: https://yandex.ru/support/marketplace/ru/
- Wildberries seller help: https://seller.wildberries.ru/instructions/ru/ru/
- UK Sanctions List: https://www.gov.uk/government/publications/the-uk-sanctions-list
- HeadHunter API docs: https://api.hh.ru/openapi/redoc
- Statuspage-compatible incident API example pattern: /api/v2/incidents.json

## Completion statement

The commissioned MADARAII-07 work is complete at qualification level: the current application-space has been systematically widened, existing coverage and non-Research routes separated, and twelve concrete application Research Topics established. No substantive research, Agenda admission, priority decision or implementation has been silently commissioned.
