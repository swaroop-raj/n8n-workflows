# n8n-workflows

Reusable n8n workflow exports by Swaroop Raj. Configure credentials and resource identifiers in your own n8n instance before running any workflow.

## Workflow catalog

| Project | Files | Purpose |
| --- | --- | --- |
| [Financial Advisor Agent](workflows/financialAdvisorAgent/) | `financialAdvisor.json`, `getSmartAlerts.json` | Existing financial-advisor and smart-alert workflow exports. |
| [Automated Financial Reporting](workflows/automated-financial-reporting/) | `automated-financial-reporting-agent.json`, `document-vector-ingestion-pipeline.json` | Document ingestion into Pinecone and retrieval-grounded NovaCart financial report generation. |

## Automated Financial Reporting

### Included workflows

- **Document vector ingestion pipeline (20 nodes):** manually searches separate internal and external Google Drive folders, downloads files, splits document text into 500-character chunks with 100-character overlap, and inserts embeddings into Pinecone. Internal files are routed by MIME type for PDF extraction or DOCX binary loading; the external branch extracts PDFs. Document metadata labels sources as `internal` or `external`.
- **Automated Financial Reporting Agent (11 nodes):** a monthly schedule downloads an Excel sales workbook, reads rows, filters `Week_Start_Date` to the configured period, and passes the rows to an AI agent. The agent has internal and external Pinecone retrieval tools and an HTML report template. Its output is passed to Gmail for delivery. The connected chat model is `gpt-5-mini`.

### Prerequisites

- An n8n instance supporting the node types and versions declared in the exports, including the AI Agent, Pinecone Vector Store, OpenAI Chat Model, OpenAI Embeddings, and document loader nodes. Import compatibility must be checked in your instance; no minimum n8n version has been verified.
- Google Drive OAuth access to the document folders and the sales workbook, and Gmail OAuth access for report delivery.
- OpenAI access for chat and embeddings. Both workflows use `text-embedding-3-large` with **1536 dimensions**. The original managed OpenAI credential references have been removed; bind your own supported credentials.
- A Pinecone index named `novacart-vectors`, or replace that index selection consistently in every ingestion and retrieval node. Configure it for 1536-dimensional vectors and use consistent namespace settings across both workflows.
- Internal PDF/DOCX documents, external PDF documents, and an Excel workbook with a `Week_Start_Date` column and the relevant sales fields. Source documents and business data are not included.

### Import and configure

1. Download both JSON files from `workflows/automated-financial-reporting/` and import each through n8n's workflow import interface. Both exports are inactive.
2. Reconnect credentials on every Google Drive, Gmail, Pinecone, and OpenAI node as applicable. The disconnected OpenRouter node is retained from the original export and is not required for the connected execution path.
3. In the ingestion workflow, replace `REPLACE_WITH_INTERNAL_RESOURCES_FOLDER_ID` and `REPLACE_WITH_EXTERNAL_RESOURCES_FOLDER_ID` in the two Drive search nodes with your folder IDs.
4. In the reporting workflow, replace `REPLACE_WITH_SALES_SPREADSHEET_FILE_ID` in **Download file**, and replace `recipient@example.com` in **Send a message** with a controlled test recipient.
5. Review **Filter Rows for Period**. The supplied sample is fixed to `2022-12-01` through `2022-12-31`; changing the schedule does not advance that date range. Update these dates for your intended reporting period.
6. Verify Pinecone index, namespace, credentials, embedding model, and dimensions match throughout. Keep `source_type` metadata consistent between ingestion and retrieval.
7. Run ingestion manually with representative documents, inspect extracted text and metadata, then test retrieval and report generation before enabling any schedule.
8. Review the monthly trigger's timing and instance timezone. Confirm Gmail body formatting and recipient, then activate only after a successful end-to-end test.

### Known limitations and review checklist

- These are sanitized assignment exports, not production-validated workflows. JSON syntax and connection targets were checked statically; no live n8n execution, provider authentication, retrieval quality, or email delivery was tested.
- The reporting date window is hard-coded to December 2022, while the HTML template includes annual-report sections. Align the analysis period and narrative before use.
- The report prompt instructs the agent to rely only on retrieved vector-store content, while spreadsheet rows are supplied in the input. Review this grounding policy so intended spreadsheet analysis is not excluded by the prompt.
- The OpenRouter model node is disconnected and preserved unchanged. The active model connection uses OpenAI.
- Only the supported MIME-type routes are configured: internal PDF/DOCX and external PDF. Unsupported file types have no explicit fallback route. Verify DOCX parsing and the extracted PDF text field selected by the default data loaders in your n8n instance.
- Internal/external source labels are configured, but source filename, date, author, and document-type metadata are not explicitly mapped by these exports. Add and verify provenance metadata before relying on the prompt's citation requirements.
- A retrieval tool query does not guarantee exhaustive retrieval of every relevant document. Inspect source filtering and retrieved chunks for both tools.
- Retries, failure notifications, deduplication/upsert controls, and output validation are not explicitly configured. Re-running ingestion may create duplicate vectors depending on node behavior.
- Check generated HTML and citations manually. Prompt instructions alone do not guarantee grounded financial claims.

### Public-export sanitization

Credential bindings, cached Drive resource names/URLs, private Drive IDs, the original recipient email, workflow instance IDs, version IDs, and the Gmail webhook ID were removed or replaced with placeholders. Workflow logic, node IDs, connections, prompts, model settings, and the sample Pinecone index name were retained. No source documents, workbook contents, or credentials are included in these new files. Existing financial-advisor exports were not modified or audited as part of this change.
