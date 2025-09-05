# Get the List of Permission Schemes

**Folder:** `1 - Project Creation`  
**Method:** `GET`  
**URL:** `https://<your-domain>.atlassian.net/rest/api/3/permissionscheme`

## Purpose
Retrieves available Jira permission schemes and stores the first scheme’s ID (`permissionSchemeId`) as a collection variable for use during project creation. Also sets a `skipProjectCreation` flag based on whether a project already exists.

## Authentication
- **Basic Auth**
  - Username: `{{username}}`
  - Password (API token): `{{jiraauth}}`

## Variables Used / Set
- **Reads (Collection):** `projectID`
- **Writes (Collection):**
  - `skipProjectCreation` — `"true"` if `projectID` already exists, otherwise `"false"`
  - `permissionSchemeId` — ID of the first permission scheme returned

## Pre-request Script
```javascript
// Fetch projectID from collection variable
let projectId = pm.collectionVariables.get("projectID");

// If projectID exists, skip project creation
if (projectId) {
    console.log(`Project already exists with ID: ${projectId}. Skipping project creation requests.`);
    pm.collectionVariables.set("skipProjectCreation", "true");
} else {
    console.log("No projectID found. Proceeding with project creation.");
    pm.collectionVariables.set("skipProjectCreation", "false");
}
```
## Post-request Script
```javascript

let response = pm.response.json();

let permissionSchemeId = response.permissionSchemes[0].id;
pm.collectionVariables.set("permissionSchemeId", permissionSchemeId);

console.log("Permission Scheme ID:", permissionSchemeId);
```

## Sample Output (Newman)
Captured from a real run. Shows 200 OK, response parsed, variables set, then jump to the next request.
```bash
└ Get the list of permission schemes
  ┌
  │ 'No projectID found. Proceeding with project creation.'
  └
  GET https://<your-domain>.atlassian.net/rest/api/3/permissionscheme [200 OK, 123.67kB, 275ms]
  ┌
  │ 'Permission Scheme ID:', 10066
  └
```