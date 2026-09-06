# lethio-food-db

Branded food products derived from [Open Food Facts](https://world.openfoodfacts.org/), published
under the Open Database License (ODbL) 1.0.

**[Download from Releases](https://github.com/TerjeRu/lethio-food-db/releases)**

| File | Size | Format |
|---|---|---|
| `lethio-off-derived-build16.sqlite` | 90 MB | SQLite 3 |
| `lethio-off-derived-build16.csv.gz` | 19 MB | Gzipped CSV, one row per food |

376,048 products with per-100 g nutrition, 514,062 barcodes, 136,118 serving sizes, across 180
country codes.

## Licence

The data is made available under the [Open Database License (ODbL) 1.0](LICENSE).

The ODbL applies exclusively to the portion of the Lethio Macro Tracker database derived from Open
Food Facts, which is the entirety of what this repository publishes.

Required attribution for reuse:

```
Contains information from Open Food Facts (https://world.openfoodfacts.org),
which is made available under the Open Database License (ODbL) 1.0.
© Open Food Facts contributors.
```

## Scope

Contains: Open Food Facts records only, `source = 'off'`.

Does not contain: the thirteen national food composition tables that Lethio Macro Tracker also
bundles (BEDCA, CoFID, Ciqual, BLS, Matvaretabellen, Livsmedelsdatabasen, Fineli, Frida, Swiss
FCDB, TCA, CNF, AFCD, USDA FoodData Central). Their licences permit use with attribution but do not
grant redistribution under ODbL. BEDCA additionally prohibits modification. Obtain those from their
publishers.

## Provenance

| | |
|---|---|
| Upstream | `en.openfoodfacts.org.products.csv.gz` |
| Downloaded | 2026-08-17 |
| Built | 2026-09-06 |
| Build version | 16 |
| Upstream records | 4,532,767 |
| Records here | 376,048 |

Open Food Facts republishes daily. The download date above identifies the input.

## Schema

### `foods`

| Column | Type | Notes |
|---|---|---|
| `id` | INTEGER | Primary key, assigned by the build |
| `name` | TEXT | Product name, normalised (see Transformations) |
| `name_folded` | TEXT | Search key, diacritics folded |
| `lang` | TEXT | Always NULL in this subset |
| `brand` | TEXT | 91% populated |
| `barcode` | TEXT | Display value; the full mapping is in `food_barcodes` |
| `kcal_100g` | REAL | 0–900, required |
| `protein_100g` | REAL | Grams, required |
| `fat_100g` | REAL | Grams, required |
| `carbs_100g` | REAL | Grams, required |
| `carbs_convention` | TEXT | Always `available` in this subset |
| `fiber_100g` | REAL | Grams, 65% populated |
| `sugar_100g` | REAL | Grams, 98% populated |
| `sat_fat_100g` | REAL | Grams, 97% populated |
| `sodium_100g` | REAL | Grams, 95% populated |
| `density_g_per_ml` | REAL | Always NULL in this subset |
| `source` | TEXT | Always `off` |
| `quality_score` | INTEGER | Application-specific completeness score |
| `basic_rank` | INTEGER | Application-specific search ordering |
| `source_rank` | INTEGER | Application-specific search ordering |
| `alias_names` | TEXT | Always NULL in this subset |
| `search_extra` | TEXT | Always NULL in this subset |
| `country` | TEXT | ISO 3166-1 alpha-2, from the product's markets |

The three `*_rank` and `quality_score` columns order results inside the app and carry no meaning
outside it. They are included rather than stripped.

### `food_barcodes`

`barcode TEXT`, `food_id INTEGER`. 514,062 rows. More barcodes than foods because duplicate
listings were merged and their barcodes retained against the surviving record.

### `food_servings`

`id`, `food_id`, `label`, `grams`, `is_default`, `sort_order`. 136,118 rows, at most one per food.

### `food_aliases`

Present in the schema, empty in this subset.

### CSV

The CSV carries the `foods` columns plus a trailing `barcodes` column, semicolon-separated, so the
one-to-many barcode mapping survives flattening. Serving sizes are in the SQLite file only.

## Transformations applied to the upstream data

- Name repair: double-encoded UTF-8 corrected, HTML entities and control characters removed,
  whitespace collapsed.
- Name rejection: names outside 2–120 characters, names containing a URL, and names containing no
  letters are dropped.
- Energy filter: records with no energy value, or outside 0–900 kcal per 100 g, are dropped.
- Barcode validation: GTIN-8/12/13/14 check digit verified. Approximately 3.7% of otherwise
  well-formed upstream codes fail this and are dropped.
- Deduplication: repeated listings of the same product are merged onto the most complete record.
  Barcodes from merged records are retained.
- Micronutrient values failing plausibility bounds are dropped rather than stored.

## Reproducing

`tools/build_nutrition_db.py` builds the full database from the raw sources; it documents each
source and where to obtain it. Raw sources are not redistributed here.

`tools/export_odbl_subset.py` produces the release artifacts:

```bash
python tools/export_odbl_subset.py --seed seed.db --out out --build build16
```

## Related

[Lethio Macro Tracker](https://macros.lethio.com), the Android application this database was built
for.

## Contact

support@lethio.com
