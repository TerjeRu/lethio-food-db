# lethio-food-db

The Open Food Facts derived database used by **Lethio Macro Tracker**, published under the Open
Database License (ODbL) 1.0 because the ODbL asks for it.

**Download: [Releases](https://github.com/TerjeRu/lethio-food-db/releases)**

| File | Size | What |
|---|---|---|
| `lethio-off-derived-build16.sqlite` | 90 MB | SQLite, the schema the app itself uses |
| `lethio-off-derived-build16.csv.gz` | 19 MB | One row per food, gzipped, for everyone else |

376,048 branded food products, with per-100 g energy, protein, fat, carbohydrate, fibre, sugar,
saturated fat and sodium, plus barcodes, serving sizes, brand and country.

## Licence, and exactly what it covers

The data in this repository is derived from [Open Food Facts](https://world.openfoodfacts.org/data)
and is made available under the **Open Database License (ODbL) 1.0**. The full text is in
[`LICENSE`](LICENSE).

> **The ODbL applies exclusively to the portion of the Lethio Macro Tracker database derived from
> Open Food Facts, which is the entirety of what this repository publishes.**

Attribution, which the licence requires you to keep if you reuse this:

```
Contains information from Open Food Facts (https://world.openfoodfacts.org),
which is made available under the Open Database License (ODbL) 1.0.
© Open Food Facts contributors.
```

### What is deliberately not here

The app also bundles thirteen national food composition tables. **None of them are in this
repository**, and that is a licence decision rather than an oversight.

Those tables are free to *use* with credit, which is not the same as free to *relicense*. BEDCA's
terms are the clearest case: non-commercial, attribution, and no modification. This pipeline
modifies every row it touches, through name cleaning, Unicode folding, unit normalisation and a
120-character name cap. Republishing those rows under a licence that grants commercial reuse would
hand out permissions their publishers never gave.

So the ODbL obligation is discharged over the Open Food Facts data, which is what the ODbL covers,
and the national tables stay where their publishers put them. Each one is credited inside the app,
with its own licence named, on the Data sources screen.

## Provenance

| | |
|---|---|
| Upstream | `en.openfoodfacts.org.products.csv.gz`, downloaded 2026-08-17 |
| Built | 2026-09-06, database build 16 |
| Rows upstream | 4,532,767 |
| Rows here | 376,048 |

Open Food Facts republishes daily, so the upstream file is a moving target. The date above is the
one that matters for reproducing this exact export.

## What was changed, and why that is the point

This is a *derived* database, not a copy. Share-alike exists so the derivation comes back out, so
the changes travel with it:

- **`name`** is repaired rather than rewritten: double-encoded UTF-8 undone (`CÃ´te` back to
  `Côte`), HTML entities and control characters removed, whitespace collapsed. A name is rejected
  outright if it falls outside 2 to 120 characters, contains a URL, or contains no letters at all,
  the last being a barcode or a weight that ended up in the name field
- **`name_folded`** is a search key with diacritics folded, including `ø`→`o`, `ł`→`l`, `ð`→`d`
- **`carbs_convention`** records whether the publisher's carbohydrate figure is by difference or
  available carbohydrate, so the number is interpretable rather than merely present
- **`quality_score`**, **`basic_rank`** and **`source_rank`** are ours: they order search results
  and mean nothing outside the app, but they are included rather than stripped
- Rows with no energy value, or outside 0 to 900 kcal per 100 g, were dropped
- Barcodes are validated against the GTIN check digit, not just length. About 3.7% of
  otherwise well-formed codes upstream fail it; those are internal or mistyped codes that no
  scanner will ever produce
- Duplicate listings of the same product were collapsed onto the best-documented row, and **every
  dropped row's barcode moved onto the survivor**, which is why there are 514,062 barcodes over
  376,048 rows

Rebuilding is `tools/build_nutrition_db.py` followed by `tools/export_odbl_subset.py`. Both are
here. The build script needs the raw sources, which are not redistributed; it names each one and
where to get it.

## The app

[Lethio Macro Tracker](https://macros.lethio.com) is a calorie and macro diary for Android, built
on national food composition tables with Open Food Facts covering branded products. The whole
database ships inside the app, so search works offline.

It also contributes back: when a scan finds a barcode Open Food Facts does not have, the app can
send the product a reader types in. That is off by default and lives behind its own switch.

## Contact

`support@lethio.com`
