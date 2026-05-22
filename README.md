# US AI Map — Geo Data Prep

Data preparation pipeline that produces the four Parquet artifacts consumed by the `us_ai_map` Django application. The pipeline joins occupation-level AI-exposure scores (Anthropic, Eloundou et al.) with ONET task labor-input shares and BLS OEWS area employment to estimate, for every `(occupation, task, geography)` triple in the United States, an employment-weighted measure of how exposed local labor markets are to current AI capabilities.

The entire pipeline lives in [geo_data_prep.ipynb](geo_data_prep.ipynb).

---

## Inputs

| Source | File / URL | Purpose |
| --- | --- | --- |
| BLS OEWS 2024 | [data/OEWS/all_data_M_2024.csv](data/OEWS/all_data_M_2024.csv) — download the "All Data [XLSX]" file from [bls.gov/oes/tables.htm](https://www.bls.gov/oes/tables.htm) and export to CSV. | Area × occupation employment (`TOT_EMP`) and annual mean wage (`A_MEAN`) for state, MSA and nonmetro areas. |
| ONET 30.2 | `Task Statements.txt` (fetched) | `(O*NET-SOC Code, Task ID, Task)` lookup. |
| ONET job families | `onetonline.org/find/family/All_Job_Families.csv` (fetched) | SOC → job family mapping. |
| MIT-WAL `job_task_input_share` | HuggingFace (fetched) | Per-task labor input share `pi` within each occupation. |
| Anthropic Economic Index | HuggingFace `Anthropic/EconomicIndex` (fetched) | Observed AI `penetration` per task. |
| Eloundou et al. — *GPTs are GPTs* | GitHub `openai/GPTs-are-GPTs` (fetched) | Theoretical AI exposure score `beta` per task. |

All remote CSVs are memoised as Parquet under [data/_cache/](data/_cache/) on first run. Delete a cache file to force a refresh; the notebook is otherwise fully offline on reruns.

### Scoring conventions

- `pi` is renormalised so it sums to 1 within every `soc_code` *after* the three-way `(anthropic × eloundou × onet)` inner merge — this preserves the "share of labor input" interpretation despite tasks dropped by the joins.
- Eloundou's `E1` class (originally `beta == 1.0`, meaning "AI accelerates this task by ≥50%") is clamped to `0.5`, the defensible scalar lower bound of that bucket.
- Only ONET *detailed* occupations are kept (codes ending in `.00`); the `.XX` suffix is stripped before any geographic fan-out so downstream joins key cleanly on the 6-digit SOC code.

---

## Outputs

All artifacts are written to [data/output/](data/output/) as Snappy-compressed Parquet with compact dtypes (`int8` / `int32` / `float32`) to minimise file size and Django ingest time.

### `state_map_data.parquet` — state fact table

One row per `(soc_code, task_id, state_code)`. Built from OEWS `AREA_TYPE ∈ {2, 3}`.

| Column | Type | Description |
| --- | --- | --- |
| `soc_code` | string | 6-digit SOC occupation code (e.g. `11-1011`). |
| `task_id` | int32 | ONET task identifier. |
| `state_code` | int8 | FIPS state code (1–56). |
| `weight` | float32 | Task-allocated employment, computed as `OEWS TOT_EMP × pi`. Sum over `task_id` for a given `(soc_code, state_code)` reconstructs the occupation's state-level employment. |
| `a_mean` | float32 | Occupation annual mean wage in the state (USD). |
| `pi` | float32 | Task's share of the occupation's labor input (renormalised, sums to 1 per `soc_code`). |
| `penetration` | float32 | Anthropic observed AI penetration for the task, `[0, 1]`. |
| `beta` | float32 | Eloundou theoretical exposure for the task, `{0, 0.25, 0.5}` after E1 clamp. |
| `job_family` | string | ONET job family (e.g. `Management`). |

Coverage at last run: **87.72 %** of the OEWS state-level workforce, **87.30 %** of the wage bill. Drops are tasks/occupations missing from the Anthropic or Eloundou label sets.

### `msa_map_data.parquet` — metro / nonmetro fact table

Same schema as `state_map_data.parquet` with two differences:

| Column | Type | Description |
| --- | --- | --- |
| `msa_code` | int32 | CBSA code (metro) or BLS nonmetro area code. |
| `area_type` | int8 | `4` = metropolitan, `6` = nonmetropolitan. |

Built from OEWS `AREA_TYPE ∈ {4, 6}`. Coverage at last run: **87.77 %** workforce / **87.39 %** wage bill.

### `task_dim.parquet` — task dimension

| Column | Type | Description |
| --- | --- | --- |
| `Task ID` | int32 | ONET task identifier (joins to `task_id` on the fact tables). |
| `Task` | string | Human-readable task statement. |

Column names are intentionally preserved as `Task ID` / `Task` to match the existing Django loader. Restricted to task ids that survive into the fact tables (~14k rows).

### `occupation_dim.parquet` — occupation dimension

| Column | Type | Description |
| --- | --- | --- |
| `onetsoc_code` | string | 6-digit SOC code (joins to `soc_code` on the fact tables). |
| `onetsoc_title` | string | OEWS occupation title. |
| `job_family` | string | ONET job family. |

One row per occupation present in the fact tables (~725 rows).

---

## Running

```bash
jupyter nbconvert --to notebook --execute geo_data_prep.ipynb --inplace
```

Or open `geo_data_prep.ipynb` and run all cells. Requirements: `pandas`, `numpy`, `pyarrow`. The first run downloads ~5 MB of remote CSVs; subsequent runs are offline.

Free disk required: ~120 MB for the OEWS CSV input, ~20 MB for the Parquet outputs, ~4 MB for the remote-source cache.

---

## Layout

```
.
├── geo_data_prep.ipynb          # the pipeline
├── data/
│   ├── OEWS/all_data_M_2024.csv # BLS OEWS source (not redistributed)
│   ├── _cache/                  # memoised remote sources (regenerated)
│   └── output/                  # the four Parquet artifacts
└── licence.md
```

## License

See [licence.md](licence.md) — CC BY-NC-ND 4.0.
