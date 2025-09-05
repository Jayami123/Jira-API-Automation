# Get Project Lead Account ID

**Folder:** `1 - Project Creation`  
**Method:** `GET`  
**URL:** `https://<your-domain>.atlassian.net/rest/api/3/users/search`

**Query Params:**
- `query = youremail@gmail.com`

## Purpose
Looks up the Jira user by email and saves their `accountId` as the `leadAccountId` collection variable. This ID will later be used when creating a new project.

## Authentication
- **Basic Auth**
  - Username: `{{username}}`
  - Password (API token): `{{jiraauth}}`

## Variables Used / Set
- **Writes (Collection):**
  - `leadAccountId` — account ID of the Atlassian user found

## Post-request Script
```javascript
let response = pm.response.json();

let user = response.find(u => u.accountType === "atlassian");

if (user) {
    pm.collectionVariables.set("leadAccountId", user.accountId);
    console.log("Saved leadAccountId:", user.accountId);
} else {
    console.error("No Atlassian user found!");
}
```

## Sample Output (Newman)
Captured from a real run. Shows 200 OK, response parsed, variables set, then jump to the next request.

```bash
└ Get Project Lead Account ID
  GET https://<your-domain>.atlassian.net/rest/api/3/users/search?query=jayamichamoda1104@gmail.com [200 OK, 13.75kB, 278ms]
  ┌
  │ 'Saved leadAccountId:', '712020:07e4efde-27aa-409c-bab3-300f05b2bdae'
  └

```