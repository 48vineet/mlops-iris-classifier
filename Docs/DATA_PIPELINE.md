vc repro (first run) executes all 4 stages in order and terminates with exit code 0. Expected terminal output includes:

Running stage 'collect':

> python src/pipeline/collect.py --output data/raw/iris_raw.csv
> 2026-XX-XX ... [INFO] Collected 150 rows -> data/raw/iris_raw.csv

Running stage 'preprocess':

> python src/pipeline/preprocess.py --input data/raw/iris_raw.csv --output data/processed/iris_preprocessed.csv
> 2026-XX-XX ... [INFO] Dropped 0 duplicate rows
> 2026-XX-XX ... [INFO] Preprocessed 150 rows -> data/processed/iris_preprocessed.csv

Running stage 'features':

> python src/pipeline/features.py ...
> 2026-XX-XX ... [INFO] Engineered 9 features -> data/processed/iris_features.csv

Running stage 'validate':

> python src/pipeline/validate.py ...
> 2026-XX-XX ... [INFO] Validation PASSED: 150 rows, 9 columns, all checks satisfied

Use `dvc push` to send your updates to remote storage.

Running dvc repro a second time without any changes reports Stage '...' didn't change, skipping, for all four stages — confirming DVC’s dependency-based caching correctly avoids redundant re-execution.

dvc dag displays the pipeline dependency graph, confirming the correct linear order: collect -> preprocess -> features -> validate.

To confirm the validation stage correctly halts on bad data: manually corrupt data/processed/iris_features.csv (e.g., set a sepal length (cm) value to 50), re-run python src/pipeline/validate.py, and confirm it exits with code 1 and logs a range-check error.

dvc.lock is generated/updated after dvc repro, capturing the exact hashes of all stage dependencies and outputs.
