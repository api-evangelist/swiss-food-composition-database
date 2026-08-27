---
name: swiss-food-composition-look-up-a-food
description: >-
  Find a food in the Swiss Food Composition Database by name and read its full nutrient profile, in German,
  French, Italian or English. Use when asked for the composition of a food available in Switzerland.
api: Swiss Food Composition Database API
base_url: https://api.webapp.prod.blv.foodcase-services.com/BLV_WebApp_WS
auth: none
operations:
  - getFoodsTurbo
  - getFood
  - getComponentSets
  - getValues
  - getVersiondb
generated: '2026-08-27'
method: generated
source: >-
  Grounded in the provider's served OpenAPI 3.0.1
  (openapi/swiss-food-composition-database-openapi.json). Every operationId, path and parameter below is
  taken from that document; the example responses were probed live on 2026-08-27.
---

# Look up a food and read its nutrients

No credentials are needed. Every call is an anonymous `GET`, and every response is JSON.

## 1. Search for the food

    GET /webresources/BLV-api/foods?search=apple&lang=en&limit=10   (getFoodsTurbo)

`lang` is `de`, `en`, `fr` or `it` and defaults to `en`. `limit` and `offset` are the only pagination
controls, and no total count is returned — if you get exactly `limit` rows, assume there may be more.

A live result row looks like:

    {"id":351915,"foodName":"Apple, fresh","generic":true,
     "categoryNames":"Fresh fruit","amount":0.0,"foodid":378,"valueTypeCode":""}

`id` here is the **DBID**. Note that the served schema for this operation
(`FoodWithNamesSynonymesCategories`) declares `names[]`, `synonyms[]` and `categories[]` and the live
response does not return them — read the fields above, not the schema.

If the user gave you a public food ID (the number printed in the Excel export or the search app), it is a
`foodid`, not a DBID. Translate it first:

    GET /webresources/BLV-api/fooddbid/378   (getFoodDbIdByFoodId)  ->  351915

That operation returns a bare integer, despite the spec declaring an array.

## 2. Read the whole food

    GET /webresources/BLV-api/food/351915?lang=en   (getFood)

Returns the food with `categories[]`, `synonyms[]` and a `values[]` array. Each value carries `value`,
`rawValue`, its `component` (name plus the EuroFIR/INFOODS `code`, e.g. `CA` for calcium), its `unit`,
`minimum`/`maximum`/`n`, and `references[]` with the actual citation the number came from. Flags
`isborrowed`, `isrescaled` and `isfixed` tell you whether the value was measured for this food or carried
over — say so when the answer matters.

## 3. Or read one nutrient bundle at a time

    GET /webresources/BLV-api/sets?lang=en      (getComponentSets)
    GET /webresources/BLV-api/values?DBID=351915&componentsetid=1&lang=en   (getValues)

`componentsetid` is required. The live sets are `1` Standard, `2` All nutrients, `3` Food labelling. Use
`3` when the user is asking about a nutrition declaration.

## Rules

- All values refer to **100 g edible portion** unless stated otherwise. Never present a value as a portion
  figure without converting and saying so.
- Always cite the database version. `GET /webresources/BLV-api/versiondb` (getVersiondb) returns
  `{"idversion":51,"versiontext":"V 7.1"}`. Food IDs and category IDs are not guaranteed stable across
  versions — V 6.0 removed branded products wholesale and V 7.0 renamed a whole category.
- The FSVO requires acknowledgement of the source when the data is used.
- Errors are bare status codes with an empty body. A `404` on `/food/{DBID}` almost always means the
  identifier space was wrong — go back to `getFoodDbIdByFoodId`.
- An invalid `lang` value returns `200` with data, not an error. Validate the language code yourself.
- Never call `GET /webresources/BLV-api/reloadCache`. It reloads the server's cache and is not a read.
