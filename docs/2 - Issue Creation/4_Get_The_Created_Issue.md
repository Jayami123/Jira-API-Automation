# Get the Created Issue

**Folder:** `2 - Issue Creation`  
**Method:** `GET`  
**URL:** `https://<your-domain>.atlassian.net/rest/api/3/issue/{{createdIssueId}}`

**Path Params:**
- `createdIssueId = {{createdIssueId}}`

## Purpose
Fetches the issue that was just created to validate its fields and capture IDs/keys for later steps.  
Also sets special variables when specific summaries are detected (e.g., **Multi-language support**, **Code review for PR #452**).

## Authentication
- **Basic Auth**
  - Username: `{{username}}`
  - Password (API token): `{{jiraauth}}`

## Variables Used / Set
- **Reads (Collection):**
  - `createdIssueId`
  - `issueTitle`
  - `projectID`
  - `projectKey`
  - `issueID` (expected issue type id)

- **Writes (Collection):**
  - `issueIDCreated` — saved from response
  - `issueIDToDelete` — saved from response (used later)
  - `multiLangIssueKey` — when summary equals **Multi-language support**
  - `codeReviewKey` — when summary equals **Code review for PR #452**

## Post-request Script
```javascript
// Parse response JSON
const response = pm.response.json();
console.log("Summary:", response.fields.summary);

// Only proceed if response.id exists
if (response.id) {
    // Save general issue IDs
    pm.collectionVariables.set("issueIDCreated", response.id);
    pm.collectionVariables.set("issueIDToDelete", response.id);
    console.log("Issue ID saved:", response.id);

    // Save specific issues based on summary
    if (response.fields.summary === "Multi-language support") {
        pm.collectionVariables.set("multiLangIssueKey", response.key);
        console.log("Multi-language support issue Key saved:", response.key);
    }

    if (response.fields.summary === "Code review for PR #452") {
        pm.collectionVariables.set("codeReviewKey", response.key);
        console.log("Code Review issue Key saved:", response.key);
    }
}

// ------------------------
// Tests
// ------------------------

// Status code check
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Summary check
pm.test("Summary matches expected title", function () {
    const expectedTitle = pm.collectionVariables.get("issueTitle");
    pm.expect(response.fields.summary).to.eql(expectedTitle);
});

// Project ID check
pm.test("Project ID matches", function () {
    pm.expect(response.fields.project.id.toString()).to.eql(pm.collectionVariables.get("projectID"));
});

// Project key check
pm.test("Project key matches", function () {
    pm.expect(response.fields.project.key).to.eql(pm.collectionVariables.get("projectKey"));
});

// Issue Type check
pm.test("Issue Type matches", function () {
    pm.expect(response.fields.issuetype.id.toString()).to.eql(pm.collectionVariables.get("issueID"));
});
```

## Sample Output (Newman)
Captured from real runs, across different iterations/summaries.

**Case — Iteration 1**

```bash
└ Get the Created Issue
  GET https://<your-domain>.atlassian.net/rest/api/3/issue/10796 [200 OK, 8.24kB, 297ms]
  ┌
  │ 'Summary:', 'UI alignment issue in dashboard'
  │ 'Issue ID saved:', '10796'
  └
  √  Status code is 200
  √  Summary matches expected title
  2. Project ID matches
  √  Project key matches
  √  Issue Type matches
```

**Case — Iteration 2 (multi-language branch)**

```bash
└ Get the Created Issue
  GET https://<your-domain>.atlassian.net/rest/api/3/issue/10797 [200 OK, 8.28kB, 251ms]
  ┌
  │ 'Summary:', 'Multi-language support'
  │ 'Issue ID saved:', '10797'
  │ 'Multi-language support issue Key saved:', 'APIQA-2'
  └
  √  Status code is 200
  √  Summary matches expected title
  √  Project ID matches
  √  Project key matches
  √  Issue Type matches
```

**Case — Iteration 3**

```bash
└ Get the Created Issue
  GET https://<your-domain>.atlassian.net/rest/api/3/issue/10798 [200 OK, 8.24kB, 200ms]
  ┌
  │ 'Summary:', 'Setup staging environment'
  │ 'Issue ID saved:', '10798'
  └
  √  Status code is 200
  √  Summary matches expected title
  √  Project ID matches
  √  Project key matches
  √  Issue Type matches
```

**Case — Iteration 4 (code review branch)**

```bash
└ Get the Created Issue
  GET https://<your-domain>.atlassian.net/rest/api/3/issue/10799 [200 OK, 8.24kB, 220ms]
  ┌
  │ 'Summary:', 'Code review for PR #452'
  │ 'Issue ID saved:', '10799'
  │ 'Code Review issue Key saved:', 'APIQA-4'
  └
  √  Status code is 200
  √  Summary matches expected title
  √  Project ID matches
  √  Project key matches
  √  Issue Type matches
```
