# Create an Issue

**Folder:** `2 - Issue Creation`  
**Method:** `POST`  
**URL:** `https://<your-domain>.atlassian.net/rest/api/3/issue`

## Purpose
Creates a Jira issue in the target project using ADF for the description. Stores the created issue’s `id` and `key` for downstream validations and conditional flows. Also captures specific iterations for update/delete scenarios when running with Newman.

## Authentication
- **Basic Auth**
  - Username: `{{username}}`
  - Password (API token): `{{jiraauth}}`

## Request Body
```json
{
  "fields": {
    "summary": "{{issueTitle}}",
    "description": {
      "type": "doc",
      "version": 1,
      "content": [
        {
          "type": "paragraph",
          "content": [
            { "type": "text", "text": "{{description}}" }
          ]
        }
      ]
    },
    "issuetype": { "id": "{{issueID}}" },
    "project": { "id": "{{projectID}}" }
  }
}
```

## Variables Used / Set

- **Reads (Collection):**
  - `issueTitle`
  - `description`
  - `issueID`
  - `projectID`

- **Writes (Collection):**
  - `createdIssueId`
  - `createdIssueKey`
  - `issueIDToUpdate` (only on Newman **iteration 2** → `pm.info.iteration === 1`)
  - `issueKeyToUpdate` (only on Newman **iteration 2**)
  - `issueIDToDelete` (only on Newman **iteration 4** → `pm.info.iteration === 3`)
  - `issueKeyToDelete` (only on Newman **iteration 4**)


## Pre-request Script
```javascript
pm.collectionVariables.set("issueTitle", pm.iterationData.get("issueTitle"));
```

## Post-request Script
```javascript
let response = pm.response.json();

// Save the most recently created issue
pm.collectionVariables.set("createdIssueId", response.id);
pm.collectionVariables.set("createdIssueKey", response.key);

console.log("Created Issue ID:", response.id);
console.log("Created Issue Key:", response.key);

// Test: status code 201
pm.test("Status code is 201", function () {
    pm.response.to.have.status(201);
});

// Log collection variables
console.log("Created Issue ID (collection var):", pm.collectionVariables.get("createdIssueId"));
console.log("Created Issue Key (collection var):", pm.collectionVariables.get("createdIssueKey"));

// ------------------------
// Save issue specifically for 2nd iteration
// ------------------------
if (pm.info.iteration === 1 && response.id) {
    pm.collectionVariables.set("issueIDToUpdate", response.id);
    pm.collectionVariables.set("issueKeyToUpdate", response.key);
    console.log("Saved 2nd iteration issue ID:", response.id);
    console.log("Saved 2nd iteration issue Key:", response.key);
}

// ------------------------
// Save issue specifically for 4th iteration
// ------------------------
if (pm.info.iteration === 3 && response.id) {
    pm.collectionVariables.set("issueIDToDelete", response.id);
    pm.collectionVariables.set("issueKeyToDelete", response.key); // renamed for consistency
    console.log("Saved 4th iteration issue ID (to delete):", response.id);
    console.log("Saved 4th iteration issue Key (to delete):", response.key);
}
```

## Sample Output (Newman)
Captured from a real run. Shows 200 OK, response parsed, variables set, then jump to the next request.

**Case — Iteration 1 (baseline create):**

```bash
└ Create an Issue
  POST https://<your-domain>.atlassian.net/rest/api/3/issue [201 Created, 1.71kB, 902ms]
  ┌
  │ 'Created Issue ID:', '10796'
  │ 'Created Issue Key:', 'APIQA-1'
  └
  √  Status code is 201
  ┌
  │ 'Created Issue ID (collection var):', '10796'
  │ 'Created Issue Key (collection var):', 'APIQA-1'
  └
```

**Case — Iteration 2 (save for update):**

```bash
└ Create an Issue
  POST https://<your-domain>.atlassian.net/rest/api/3/issue [201 Created, 1.71kB, 649ms]
  ┌
  │ 'Created Issue ID:', '10797'
  │ 'Created Issue Key:', 'APIQA-2'
  └
  √  Status code is 201
  ┌
  │ 'Created Issue ID (collection var):', '10797'
  │ 'Created Issue Key (collection var):', 'APIQA-2'
  │ 'Saved 2nd iteration issue ID:', '10797'
  │ 'Saved 2nd iteration issue Key:', 'APIQA-2'
  └
```

**Case — Iteration 3 (baseline create, no special save):**

```bash
└ Create an Issue
  POST https://<your-domain>.atlassian.net/rest/api/3/issue [201 Created, 1.71kB, 697ms]
  ┌
  │ 'Created Issue ID:', '10798'
  │ 'Created Issue Key:', 'APIQA-3'
  └
  √  Status code is 201
```

**Case — Iteration 4 (save for delete):**

```bash
└ Create an Issue
  POST https://<your-domain>.atlassian.net/rest/api/3/issue [201 Created, 1.71kB, 586ms]
  ┌
  │ 'Created Issue ID:', '10799'
  │ 'Created Issue Key:', 'APIQA-4'
  └
  √  Status code is 201
  ┌
  │ 'Created Issue ID (collection var):', '10799'
  │ 'Created Issue Key (collection var):', 'APIQA-4'
  │ 'Saved 4th iteration issue ID (to delete):', '10799'
  │ 'Saved 4th iteration issue Key (to delete):', 'APIQA-4'
  └
```