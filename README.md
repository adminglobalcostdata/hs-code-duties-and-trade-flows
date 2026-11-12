# HS Code Duties and Trade Flows - US, EU, Canada, Australia

[![Apify Actor](https://img.shields.io/badge/Apify-Actor-1f6feb)](https://apify.com/phonic_lily/hs-code-duties-and-trade-flows) ![Licence](https://img.shields.io/badge/licence-Mixed_per_source_Eurostat_Comext_reuse_a-blue) ![Records](https://img.shields.io/badge/records-5%2C606-green)

One record per 6-digit HS code: base import duty rate in the US, EU, Canada and Australia, crossed with real EU27 trade flows (value, weight, top 3 partners). Pay per record.

## What you get
- **5,606 records**, each carrying its own `license` and `attribution` fields, ready to republish with the required credit.
- A **100-record free sample** in this repository: [`sample.jsonl`](sample.jsonl) and [`sample.csv`](sample.csv) (opens as a table on GitHub).
- The **full dataset** as a filterable API (JSON, CSV, Excel), no scraping, no login: [HS Code Duties and Trade Flows - US, EU, Canada, Australia on Apify](https://apify.com/phonic_lily/hs-code-duties-and-trade-flows).

## One record
```json
{
  "hs6": "010121",
  "hs2": "01",
  "hs4": "0101",
  "description": "Pure-bred breeding horses",
  "us_base_rate_pct": 0.0,
  "us_rate_published": "Free",
  "eu_base_rate_pct": 0.0,
  "eu_rate_published": "0.000 %",
  "ca_base_rate_pct": 0.0,
  "ca_rate_published": "Free",
  "au_base_rate_pct": 0.0,
  "au_rate_published": "Free"
}
```

## Quick start
Python:
```python
from apify_client import ApifyClient
client = ApifyClient('<YOUR_APIFY_TOKEN>')
run = client.actor('phonic_lily/hs-code-duties-and-trade-flows').call(run_input={'maxResults': 10})
for item in client.dataset(run['defaultDatasetId']).iterate_items():
    print(item)
```
Or from the command line:
```bash
curl -X POST 'https://api.apify.com/v2/acts/phonic_lily~hs-code-duties-and-trade-flows/run-sync-get-dataset-items?token=<YOUR_APIFY_TOKEN>' \
  -H 'Content-Type: application/json' -d '{"maxResults": 10}'
```
Pricing is pay-per-record on Apify (see the Actor page); the sample here is free.

## Fields
| Field | Description |
|---|---|
| `hs6` | 6-digit Harmonized System code. |
| `hs2` | HS chapter (first 2 digits). |
| `hs4` | HS heading (first 4 digits). |
| `description` | Eurostat HS6 product description (English). |
| `us_base_rate_pct` | US base (column 1 general) duty in percent; null unless every US line under the HS6 is the same plain ad valorem rate. 0 = Free. |
| `us_rate_published` | US general rates as published, distinct values separated by ' / '. Null when the HS6 has no US line. |
| `eu_base_rate_pct` | EU third-country duty in percent; null unless every TARIC code under the HS6 carries the same plain ad valorem duty. |
| `eu_rate_published` | EU third-country duties as published, distinct values separated by ' / '. Null when the extract holds no such duty. |
| `ca_base_rate_pct` | Canada MFN duty in percent; null unless every line under the HS6 is the same plain ad valorem rate. |
| `ca_rate_published` | Canada MFN rates as published, distinct values separated by ' / '. |
| `au_base_rate_pct` | Australia general duty in percent; null unless every line under the HS6 is the same plain ad valorem rate. |
| `au_rate_published` | Australia general rates as published (preferential markers removed), distinct values separated by ' / '. |
| `flow_year` | Calendar year of the trade flows (2025). |
| `eu27_import_value_eur` | EU27 imports from non-EU countries, value in EUR. Null when Comext gives no value. |
| `eu27_import_weight_kg` | EU27 extra-EU imports, net mass in kg. Null when not given. |
| `eu27_import_top_partners` | Top 3 supplier countries of EU27 imports: partner_code (ISO), partner_name, value_eur, share_pct of the extra-EU total. Null when the partner figures do not rec |
| `eu27_import_partner_codes` | ISO codes of the top 3 import partners (for filtering). |
| `eu27_export_value_eur` | EU27 exports to non-EU countries, value in EUR. Null when Comext gives no value. |
| `eu27_export_weight_kg` | EU27 extra-EU exports, net mass in kg. Null when not given. |
| `eu27_export_top_partners` | Top 3 destination countries of EU27 exports: partner_code (ISO), partner_name, value_eur, share_pct. Null when the partner figures do not reconcile with the tot |
| `eu27_export_partner_codes` | ISO codes of the top 3 export partners (for filtering). |

## Data, licence and attribution
One record per 6-digit Harmonized System (HS6) code that has an EU27 trade flow, crossing two things. (1) The base import duty rate of four jurisdictions (United States, European Union, Canada, Australia), read from their national tariff lines. (2) The real extra-EU27 trade of the European Union for calendar year 2025: import and export value in EUR, net mass in kg and the top 3 partner countries, from Eurostat Comext.

Trade flows (single source: Eurostat Comext, dataset DS-045409 'EU trade since 1988 by HS2-4-6 and CN8', reporter EU27_2020, annual 2025, dataset updated 2026-10-16). Value is the Extra-EU27 total in EUR (partner EXT_EU27_2020); weight is the net mass indicator QUANTITY_IN_100KG multiplied by 100. Imports and exports are with non-EU countries only; trade between EU member states is excluded. A figure is returned only if it agrees with the source: the sum of the partner countries plus the special partner codes of extra-EU trade (high seas, stores and provisions, countries not specified) must be within 1 percent of the extra-EU27 total, otherwise the partner list is null; the special codes are never ranked as partners. A missing value or mass is null, never 0. Every HS6 code that Comext reports as traded is used; the description is the Eurostat HS6 label.

Base duty rule (per jurisdiction and per HS6): each national tariff line under the HS6 is read for its erga omnes / most-favoured-nation rate (US: general rate of column 1; EU: third-country duty; Canada: MFN tariff; Australia: general rate, the value placed before the preferential-rate markers). A rate in percent (pct field) is returned only if every line under the HS6 is a plain ad valorem rate ('Free' counts as 0) AND all lines carry the same value. It is null for specific duties (per kg, per litre, per head, EUR per 100 kg), mixed or compound duties, phase-in schedules with future dates, lines that differ from each other, and EU lines for which the extract holds no third-country duty. EU TARIC codes with a 10-digit suffix other than 00 inherit the duty of their 8-digit parent when they have none of their own, and the generic HS6 node (suffix 0000) is ignored when finer codes exist. The raw published text of the distinct rates (up to 6) is always returned next to it so the reader can judge. Preferential, anti-dumping and other special measures are not included: this is the base rate, not the rate applied to a given origin.

Sources and licences of the tariffs:
- United States: HTS lines (code, description, general duty rate). USITC, public domain.
- European Union: TARIC measures, third-country duty. DG TAXUD, CC BY 4.0 (Decision 2011/833/EU), changes made.
- Canada: Customs Tariff items, MFN rate. CBSA, Reproduction of Federal Law Order SI/97-5; not an official version.
- Australia: Working Tariff (Schedule 3) codes and rate. Australian Border Force, CC BY 3.0 Australia, changes made.
- United Kingdom: not included. The only UK data held is the list of commodity codes under a tariff quota, with no duty rate.

No natural person appears in the data. National HS versions differ slightly: an HS6 absent from a jurisdiction simply has no line there (rate null). Informational only; the official national tariffs and Eurostat prevail. Null means the source gives no certain value; it is never 0.

**Licence:** Mixed, per source: Eurostat Comext reuse authorised with acknowledgement of the source (Commission Decision 2011/833/EU); US HTS public domain (USITC); EU TARIC CC BY 4.0 (DG TAXUD); Canada Reproduction of Federal Law Order SI/97-5 (CBSA, unofficial copy); Australia CC BY 3.0 AU (ABF)

**Attribution:** Trade flows: Source: Eurostat, Comext international trade database (dataset DS-045409, EU27 extra-EU trade, calendar year 2025, updated 2026-10-16); reuse authorised with acknowledgement of the source under Commission Decision 2011/833/EU; changes made: aggregated by HS6, partners ranked, net mass converted from 100 kg to kg. Duties: United States International Trade Commission, HTS (public domain); European Commission DG TAXUD, TARIC (CC BY 4.0, Decision 2011/833/EU; changes made: third-country duty read per code and summarised by HS6); Canada Border Services Agency, Customs Tariff (Reproduction of Federal Law Order SI/97-5, not an official version); Australian Border Force, Working Tariff Schedule 3 (CC BY 3.0 Australia; changes made: general rate read per code and summarised by HS6).

Snapshot date: Tariffs: July 2026 extraction; trade flows: calendar year 2025, Eurostat Comext dataset updated 2026-10-16. Empty values are `null`, never 0.

## Cite this dataset
GitHub's *Cite this repository* button reads [`CITATION.cff`](CITATION.cff).

## Feedback
Found an error or want a field added? Open an issue in this repository.
