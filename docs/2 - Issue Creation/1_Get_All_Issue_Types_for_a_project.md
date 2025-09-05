# Get All Issue Types for a Project

**Folder:** `2 - Issue Creation`  
**Method:** `GET`  
**URL:** `https://<your-domain>.atlassian.net/rest/api/3/issuetype/project?projectId={{projectID}}`

**Query Params:**
- `projectId = {{projectID}}`

## Purpose
Retrieves all available issue types for a given Jira project.  
If the `issueName` variable matches an existing issue type, its ID is saved to `issueID`.  
Additionally checks whether the project exists by name and, if found, updates the `projectID` and skips ahead to avoid duplicate project creation.

## Authentication
- **Basic Auth**
  - Username: `{{username}}`
  - Password (API token): `{{jiraauth}}`

## Variables Used / Set
- **Reads (Collection/Environment):**
  - `projectID`
  - `issueName`
  - `projectName`

- **Writes (Collection/Environment):**
  - `issueID` — ID of the issue type matching `issueName`
  - `projectID` (environment) — overwritten if project is found by name

## Post-request Script
```javascript
const response = pm.response.json();   // Correct spelling

// Extract all "name" values
const desiredValues = response.map(value => value.name);

// Find index of the issue type that matches the variable "issueName"
const index = desiredValues.indexOf(pm.variables.get("issueName"));

if (index !== -1) {
    pm.collectionVariables.set("issueID", response[index].id);
} else {
    console.log("Issue type not found:", pm.variables.get("issueName"));
}

// Get project name from environment
const projectName = pm.environment.get("projectName");

let jsonData;
try {
    jsonData = pm.response.json();
} catch (e) {
    console.log("Response is not JSON:", e);
    pm.test("Response must be JSON", function () {
        pm.expect.fail("Invalid response format");
    });
    return;
}

// Look for project by name
let foundProject = null;
if (jsonData.values && jsonData.values.length > 0) {
    foundProject = jsonData.values.find(p => p.name.toLowerCase() === projectName.toLowerCase());
}

if (foundProject) {
    // ✅ Project exists
    console.log(`Project already exists with ID: ${foundProject.id}`);
    pm.environment.set("projectID", foundProject.id);

    // Skip Create Project, jump to Get All Issue Types
    pm.execution.setNextRequest("Get All Issue Types for a project");
} else {
    // ❌ Project not found → flow continues to Create Project request
    console.log("Project not found. Proceeding to create it...");
}
```

## Sample Output (Newman)
Captured from a real run. Shows 200 OK, response parsed, variables set, then jump to the next request.

**Case 1 — Project not found (flow continues to project creation):**

```bash
└ Get All Issue Types for a project
  GET https://<your-domain>.atlassian.net/rest/api/3/issuetype/project?projectId=10231 [200 OK, 3.75kB, 206ms]
  ┌
  │ 'Project not found. Proceeding to create it...'
  └

```

**Case 2 — Project already exists (project ID confirmed and flow skips creation):**

```bash
└ Get All Issue Types for a project
  GET https://<your-domain>.atlassian.net/rest/api/3/issuetype/project?projectId=10231 [200 OK, 3.75kB, 147ms]
  ┌
  │ 'Project already exists with ID: 10231'
  └

````