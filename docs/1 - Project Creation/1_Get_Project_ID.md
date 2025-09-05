# Get Project ID

**Folder:** `1 - Project Creation`  
**Method:** `GET`  
**URL:** `https://<your-domain>.atlassian.net/rest/api/3/project/search`

## Purpose
Lists Jira projects and checks whether **JIRA End to End Testing** already exists.  
If found, saves `projectID` and `projectKey` as **collection variables** and jumps the run to **“Get All Issue Types for a project.”**  
If not found, sets `projectExists='false'` so the flow proceeds to project creation.

## Authentication
- **Basic Auth**
  - Username: `{{username}}` (e.g., your Atlassian email)
  - Password (API token): `{{jiraauth}}`  
> Keep secrets in Postman variables; never commit real credentials.

## Variables Used / Set
- **Writes (Collection):**
  - `projectID` — the existing project’s ID (if found)  
  - `projectKey` — the existing project’s key (if found)  
  - `projectExists` — `'true'` if found, `'false'` otherwise  

## Tests (Post-request Script)
```javascript
try {
    const response = pm.response.json();
    console.log("Response:", response);

    const projectDetails = response.values || [];
    const projectNameList = projectDetails.map(p => p.name);
    console.log("Project names:", projectNameList);

    const projectIndex = projectNameList.indexOf('JIRA End to End Testing');

    if (projectIndex === -1) {
        console.log("Project not found. Proceeding with next steps.");
        pm.collectionVariables.set('projectExists', 'false'); // Project does not exist
    } else {
        const selectedProject = projectDetails[projectIndex];
        const projectId = selectedProject.id;
        const projectKey = selectedProject.key;

        // Save variables
        pm.collectionVariables.set('projectID', projectId);
        pm.collectionVariables.set('projectKey', projectKey);
        pm.collectionVariables.set('projectExists', 'true'); // Project exists

        console.log("Project found:", selectedProject.name);
        console.log("Project ID:", projectId);
        console.log("Project Key:", projectKey);

        // Skip requests up to "Get All Issue Types for a project"
        pm.execution.setNextRequest("Get All Issue Types for a project");
    }

} catch (err) {
    console.error("Error in test script:", err.message);
    pm.collectionVariables.set('projectExists', 'false'); // Safe fallback
}

```

## Sample Output (Newman)
Captured from a real run. Shows 200 OK, response parsed, variables set, then jump to the next request.
```bash
┌
│ 'Response:', {
│   self: 'https://<your-domain>.atlassian.net/rest/api/3/project/search?maxResults=50&startAt=0'
│ ,
│   maxResults: 50,
│   startAt: 0,
│   total: 5,
│   isLast: true,
│   values: [
│     {
│       expand: 'description,lead,issueTypes,url,projectKeys,permissions,insight',
│       self: 'https://<your-domain>.atlassian.net/rest/api/3/project/10202',
│       id: '10202',
│       key: 'APIQA',
│       name: 'JIRA End to End Testing',
│       avatarUrls: {
│         '48x48': 'https://<your-domain>.atlassian.net/rest/api/3/universal_avatar/view/type/project/avatar/10200',
│         '24x24': 'https://<your-domain>.atlassian.net/rest/api/3/universal_avatar/view/type/project/avatar/10200?size=small',
│         '16x16': 'https://<your-domain>.atlassian.net/rest/api/3/universal_avatar/view/type/project/avatar/10200?size=xsmall',
│         '32x32': 'https://<your-domain>.atlassian.net/rest/api/3/universal_avatar/view/type/project/avatar/10200?size=medium'
│       },
│       projectTypeKey: 'software',
│       simplified: true,
│       style: 'next-gen',
│       isPrivate: false,
│       properties: {},
│       entityId: '0453c3a9-be82-43c8-bd24-1cd6fdc76f82',
│       uuid: '0453c3a9-be82-43c8-bd24-1cd6fdc76f82'
│     },
│     { id: '10066', key: 'AT',  name: 'API Testing',           ... },
│     { id: '10033', key: 'ATP', name: 'API Testing Postman',   ... },
│     { id: '10001', key: 'LEARNJIRA', name: '(Learn) Jira Premium benefits in 5 min 👋', ... },
│     { id: '10000', key: 'SCRUM', name: 'Testing Project',     ... }
│   ]
│ }
│ 'Project names:', [
│   'JIRA End to End Testing',
│   'API Testing',
│   'API Testing Postman',
│   '(Learn) Jira Premium benefits in 5 min 👋',
│   'Testing Project'
│ ]
│ 'Project found:', 'JIRA End to End Testing'
│ 'Project ID:', '10202'
│ 'Project Key:', 'APIQA'
└

Attempting to set next request to Get All Issue Types for a project
```