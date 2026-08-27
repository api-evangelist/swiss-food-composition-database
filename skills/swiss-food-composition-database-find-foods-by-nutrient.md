---
name: swiss-food-composition-find-foods-by-nutrient
description: >-
  Find Swiss foods above or below a nutrient threshold — "which foods have more than 10 mg of iron per
  100 g" — using the component/operator/amount filter, optionally scoped to a food category.
api: Swiss Food Composition Database API
base_url: https://api.webapp.prod.blv.foodcase-services.com/BLV_WebApp_WS
auth: none
operations:
  - getComponents
  - getCategories
  - getSubCategories
  - getFoodsTurbo
  - getCategorizedFoods
  - getUnits
generated: '2026-08-27'
method: generated
source: >-
  Grounded in the provider's served OpenAPI 3.0.1
  (openapi/swiss-food-composition-database-openapi.json); parameter names, the operator enum and the
  component-set values were probed live on 2026-08-27.
---

# Find foods by nutrient threshold

## 1. Resolve the nutrient to a component ID

    GET /webresources/BLV-api/components?lang=en   (getComponents)

Returns every nutrient with `id`, localised `name`, a EuroFIR/INFOODS `code`, its `group`, its `unit` id and
the component sets it belongs to. Live examples: `{"name":"Calcium (Ca)","id":34,"code":"CA",...}`,
`{"name":"Carbohydrates, available","id":55,"code":"CHO",...}`. Match on `code` when the user names a
standard nutrient, on `name` when they use everyday language.

Resolve the unit id with `GET /webresources/BLV-api/units?lang=en` (getUnits) so you can state the threshold
in the unit the database actually uses. Getting this wrong by a factor of 1000 (mg versus µg) is the most
likely failure of this skill.

## 2. Optionally scope to a category

    GET /webresources/BLV-api/topcategories?lang=en        (getCategories)
    GET /webresources/BLV-api/subcategories/6754?lang=en   (getSubCategories)

Live top-level categories include `{"name":"Vegetables","id":6777}` and `{"name":"Fruit","id":6776}`.

## 3. Run the threshold query

    GET /webresources/BLV-api/foods?component=34&operator=>&amount=100&lang=en&limit=50   (getFoodsTurbo)

Parameters, all from the spec:

- `component` — the nutrient id from step 1.
- `operator` — one of `<`, `>`, `=`. URL-encode it (`%3E`, `%3C`).
- `amount` — an integer threshold in the component's own unit.
- `category` — one top-level category id; `subcategory` — repeatable subcategory ids.
- `type=true` — restrict to generic foods (the 1,246 non-branded entries).
- `search` — combine with a name fragment.
- `limit` / `offset` — the only paging controls.

For a faceted overview rather than a flat list:

    GET /webresources/BLV-api/categorizedfoods?component=34&operator=>&amount=100&lang=en   (getCategorizedFoods)

returns `{categoryId, categoryName, numberOfFoods}` rows. Its `offset` parameter is documented as unused —
do not rely on it.

## Rules

- `amount` is an integer in the component's own unit, compared against the per-100 g value. State both the
  unit and the 100 g basis in any answer.
- The API applies no validation you can trust: an unknown `lang` returns `200`. Check your own inputs.
- Results are a bare JSON array with no total. Treat a full page as "possibly more".
- Report the database version from `getVersiondb` alongside any list, and acknowledge the FSVO as source.
- Use `/webresources/BLV-api/foods`, never `/foods_old` (getFoods), which is the superseded search operation.
