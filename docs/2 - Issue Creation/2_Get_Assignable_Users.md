# Get Assignable Users

**Folder:** `2 - Issue Creation`  
**Method:** `GET`  
**URL:** `https://<your-domain>.atlassian.net/rest/api/3/user/assignable/search?project={{projectID}}`

**Query Params:**
- `project = {{projectID}}`

## Purpose
Fetches users who can be assigned issues in the target project and saves the first user’s `accountId` for downstream requests (e.g., issue creation).

## Authentication
- **Basic Auth**
  - Username: `{{username}}`
  - Password (API token): `{{jiraauth}}`

## Variables Used / Set
- **Reads (Collection):** `projectID`
- **Writes (Collection):** `accountId` — the first assignable user’s account ID

## Post-request Script
```javascript
const response = pm.response.json();

// Check if response is an array and not empty
if (Array.isArray(response) && response.length > 0) {
    const accountId = response[0].accountId;
    console.log("Account ID:", accountId);
    
    // Save to collection variable
    pm.collectionVariables.set('accountId', accountId);
} else {
    console.warn("No users found in the response.");
}
```

## Sample Output (Newman)
Captured from a real run. Shows 200 OK, response parsed, variables set, then jump to the next request.

```bash 
└ Get Assignable Users
  GET https://<your-domain>.atlassian.net/rest/api/3/user/assignable/search?project=10231 [200 OK, 2.65kB, 390ms]
  ┌
  │ 'Account ID:', '712020:07e4efde-27aa-409c-bab3-300f05b2bdae'
  └

```