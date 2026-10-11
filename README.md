# lethio-food-db

Branded food products derived from [Open Food Facts](https://world.openfoodfacts.org/), published
under the Open Database License (ODbL) 1.0.

**[Download from Releases](https://github.com/TerjeRu/lethio-food-db/releases)**

| File | Size | Format |
|---|---|---|
| `lethio-off-derived-build19.sqlite` | 112 MB | SQLite 3 |
| `lethio-off-derived-build19.csv.gz` | 22 MB | Gzipped CSV, one row per food |

520,432 products with per-100 g nutrition, 520,432 barcodes, 156,487 serving sizes, across 187
country codes.

## Licence

The data is made available under the [Open Database License (ODbL) 1.0](LICENSE).
Individual contents retain the [Database Contents License](https://opendatacommons.org/licenses/dbcl/1-0/).
The export script is [MIT licensed](LICENSE-CODE).

The ODbL applies exclusively to the portion of the Lethio Macro Tracker database derived from Open
Food Facts, which is the entirety of what this repository publishes.

Required attribution for reuse:

```
Contains information from Open Food Facts (https://world.openfoodfacts.org),
which is made available under the Open Database License (ODbL) 1.0.
© Open Food Facts contributors.
```

## Scope

This export contains Open Food Facts records (`source = 'off'`) and their barcodes and servings.
The app's national food tables have separate terms, listed in
[nutrition data licenses](https://github.com/TerjeRu/lethio-macro-tracker/blob/main/LICENSES/Nutrition-data.txt).

## Provenance

| | |
|---|---|
| Upstream | `en.openfoodfacts.org.products.csv.gz` |
| Downloaded | 2026-08-17 |
| Built | 2026-10-08 |
| Build version | 19 |
| Upstream records | 4,532,767 |
| Records here | 520,432 |

Open Food Facts republishes daily. The download date above identifies the input.

## Schema

### `foods`

| Column | Type | Notes |
|---|---|---|
| `id` | INTEGER | Primary key, assigned by the build |
| `name` | TEXT | Product name, normalised (see Transformations) |
| `name_folded` | TEXT | Search key, diacritics folded |
| `lang` | TEXT | Always NULL in this subset |
| `brand` | TEXT | Brand, where supplied |
| `barcode` | TEXT | Product's own GTIN; also indexed in `food_barcodes` |
| `kcal_100g` | REAL | 0–900, required |
| `protein_100g` | REAL | Grams, required |
| `fat_100g` | REAL | Grams, required |
| `carbs_100g` | REAL | Grams, required |
| `carbs_convention` | TEXT | Always `available` in this subset |
| `fiber_100g` | REAL | Grams, where supplied |
| `sugar_100g` | REAL | Grams, where supplied |
| `sat_fat_100g` | REAL | Grams, where supplied |
| `sodium_100g` | REAL | Grams, where supplied |
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

`barcode TEXT`, `food_id INTEGER`. 520,432 rows. Each barcode resolves to its own product record,
preserving that product's nutrition and serving sizes.

### `food_servings`

`id`, `food_id`, `label`, `grams`, `is_default`, `sort_order`. 156,487 rows, at most one per food.

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
- Barcode validation: GTIN-8/12/13/14 check digit verified.
- Duplicate records for one barcode retain the most complete record. Different barcodes keep
  their own nutrition and serving sizes.
- Micronutrient values failing plausibility bounds are dropped rather than stored.

## Reproducing

The [app source repository](https://github.com/TerjeRu/lethio-macro-tracker) contains the database
builder, its helper modules, and the Room schema. Follow its README to obtain the upstream inputs
and build a catalogue.

`tools/export_odbl_subset.py` produces the release artifacts:

```bash
python tools/export_odbl_subset.py --seed seed.db --out out --build build19
```

## Related

[Lethio Macro Tracker](https://macros.lethio.com), the Android application this database was built
for. [Get the app on Google Play](https://play.google.com/store/apps/details?id=com.lethio.macros)
or browse the [app source](https://github.com/TerjeRu/lethio-macro-tracker).

## Contact

support@lethio.com
