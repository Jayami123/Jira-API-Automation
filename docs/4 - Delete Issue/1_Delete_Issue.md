# Delete Issue 

**Folder:** `4 - Delete Issue`  
**Method:** `DELETE`  
**URL:** `https://<your-domain>.atlassian.net/rest/api/3/issue/{{codeReviewKey}}`

**Path Params:**
- `codeReviewKey = {{codeReviewKey}}`

## Purpose
Deletes the “Code review for PR #452” issue captured earlier as `codeReviewKey`.  
This request is designed to **run only on iteration 4** of the Newman run (`pm.info.iteration === 3`) and be skipped otherwise.

## Authentication
- **Basic Auth**
  - Username: `{{username}}`
  - Password (API token): `{{jiraauth}}`

## Variables Used / Set
- **Reads (Collection):** `codeReviewKey`

## Pre-request Script
```javascript
const issueKey = pm.collectionVariables.get("codeReviewKey");

// Only run update on iteration 3 when codeReviewID exists
if (!issueKey || pm.info.iteration !== 3) {
    console.log(
        !issueKey
            ? "Skipping Update Issue because codeReviewKey is missing"
            : `Skipping Update Issue because iteration is ${pm.info.iteration}`
    );
    return pm.execution.skipRequest();
}

console.log(`Running Update Issue with key: ${issueKey} (iteration ${pm.info.iteration})`);
```

## Post-request Script
```javascript
const issueKey = pm.collectionVariables.get("codeReviewKey");

if (pm.info.iteration === 3 && issueKey) {
    pm.test("Code Review Issue deleted successfully (204)", function () {
        pm.response.to.have.status(204);
    });
    console.log(`Code Review issue ${issueKey} deleted successfully (iteration ${pm.info.iteration})`);
} else {
    const reason = !issueKey
        ? "issueKey is missing"
        : `current iteration is ${pm.info.iteration}`;
    console.log(`Skipping delete validation: ${reason}`);
}
```

## Sample Output (Newman)
Captured from a real run. Shows 200 OK, response parsed, variables set, then jump to the next request.

**Case — Runs on iteration 4 (index 3):**

```bash
└ Delete Issue
  ┌
  │ 'Running Update Issue with key: APIQA-4 (iteration 3)'
  └
  DELETE https://<your-domain>.atlassian.net/rest/api/3/issue/APIQA-4 [204 No Content, 1.58kB, 459ms]
  √  Code Review Issue deleted successfully (204)
  ┌
  │ 'Code Review issue APIQA-4 deleted successfully (iteration 3)'
  └
```

**Case — Skipped on other iterations or when key missing:**

```bash
└ Delete Issue
  ┌
  │ 'Skipping Update Issue because codeReviewKey is missing'
  └
```