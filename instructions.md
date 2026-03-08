# Instructions for Working with Peviitor Project

This directory contains project files for interacting with the peviitor.ro job search platform.

## Key Resources

### 1. Job & Company Models
Read the README from peviitor-core to understand the data models:
- **URL**: https://github.com/peviitor-ro/peviitor_core/blob/main/README.md
- **Content**: Job Model Schema and Company Model Schema

### 2. Solr Schemas
Access the live Solr instance to see the actual field definitions:
- **Base URL**: https://solr.peviitor.ro
- **Credentials**: `solr:SolrRocks`

#### Job Core Schema
```bash
curl -u solr:SolrRocks "https://solr.peviitor.ro/solr/job/schema"
```

#### Company Core Schema
```bash
curl -u solr:SolrRocks "https://solr.peviitor.ro/solr/company/schema"
```

## Job Model Fields (from Solr)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `url` | string | Yes | Full URL to job detail page (unique key) |
| `title` | text_general | Yes | Position title |
| `company` | string | No | Hiring company name, Always use uppercase. |
| `cif` | string | No | CIF/CUI of the company |
| `location` | text_general | No | Romanian cities/addresses |
| `tags` | text_general[] | No | Skills/education/experience |
| `workmode` | string | No | "remote", "on-site", "hybrid" |
| `date` | pdate | No | Scrape date (ISO8601) |
| `status` | string | No | "scraped", "tested", "published", "verified" |
| `vdate` | pdate | No | Verified date |
| `expirationdate` | pdate | No | Job expiration date |
| `salary` | text_general | No | Format: "MIN-MAX CURRENCY" |

## Company Model Fields (from Solr)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | CIF/CUI (unique key) |
| `company` | string | Yes | Legal name from Trade Register . always use uppercase.|
| `brand` | string | No | Commercial brand name |
| `group` | string | No | Parent company group |
| `status` | string | No | "activ", "suspendat", "inactiv", "radiat" |
| `location` | text_general | No | Romanian cities/addresses |
| `website` | string[] | No | Official company website(s) |
| `career` | string[] | No | Career page URL(s) |
| `lastScraped` | string | No | Last scrape date (ISO8601) |
| `scraperFile` | string | No | Name of scraper file used |

## Workflow: Verifying Jobs

### Step 1: Find Unverified Jobs in SOLR
When starting a new verification session, query SOLR for jobs that are NOT verified:
```bash
curl -u solr:SolrRocks "https://solr.peviitor.ro/solr/job/select?q=status:scraped&wt=json&rows=1"
```

### Step 2: Open Job URL
Navigate to the job's URL from the SOLR response.

### Step 3: Extract & Verify Data
Follow the extraction process in AGENTS.md
transform company name to uppercase

### Step 4: Push Updates to SOLR
Use **atomic update** to add verified fields. Status should be set to "verified".

**Important**: Use atomic update with `{"set": "value"}` to preserve existing fields:

```bash
curl -u solr:SolrRocks -X POST -H "Content-Type: application/json" \
  "https://solr.peviitor.ro/solr/job/update?commit=true" \
  -d "{\"add\": {\"doc\": {\"url\": \"<JOB_URL>\", \
  \"company\": {\"set\": \"<company>\"}, \
  \"cif\": {\"set\": \"<cif>\"}, \
  \"salary\": {\"set\": \"<salary>\"}, \
  \"workmode\": {\"set\": \"<workmode>\"}, \
  \"tags\": {\"set\": [\"tag1\", \"tag2\"]}, \
  \"status\": {\"set\": \"verified\"}, \
  \"vdate\": {\"set\": \"2026-03-08T00:00:00Z\"}}}}"
```

### Step 5: Verify the Update in SOLR
Always query SOLR to confirm all fields were updated correctly:
```bash
curl -u solr:SolrRocks "https://solr.peviitor.ro/solr/job/select?q=url:<JOB_URL>&wt=json"
```

**Note**: Date fields (vdate, date, expirationdate) must use ISO8601 format: `YYYY-MM-DDTHH:MM:SSZ`

### Step 6: Handle Expired Jobs
If job is no longer available on the original URL:
```bash
curl -u solr:SolrRocks -X POST -H "Content-Type: application/json" \
  "https://solr.peviitor.ro/solr/job/update?commit=true" \
  -d "{\"delete\": [\"<JOB_URL>\"]}"
```

## Notes
- Use `curl` with `-u solr:SolrRocks` for authentication
- The Solr instance uses `text_general` field type for most text fields
- Both cores have copy fields that aggregate text to `_text_` for full-text search
