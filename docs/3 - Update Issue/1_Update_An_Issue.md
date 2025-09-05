# Update an Issue 

**Folder:** `3 - Update Issue`  
**Method:** `PUT`  
**URL:** `https://<your-domain>.atlassian.net/rest/api/3/issue/{{multiLangIssueKey}}`

**Path Params:**
- `multiLangIssueKey = {{multiLangIssueKey}}`

## Purpose
Updates the issue’s **description** (ADF format) for the issue captured earlier as `multiLangIssueKey`.  
Runs **only** on Newman **iteration 2** (`pm.info.iteration === 1`) and skips otherwise.

## Authentication
- **Basic Auth**
  - Username: `{{username}}`
  - Password (API token): `{{jiraauth}}`

## Request Body
```json
{
  "fields": {
    "description": {
      "type": "doc",
      "version": 1,
      "content": [
        {
          "type": "paragraph",
          "content": [
            { "type": "text", "text": "Hi! This is the updated description" }
          ]
        }
      ]
    }
  }
}
```

## Variables Used / Set
- **Reads (Collection):**
  - `multiLangIssueKey`
  
## Pre-request Script
```javascript
const issueKey = pm.collectionVariables.get("multiLangIssueKey");

// Only run update on iteration 1 when issueKey exists
if (!issueKey || pm.info.iteration !== 1) {
    console.log(
        !issueKey
            ? "Skipping Update Issue because multiLangIssueKey is missing"
            : `Skipping Update Issue because iteration is ${pm.info.iteration}`
    );
    return pm.execution.skipRequest();
}

console.log(`Running Update Issue with key: ${issueKey} (iteration ${pm.info.iteration})`);
```

## Post-request Script
```javascript
const issueKey = pm.collectionVariables.get("multiLangIssueKey");

if (issueKey && pm.info.iteration === 1) {
    pm.test("Status code is 204", () => pm.response.to.have.status(204));
    console.log(`Validated: Issue ${issueKey} updated successfully (204)`);
} else {
    console.log("Skipping status code validation (request not meant to run)");
}
```

## Sample Output (Newman)
Captured from a real run. Shows 200 OK, response parsed, variables set, then jump to the next request.

**Case — Runs on iteration 2 (index 1):**

```bash
└ Update an issue
  ┌
  │ 'Running Update Issue with key: APIQA-2 (iteration 1)'
  └
  PUT https://<your-domain>.atlassian.net/rest/api/3/issue/APIQA-2 [204 No Content, 1.58kB, 364ms]
  √  Status code is 204
  ┌
  │ 'Validated: Issue APIQA-2 updated successfully (204)'
  └
```

**Case — Skipped on other iterations or when key missing:**

```bash
└ Update an issue
  ┌
  │ 'Skipping Update Issue because iteration is 2'
  └
```