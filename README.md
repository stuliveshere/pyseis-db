# pyseis-db

A lightweight, stand‑alone Python tool for scanning SEG‑Y / SEG‑D files into an analytics‑ready Parquet database.

The goal is simple: **analyse and QC multi‑terabyte surveys without pulling every trace off disk each time**. We scan each file once, save the essentials, and work from that lightweight cache.

**Why pre‑scan?**

* Eliminates repeat I/O—hours of reads shrink to milliseconds.
* Enables instant SQL/pandas filtering and joins on millions of traces.
* Surfaces data issues (dead traces, header gaps) before full processing.

**Why Parquet?**

* Columnar and compressed—about 10× smaller and reads only the columns you ask for.
* Works everywhere: Python, R, Julia, DuckDB, Spark, cloud warehouses.
* Append‑friendly schema: add new surveys or QC fields without rewriting old data.

---

## Why pyseis-db?

* **Customisable** – parse either a built‑in default header set or your own YAML definition.
* **Portable** – writes highly‑compressed Parquet tables that open in Python, R, Julia, DuckDB, Spark, and more.
* **Incremental** – append new surveys to an existing QC database with a single command.
* **Fast QC** – trace‑level statistics (`pandas.DataFrame.describe`) pre‑computed for instant exploration.

---

## Key Features

| Feature                 | Details                                                                                  |
| ----------------------- | ---------------------------------------------------------------------------------------- |
| **Custom SEG‑Y reader** | Pure‑Python; no `segyio` dependency.                                                     |
| **Header extraction**   | Reads line/shot/CDP, sample interval, coordinates and any fields you define in YAML.     |
| **Relational schema**   | Four core Parquet tables (file -> shot -> receiver -> trace) with hashes for fast joins. |
| **Trace statistics**    | Mean, RMS, min, max, std … per trace; stored in the **trace** table.                     |
| **Multi‑input modes**   | Single file, recursive directory scan, or text file list.                                |
| **Seamless append**     | Adds rows to an existing Parquet dataset without rewriting.                              |
| **DuckDB support**      | Optional DuckDB engine lets you run ad‑hoc SQL instantly.                                |

---

## Data Model

```
file.parquet        # filename, md5, filename_hash
shot.parquet        # ffid, line, station, x, y, z
receiver.parquet    # rec_id, line, station, x, y, z
trace.parquet       # trace_id, file_hash, shot_id, rec_id,
                    # trace_no_in_file, channel_no,
                    # mean, rms, std, min, max, ...
```

All tables share integer or hash keys so you can join them in any analytics engine.

---

## Technical Stack

| Layer        | Library           |
| ------------ | ----------------- |
| Language     | Python ≥ 3.8      |
| In‑memory    | NumPy, pandas     |
| Storage      | pyarrow / Parquet |
| YAML parsing | PyYAML            |
| SQL engine   | DuckDB (optional) |

> Note: The SEG‑Y reader is implemented in pure Python—no external C libraries or `segyio` required.

---

## Installation

```bash
pip install pyseis-db        # coming soon
```

Or install from a local clone while you develop:

```bash
git clone https://github.com/your-org/pyseis-db.git
cd pyseis-db
pip install -e .
```

---

## Usage

```bash
# 1. Scan a single file into a new Parquet dataset
pyseis-db scan survey.sgy --out qc_dataset

# 2. Append another file
pyseis-db scan new_line.sgy --out qc_dataset --append

# 3. Recursively scan a directory
pyseis-db scan /data/segy/ --recursive --out qc_dataset

# 4. Use a YAML header map
pyseis-db scan filelist.txt --header-map headers.yaml --out qc_dataset
```

### Minimal YAML header example

```yaml
# headers.yaml
line_number: { byte_start: 9,  byte_end: 12, dtype: int32 }
shot_point : { byte_start: 17, byte_end: 20, dtype: int32 }
cdp_number : { byte_start: 21, byte_end: 24, dtype: int32 }
```

---

## Command‑line Options

| Flag                  | Purpose                                                         |
| --------------------- | --------------------------------------------------------------- |
| `--out <path>`        | Destination directory for Parquet tables.                       |
| `--append`            | Append to existing dataset instead of creating a new one.       |
| `--recursive`         | Recursively scan sub‑directories.                               |
| `--header-map <yaml>` | Path to a custom YAML header definition.                        |
| `--workers N`         | Number of parallel read workers (default: number of CPU cores). |

Run `pyseis-db --help` for the full list.

---

## Example DuckDB Query

```sql
-- List the ten traces with the highest RMS
SELECT t.trace_id, f.filename, t.rms
FROM 'qc_dataset/trace.parquet' AS t
JOIN 'qc_dataset/file.parquet'  AS f
ON t.file_hash = f.filename_hash
ORDER BY t.rms DESC
LIMIT 10;
```

---

## Road‑map

*

---

## Contributing

Pull requests and issues are welcome! See `CONTRIBUTING.md` for guidelines.

---

## License

Distributed under the MIT License—see `LICENSE` for details.
