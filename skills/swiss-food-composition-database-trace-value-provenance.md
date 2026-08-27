---
name: swiss-food-composition-trace-value-provenance
description: >-
  Explain where a single nutrient number in the Swiss Food Composition Database came from — its citation,
  analytical method, borrowed/rescaled status, and the LanguaL/FoodEx2 description of the food it belongs
  to. Use when the trustworthiness or origin of a value matters.
api: Swiss Food Composition Database API
base_url: https://api.webapp.prod.blv.foodcase-services.com/BLV_WebApp_WS
auth: none
operations:
  - getFood
  - getValues
  - getValue
  - getValue_1
  - getLangualCodes
  - getIngredients
generated: '2026-08-27'
method: generated
source: >-
  Grounded in the provider's served OpenAPI 3.0.1
  (openapi/swiss-food-composition-database-openapi.json); the example provenance payloads were probed live
  on 2026-08-27.
---

# Trace where a nutrient value came from

This database documents provenance per value, which is unusual and worth using.

## 1. Get the value and its id

    GET /webresources/BLV-api/food/351915?lang=en   (getFood)

or, for one component set,

    GET /webresources/BLV-api/values?DBID=351915&componentsetid=2&lang=en   (getValues)

Each entry in `values[]` has its own `id`. Live example for "Apple, fresh" calcium:

    {"id":6370553,"value":5.3,"rawValue":5.34,
     "references":[{"id":2277,"citation":"Anses - Table de composition nutritionnelle des aliments Ciqual 2020",
                    "typeDescriptor":"Webpage"}],
     "component":{"name":"Calcium (Ca)","code":"CA"},"unit":{"name":"milligram","code":"mg"},
     "minimum":5.3,"maximum":5.3,"n":1,"methodIndicatorCode":"MIR003",
     "isborrowed":false,"isrescaled":false,"isfixed":false}

## 2. Read the source record

    GET /webresources/BLV-api/value/6370553?lang=en   (getValue)      -> formatted value, derivationOfValue, sourceInfoLevel
    GET /webresources/BLV-api/source/6370553          (getValue_1)    -> SourceValueDTO

`SourceValueDTO` carries `acquistionTypeDescriptor` (note the provider's spelling),
`methodIndicatorDescriptor`, `references[]`, `updated`, `minimum`/`maximum`/`n`,
`contributingvalues`, and the `isborrowed` / `isrescaled` / `isfixed` flags. `getValue_1` takes no `lang`.

## 3. Describe the food itself in standard terms

    GET /webresources/BLV-api/langualcodes?DBID=351915   (getLangualCodes)

Returns LanguaL facet rows, e.g.
`{"letter":"A","description":"PRODUCT TYPE","classification":"FRUIT OR FRUIT PRODUCT (EUROFIR) (A0833), FRUIT OR FRUIT PRODUCT (US CFR) (A0143)"}`
and EFSA FoodEx2 terms such as `"020 - CEREALS AND CEREAL PRIMARY DERIVATIVES (EFSA FOODEX2)"`. Use these
to align a Swiss food with an EU or EuroFIR classification instead of matching on name.

If the food is a recipe (`isrecipe: true`), its composition is calculated from parts:

    GET /webresources/BLV-api/ingredients?DBID={dbid}&lang=en   (getIngredients)

## Rules

- Say plainly when a value is **borrowed** (`isborrowed: true`) — it was taken from another food or another
  database rather than analysed for this one. `n` tells you how many contributing measurements there were;
  `n: 1` is a single source.
- Quote the `citation` string verbatim. Many values come from foreign tables (Ciqual, Souci-Fachmann-Kraut)
  and that is material to how a user should read them.
- `references[]` can be empty. If it is, say the value has no published citation rather than inventing one.
- `getValue_1` returns no language parameter and no localisation; do not translate descriptor codes.
- Never assert a value's method or window beyond what these fields return.
