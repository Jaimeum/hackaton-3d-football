# Coding Standards — Phantom Runs Project

## Data Processing
- ALWAYS use `polars` over `pandas` — the dataset is 141M data points per match
- Use `pyarrow` for reading nested parquet, then convert to polars for analysis
- Read parquet in chunks (by row group), never load entire file to memory
- For exploratory work, sample first 1000 frames before running on full match

## Coordinates
- Skeleton data (parquet): positions in **centimeters** from pitch center
- Positional data (XML): positions in **meters** (x,y) from pitch center
- Always convert skeleton cm → meters (divide by 100) before combining data sources
- Speed in positional XML is km/h; skeleton velocity must be computed from displacement

## File Organization
- Raw data: `data/` (never modify, never commit to git)
- Processed outputs: `output/{match_id}/` (parquet files + JSON summaries)
- Visualizations: `output/{match_id}/viz/` (PNG at 300dpi and 150dpi)
- Documentation: `docs/` (original PDFs from DFL/TRACAB)
- Source code: `src/` for Python modules, `notebooks/` for Jupyter notebooks

## AWS Budget
- Total budget: $50 USD
- Stop SageMaker instances immediately after use
- Prefer local development with sample data, deploy to SageMaker only for full runs
- Use S3 for data storage, not EBS volumes

## Code Quality
- Type hints on all function signatures
- Docstrings with parameter descriptions and return types
- Log progress with timestamps for long-running operations
- Save intermediate results to parquet for debugging and reuse

## Git Hygiene
- NEVER commit data files (parquet, xml) to the repo
- Commit at least once per hour when actively working
- Branch per feature: `feature/skeleton-processing`, `feature/phantom-detection`, etc.
- Main branch should always have a working pipeline
