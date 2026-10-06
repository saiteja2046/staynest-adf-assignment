# StayNest ADF Assignment

Azure Data Factory pipeline that copies `staynest/raw/hotels.csv` to `staynest/bronze/hotels.csv`, then lists the raw folder using Get Metadata.

## Resources

| Resource | Purpose |
| --- | --- |
| `ls_staynest_storage` | Reusable ADLS Gen2 connection using the factory system-assigned managed identity |
| `ds_source` | Delimited-text dataset reading `staynest/raw/hotels.csv` |
| `ds_sink` | Delimited-text dataset writing to `staynest/bronze` |
| `ds_raw_folder` | Delimited-text dataset pointing to `staynest/raw`, with the filename empty |
| `pl_staynest_copy` | `Copy_Hotels` followed on success by `List_Raw_Files` |

Both copy datasets use a comma delimiter and first row as header. The copy sink uses Preserve Hierarchy and a `.csv` extension to preserve `hotels.csv`. No dataset parameters or dynamic expressions are needed for the four required tasks.

## How to run

1. Upload the supplied `hotels.csv`, `customers.csv`, and `bookings.csv` into `staynest/raw`.
2. Open the connected Data Factory Studio in GitHub mode and select the `main` branch.
3. Under Manage > Linked services, open `ls_staynest_storage`. Test connection using **To file path**, file system `staynest`, directory `raw`. The managed identity has access within the assignment container; an account-wide test against the default `testconnection` container can return Forbidden.
4. Under Author > Pipelines, open `pl_staynest_copy`, click Validate, then Debug.
5. Confirm both activities succeed. Open `List_Raw_Files` > Output to inspect `childItems`. Verify `hotels.csv` in the bronze folder.
6. Save all to commit JSON to GitHub. For a run in Monitor > Triggered, Publish, then Add trigger > Trigger now. No recurring schedule is configured.

## Verified results

Verified on 6 October 2026:

- Linked service connection test to `staynest/raw`: successful.
- Pipeline validation: no errors.
- Debug run: `Copy_Hotels` and `List_Raw_Files` both succeeded.
- Published manual run: pipeline and both activities succeeded in Monitor.
- Bronze output: `hotels.csv` present, 1.53 KiB.
- Metadata output: the three entries below, all of type `File`.

```json
{
  "childItems": [
    { "name": "bookings.csv", "type": "File" },
    { "name": "customers.csv", "type": "File" },
    { "name": "hotels.csv", "type": "File" }
  ]
}
```

## Submission evidence

Screenshots of the pipeline canvas, successful Debug run, metadata output, successful connection test, bronze folder, and Monitor results have been captured. Their upload to `screenshots/` is pending approval to publish the account and run identifiers visible in the images.

The `main` branch contains the authoring JSON under `factory/`, `linkedService/`, `dataset/`, and `pipeline/`. ADF generated deployment templates are on `adf_publish`. No storage keys or connection strings are included.

## Understanding the pattern

ADF orchestrates and moves data; complex transformations can be delegated to compute such as Databricks or SQL. The optional, ungraded stretch would use a filename parameter and a ForEach over metadata `childItems`, passing `@item().name` to a Copy activity. One reusable parameterized dataset can then handle new files without creating a separate dataset for each filename.
