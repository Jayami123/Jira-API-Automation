# Get the Updated Issue

**Folder:** `3 - Update Issue`  
**Method:** `GET`  
**URL:** `https://<your-domain>.atlassian.net/rest/api/3/issue/{{multiLangIssueKey}}`

**Path Params:**
- `multiLangIssueKey = {{multiLangIssueKey}}`

## Purpose
Fetches the updated issue (the one whose description was changed) and verifies that the description matches the expected ADF text: **“Hi! This is the updated description”**.  
This request is designed to **run only on iteration 2** of the Newman run (`pm.info.iteration === 1`) and be skipped otherwise.

## Authentication
- **Basic Auth**
  - Username: `{{username}}`
  - Password (API token): `{{jiraauth}}`

## Variables Used / Set
- **Reads (Collection):** `multiLangIssueKey`

## Pre-request Script
```javascript
const issueKey = pm.collectionVariables.get("multiLangIssueKey");

// Only run if issueKey exists and iteration is the one you want (iteration 2 => index 1)
if (!issueKey || pm.info.iteration !== 1) {
    console.log(
        !issueKey
            ? "Skipping this request because multiLangIssueKey is missing"
            : `Skipping this request because iteration ${pm.info.iteration} is not the target`
    );
    return pm.execution.skipRequest();
}

// If we reach here, issueKey exists and iteration == 1 → run request
console.log(`Running Get Updated Issue with multiLangIssueKey: ${issueKey} on iteration ${pm.info.iteration}`);
```

## Post-request Script
```javascript
// Post-request Script for Get Updated Issue
const issueKey = pm.collectionVariables.get("multiLangIssueKey");

// Only run test/assertions if issueKey exists and iteration = 2 (index 1)
if (!issueKey || pm.info.iteration !== 1) {
    console.log(
        !issueKey 
            ? "Skipping test because multiLangIssueKey is missing" 
            : `Skipping test because iteration ${pm.info.iteration} is not the target`
    );
    return; // Skip tests
}

// Parse response JSON
const response = pm.response.json();

// Extract description text from ADF
let descriptionText = "";
if (response.fields.description && response.fields.description.content) {
    descriptionText = response.fields.description.content
        .map(block => {
            if (block.content) {
                return block.content.map(item => item.text).join("");
            }
            return "";
        })
        .join("\n");
}

// Log extracted text
console.log("Description:", descriptionText);

// Test: check if description matches expected value
pm.test("Description matches expected text", function () {
    pm.expect(descriptionText).to.eql("Hi! This is the updated description");
});
```

## Sample Output (Newman)
Captured from a real run. Shows 200 OK, response parsed, variables set, then jump to the next request.

**Case — Runs on iteration 2 (index 1):**

```bash
└ Get the updated issue
  ┌
  │ 'Running Get Updated Issue with multiLangIssueKey: APIQA-2 on iteration 1'
  └
  GET https://<your-domain>.atlassian.net/rest/api/3/issue/APIQA-2 [200 OK, 8.26kB, 219ms]
  ┌
  │ 'Description:', 'Hi! This is the updated description'
  └
  √  Description matches expected text

```

**Case — Skipped on other iterations or when key missing:**

```bash
└ Get the updated issue
  ┌
  │ 'Skipping this request because iteration 2 is not the target'
  └
```