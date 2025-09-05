# Get the Created Project

**Folder:** `1 - Project Creation`  
**Method:** `GET`  
**URL:** `https://<your-domain>.atlassian.net/rest/api/3/project/{{projectID}}`

## Purpose
Fetches details of the newly created Jira project to validate its creation and confirm it matches the expected values. Also saves the created `issueIDCreated` to collection variables for subsequent steps.

## Authentication
- **Basic Auth**
  - Username: `{{username}}`
  - Password (API token): `{{jiraauth}}`

## Variables Used / Set
- **Reads (Collection):**
  - `projectID`  
  - `projectKey`  
  - `issueTitle`  
  - `issueID`  

- **Writes (Collection):**
  - `issueIDCreated` — ID of the created issue  

## Post-request Script
```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

const response = pm.response.json();
console.log("Summary:", response.fields.summary);

// Save the created Issue ID to collection variable
if (response.id) {
    pm.collectionVariables.set("issueIDCreated", response.id);
    console.log("Issue ID saved:", response.id);
}

// Status code check
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Field validations
pm.test("Summary match", function () {
    pm.expect(response.fields.summary).to.eql(pm.collectionVariables.get("issueTitle"));
});

pm.test("projectID match", function () {
    pm.expect(response.fields.project.id).to.eql(pm.collectionVariables.get("projectID").toString());
});

pm.test("projectKey match", function () {
    pm.expect(response.fields.project.key).to.eql(pm.collectionVariables.get("projectKey"));
});

pm.test("IssueType match", function () {
    pm.expect(response.fields.issuetype.id).to.eql(pm.collectionVariables.get("issueID"));
});
```

## Sample Output (Newman)
Captured from a real run. Shows 200 OK, response parsed, variables set, then jump to the next request.

```bash
└ Get the created project
  GET https://<your-domain>.atlassian.net/rest/api/3/project/10231 [200 OK, 6.03kB, 161ms]
  √  Status code is 200

```