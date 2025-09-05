# Get the List of Notification Schemes

**Folder:** `1 - Project Creation`  
**Method:** `GET`  
**URL:** `https://<your-domain>.atlassian.net/rest/api/3/notificationscheme`

## Purpose
Retrieves available Jira notification schemes and stores the first scheme’s ID (`notificationSchemeId`) as a collection variable for use during project creation.

## Authentication
- **Basic Auth**
  - Username: `{{username}}`
  - Password (API token): `{{jiraauth}}`

## Variables Used / Set
- **Writes (Collection):**
  - `notificationSchemeId` — ID of the first notification scheme returned

## Post-request Script
```javascript
let response = pm.response.json();

let notificationSchemeId = response.values[0].id;
pm.collectionVariables.set("notificationSchemeId", notificationSchemeId);

console.log("Notification Scheme ID:", notificationSchemeId);
```

## Sample Output (Newman)
Captured from a real run. Shows 200 OK and the first notification scheme ID being saved.

```bash
└ Get the list of notification schemes
  GET https://<your-domain>.atlassian.net/rest/api/3/notificationscheme [200 OK, 2.69kB, 264ms]
  ┌
  │ 'Notification Scheme ID:', 10034
  └

```