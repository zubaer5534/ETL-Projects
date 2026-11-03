# Data Orchestration in Microsoft Azure

An end-to-end Azure data engineering project that builds a data flow and data pipeline, then adds pipeline orchestration with success and failure handling.

**Tools:** Azure Blob Storage, Azure Data Factory (Data Flows and Pipelines), Azure SQL Database, Visual Studio Code

---

## Part 1: Simple Data Flow and Data Pipeline

1. **Ingest:** Uploaded the CSV files to a container folder in Azure Blob Storage.
2. **Transform:** Built a data flow that joins and filters the two source tables.
3. **Load:** Wrote the transformed dataset to Azure SQL Database under the `dbo` schema. The database was connected to Visual Studio Code for querying.
4. **Validate and publish:** Ran **Validate all** to check for errors or breaks, then published the data flow for deployment.
5. **Operationalize:** Added the data flow to a pipeline, validated it, and triggered it. The resulting table was visible in VS Code for read and write.

<img width="1517" height="440" alt="Screenshot 2026-11-02 155700" src="https://github.com/user-attachments/assets/75e8395d-a17c-4597-a5b0-033a27ddf1e3" />


---

## Part 2: Pipeline Orchestration

Orchestration matters because downstream reports and dashboards depend on upstream data. If a step fails, the pipeline should detect it and alert you instead of silently feeding bad data to a report.

### Simulating a real-world scenario

1. Deleted the source files from the original Blob folder (`bluecross`) and uploaded them to a new source folder (`bluecrosstext`), so the same data could be reprocessed with the same data flow.
2. Dropped the table previously created in Azure SQL Database so the pipeline could recreate it from scratch. This required multiple pipelines.

### Building the orchestration pipeline

1. Created a new pipeline named `PaypalPipeline`.
2. Added a **Copy data** activity named `PaypalCopyPipeline` with a timeout of 1 hour, and set its source to the new Blob folder (`bluecrosstext`).
3. Repeated the process to create a second Copy data activity for the other source file, chained to run **on success** of the first.
4. Added two branches after the Copy activities:
   - **On success:** an **Execute Pipeline** activity that runs `PaypalTransformationPipeline`.
   - **On failure:** a **Fail** activity named `paypalFail` that returns an error message if the pipeline breaks.
5. Created a second data flow that reads from Azure SQL Database and writes to an **inline sink** named `exporttocsv`, exporting the data as CSV to a Blob folder.

<img width="1172" height="485" alt="Screenshot 2026-11-02 155827" src="https://github.com/user-attachments/assets/dfa99495-26cf-427c-9f66-1f9848440135" />
  
7. Added this data flow to the orchestration pipeline, connected **on success**.

<img width="1402" height="587" alt="Screenshot 2026-11-02 160349" src="https://github.com/user-attachments/assets/83c3954d-f47e-4d65-b8f2-716bbaf2d7f8" />

8. Ran **Validate all** (no errors) and published everything.
9. Triggered the pipeline. The exported CSV file was created in Blob Storage.

<img width="1762" height="850" alt="Screenshot 2026-11-02 160608" src="https://github.com/user-attachments/assets/a8657e6d-199a-4907-921c-e00317767dcb" />


## Key Takeaways

- Built a complete ingest, transform, load, and export workflow in Azure.
- Used activity dependencies (on success / on failure) to control pipeline flow.
- Added failure handling so errors surface immediately rather than breaking dashboards downstream.
- Validated and published all components before triggering runs.


