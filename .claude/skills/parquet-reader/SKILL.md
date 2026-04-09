---
name: parquet-reader
description: Instructions for reading nested TRACAB parquet skeleton files using pyarrow and converting to polars DataFrames
user-invocable: false
---

# Parquet Reader Skill

## How to Read the Nested Skeleton Parquet

The TRACAB parquet file is **nested** (structs within arrays within structs). You cannot read it directly with `polars.read_parquet()`. Use pyarrow first, then convert.

## Step-by-Step Reading

```python
import pyarrow.parquet as pq
import polars as pl
import json

# 1. Read metadata from parquet file footer
parquet_file = pq.ParquetFile("data/FCU-FCB.parquet")
metadata = parquet_file.schema_arrow.pandas_metadata  # or use:
# metadata_dict = json.loads(parquet_file.schema_arrow.metadata[b'metadata_header'])

# 2. Read the file — this gives a pyarrow Table with nested columns
table = parquet_file.read()

# 3. Check schema to understand nesting
print(table.schema)

# 4. Access skeleton frames
# The exact column names depend on the file — inspect schema first
# Typical pattern:
#   table.column('frame_number')  — frame numbers
#   table.column('skeletons')     — nested array of skeleton targets
```

## Flattening Strategy

```python
# For each frame, flatten skeletons into rows
rows = []
for i in range(len(table)):
    frame_num = table.column('frame_number')[i].as_py()
    skeletons = table.column('skeletons')[i].as_py()  # list of dicts
    
    if skeletons is None:
        continue
    
    for skeleton in skeletons:
        team = skeleton['team']
        jersey = skeleton['jersey_number']
        parts = skeleton['parts']
        
        if parts is None:
            continue
        
        for part in parts:
            rows.append({
                'frame_number': frame_num,
                'team': team,
                'jersey_number': jersey,
                'part_id': part['name'],
                'x_cm': part['position_x'],
                'y_cm': part['position_y'],
                'z_cm': part['position_z'],
            })

# Convert to polars
df = pl.DataFrame(rows)
```

## Chunk Reading for Large Files

```python
# Read by row groups to limit memory
n_row_groups = parquet_file.metadata.num_row_groups

for rg_idx in range(n_row_groups):
    table_chunk = parquet_file.read_row_group(rg_idx)
    # Process chunk...
    # Each row group typically contains ~1000-5000 frames
```

## Performance Tips

- **Never** load the entire parquet into memory for a full match (141M data points)
- Process row group by row group
- After flattening to polars, filter immediately for the joints you need (e.g., pelvis only for speed)
- For body orientation, only need: l_shoulder(4), r_shoulder(6), l_hip(11), r_hip(13), pelvis(12)
- Save intermediate results to parquet for reuse

## Alternative: DuckDB

```python
import duckdb
# DuckDB can handle nested parquet natively
con = duckdb.connect()
result = con.execute("""
    SELECT * FROM READ_PARQUET('data/FCU-FCB.parquet')
    WHERE frame_number = 3752711
    LIMIT 1
""").fetchall()
```

Note: DuckDB may handle the nested flattening automatically — test first before writing custom code.
