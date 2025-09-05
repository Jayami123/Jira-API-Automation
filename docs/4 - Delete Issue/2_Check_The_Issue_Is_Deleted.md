# Check the Issue is Deleted

**Folder:** `4 - Delete Issue`  
**Method:** `GET`  
**URL:** `https://<your-domain>.atlassian.net/rest/api/3/issue/{{codeReviewKey}}`

**Path Params:**
- `codeReviewKey = {{codeReviewKey}}`

## Purpose
Confirms that the **Code review for PR #452** issue was deleted by expecting a **404 Not Found**.  
Runs **only on iteration 4** of the Newman run (`pm.info.iteration === 3`) and skips otherwise.

## Authentication
- **Basic Auth**
  - Username: `{{username}}`
  - Password (API token): `{{jiraauth}}`

## Variables Used / Set
- **Reads (Collection):** `codeReviewKey`

## Pre-request Script
```javascript
const issueKey = pm.collectionVariables.get("codeReviewKey");

// Only run update on iteration 3 when issueKey exists
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

// Only run on iteration 3 when codeReviewKey exists
if (!issueKey || pm.info.iteration !== 3) {
    console.log(
        !issueKey
            ? "Skipping Get Deleted Issue because codeReviewKey is missing"
            : `Skipping Get Deleted Issue because iteration ${pm.info.iteration} is not the target`
    );
    return pm.execution.skipRequest();
}

console.log(`Running Get Deleted Issue with key: ${issueKey} (iteration ${pm.info.iteration})`);

// Post-request Script
if (pm.response.code === 404) {
    console.log(`Issue ${issueKey} not found — deleted successfully.`);
    pm.test("Issue deleted successfully (404)", () => pm.expect(pm.response.code).to.eql(404));
    return; // Skip further parsing
}

// If by any chance the issue still exists, parse safely
let descriptionText = "";
if (pm.response.json().fields?.description?.content) {
    descriptionText = pm.response.json().fields.description.content
        .map(block => block.content?.map(item => item.text).join("") || "")
        .join("\n");
}

console.log("Description:", descriptionText);
pm.test("Description matches expected text (unexpected)", () => 
    pm.expect(descriptionText).to.eql("Hi! This is the updated description")
);
```

## Sample Output (Newman)
Captured from a real run. Shows 200 OK, response parsed, variables set, then jump to the next request.

```bash
└ Check the issue is deleted
  ┌
  │ 'Running Get Deleted Issue with key: APIQA-4 (iteration 3)'
  └
  GET https://<your-domain>.atlassian.net/rest/api/3/issue/APIQA-4 [404 Not Found, 1.46kB, 163ms]
  ┌
  │ 'Running Get Deleted Issue with key: APIQA-4 (iteration 3)'
  │ 'Issue APIQA-4 not found — deleted successfully.'
  └
  √  Issue deleted successfully (404)
```