# Get Project Types

**Folder:** `1 - Project Creation`  
**Method:** `GET`  
**URL:** `https://<your-domain>.atlassian.net/rest/api/3/project/type`

## Purpose
Retrieves the available Jira project types and saves the `"software"` type key (`projectTypeKey`) as a collection variable for use when creating a project.

## Authentication
- **Basic Auth**
  - Username: `{{username}}`
  - Password (API token): `{{jiraauth}}`

## Variables Used / Set
- **Writes (Collection):**
  - `projectTypeKey` — key of the `"software"` project type if found

## Post-request Script
```javascript
let response = pm.response.json();

let projectType;
if (Array.isArray(response)) {
    projectType = response.find(item => item.key === "software");
} else if (response.key === "software") {
    projectType = response;
}

if (projectType) {
    pm.collectionVariables.set("projectTypeKey", projectType.key);
    console.log("ProjectTypeKey:", projectType.key);
} else {
    console.error("Software project type not found in response!");
}
```

## Sample Output (Newman)
Captured from a real run. Shows 200 OK, response parsed, variables set, then jump to the next request.

```bash
└ Get Project Types
  GET https://<your-domain>.atlassian.net/rest/api/3/project/type [200 OK, 7.33kB, 262ms]
  ┌
  │ 'ProjectTypeKey:', 'software'
  └

```

