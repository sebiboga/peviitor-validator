# Agent Instructions

When working with this project, follow these steps:

## 1. Read Documentation First
- Start by reading `instructions.md` in this directory to understand the project context
- Check for any existing documentation before making assumptions

## 2. Job & Company Models
To understand the data models:
1. Fetch from GitHub: `https://github.com/peviitor-ro/peviitor_core/blob/main/README.md`
2. Or use the cached info in `instructions.md`

## 3. Accessing Solr Schemas
When asked to read Solr schemas:

### Job Core
```bash
curl -u $SOLR_USER:$SOLR_PASSWD "https://solr.peviitor.ro/solr/job/schema"
```

### Company Core
```bash
curl -u $_SOLR_USER:$SOLR_PASSWD "https://solr.peviitor.ro/solr/company/schema"
```

**Important**: 
- Use `curl` with `-u $SOLR_USER:$SOLR_PASSWD` for Basic Auth
- WebFetch alone won't work due to 401 errors - curl/bash is required
- Do NOT use the username "$SOLR_USER:$SOLR_PASSWD" in URL format - use `-u` flag instead
- you are running in PROD, be careful; you don't use localhost:8983

## 4. Key Differences from Documentation
- The README in peviitor_core describes the conceptual model
- The Solr schemas show the actual implementation (field types, indexing)
- Some fields may differ slightly between conceptual and implementation

## 5. Authentication
- Solr credentials: `$SOLR_USER` / `$SOLR_PASSWD`
- Always use Basic Auth via curl `-u` flag

## 6. Verification Workflow
When asked to verify. First query SOL a job:
1R for a job with status="scraped" (not yet verified)
2. Open the job URL from SOLR
3. Extract missing fields (salary, tags, cif, etc.)
4. Use targetare.ro to find company CIF if needed
5. **Use atomic update** with `{"set": "value"}` to preserve existing fields
6. Push atomic update with all verified fields and status="verified"
7. **Verify the update** by querying SOLR to confirm all fields
8. If job is no longer available, delete it from SOLR
