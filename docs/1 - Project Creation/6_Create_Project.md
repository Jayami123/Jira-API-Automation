# Create Project

**Folder:** `1 - Project Creation`  
**Method:** `POST`  
**URL:** `https://<your-domain>.atlassian.net/rest/api/3/project`

## Purpose
Creates a new Jira project with the provided details, linking it to the correct **permission scheme**, **notification scheme**, and **lead account ID**. Saves the new project’s `key` and `id` as collection variables for downstream requests.

## Authentication
- **Basic Auth**
  - Username: `{{username}}`
  - Password (API token): `{{jiraauth}}`

## Request Body
```json
{
  "assigneeType": "PROJECT_LEAD",
  "avatarId": 10200,
  "description": "This project automates the end-to-end process of issue management in Jira, including project creation, issue creation via CSV input, workflow transitions, and reporting. It demonstrates the ability to programmatically configure Jira projects, populate custom fields, and generate structured reports, simulating a real-world QA automation workflow.",
  "issueSecurityScheme": 10001,
  "key": "APIQA",
  "leadAccountId": "{{leadAccountId}}",
  "name": "JIRA End to End Testing",
  "notificationScheme": {{notificationSchemeId}},
  "permissionScheme": {{permissionSchemeId}},
  "projectTemplateKey": "com.pyxis.greenhopper.jira:gh-simplified-agility-scrum",
  "projectTypeKey": "{{projectTypeKey}}"
}
```
## Variables Used / Set
- **Reads (Collection):**
  - `leadAccountId`
  - `notificationSchemeId`
  - `permissionSchemeId`
  - `projectTypeKey`

- **Writes (Collection):**
  - `projectKey` — key of the created project  
  - `projectID` — ID of the created project

## Post-request Script
```javascript
let response = pm.response.json();

if (response.key && response.id) {
    pm.collectionVariables.set("projectKey", response.key);
    pm.collectionVariables.set("projectID", response.id);

    console.log("Project Key:", response.key);
    console.log("Project ID:", response.id);
} else {
    console.error("Project key or id not found in response!");
}

pm.test("Status code is 201", function () {
    pm.response.to.have.status(201);
});

```

## Sample Output (Newman)
Captured from a real run. Shows 200 OK, response parsed, variables set, then jump to the next request.

```bash
└ Create project
  POST https://<your-domain>.atlassian.net/rest/api/3/project [201 Created, 1.79kB, 8.1s]
  ┌
  │ 'Project Key:', 'APIQA'
  │ 'Project ID:', 10231
  └
  √  Status code is 201
```