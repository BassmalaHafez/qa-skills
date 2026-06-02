---
name: generate-test-cases-from-azure
description: Fetches an Azure DevOps User Story via REST API, extracts acceptance criteria, and generates traceable manual and Azure-ready test cases with optional code/wiki evidence.
user-invocable: true
---

# Generate Test Cases from Azure DevOps User Story

## When to use

- User provides an Azure DevOps work item ID (e.g., `PROJ-123`) or story URL
- User asks for test cases to be generated from a user story
- User wants to publish test cases back to Azure Test Plans

## Required inputs

| Input | Notes |
| --- | --- |
| Work item ID or URL | e.g., `PROJ-123` or `https://dev.azure.com/org/project/_workitems/edit/123` |
| Organization URL | e.g., `https://dev.azure.com/myorg` (can be inferred from URL) |
| Project name | e.g., `MyProject` |

## Optional inputs

- Test case format — Manual / Automation-Ready / Both
- Publishing preference — Save locally / Push to Azure Test Plans
- Coverage level — Happy path only / Happy path + edge cases / Happy path + edge cases + error scenarios

---

## Workflow

```
Task Progress:
- [ ] Parse work item ID and organization details
- [ ] Fetch story from Azure DevOps
- [ ] Extract title, description, and acceptance criteria
- [ ] Generate test cases (manual + automation-ready)
- [ ] Present for review and approval
- [ ] Publish to Azure Test Plans (if requested)
- [ ] Report test case IDs and links
```

---

### 1. Parse input and resolve details

If the user provides a URL:
```
https://dev.azure.com/{org}/{project}/_workitems/edit/{id}
```

Extract `{org}`, `{project}`, and `{id}`. If only an ID is provided, ask for the organization URL.

---

### 2. Fetch the User Story

Call Azure DevOps Work Items API:
```
GET https://dev.azure.com/{organization}/{project}/_apis/wit/workitems/{id}?api-version=7.0&$expand=all
```

Extract:
- **Title** — Story summary
- **Description** — Story narrative and context
- **Acceptance Criteria** — Look in Description field or custom field (e.g., "Acceptance Criteria")
- **Priority** — Business urgency
- **Area Path** — Feature/module area
- **Tags** — For classification (e.g., `critical-path`, `payments`)
- **Linked work items** — Related stories, dependencies, or acceptance tests

> If AC is not in a dedicated field, scan the Description for a section titled "Acceptance Criteria", "AC", or "Definition of Done".

---

### 3. Parse Acceptance Criteria

Extract each AC as a separate item:

```
AC 1: User can log in with valid email and password
AC 2: User sees "Invalid credentials" message on failed login
AC 3: Account locks after 5 failed attempts
AC 4: "Forgot Password" link sends reset email within 2 minutes
```

---

### 4. Generate Manual Test Cases

For each AC, generate a manual test case:

```markdown
## Test Case: [Module] - [AC description]

**Preconditions:**
- [Setup: valid test data, user roles, environment state]

**Steps:**
1. [Action 1]
2. [Action 2]
3. [Continue until AC is verified]

**Expected Result:**
- [Observable outcome that matches AC]

**Acceptance Criteria Link:**
- AC #1: [User can log in with valid email and password]

**Data Requirements:**
- [Test data: usernames, passwords, files, etc.]

**Environment:**
- [Staging / Production / Local]
```

---

### 5. Generate Automation-Ready Test Cases

For each manual test case, also generate a Cypress/Playwright-ready variant:

```markdown
## Automation Script: [Module] - [AC description]

**Technology:** Cypress / Playwright  
**POM Layer Ownership:**
- Locators: [fixture name]
- Navigation: [page object name]
- Assertions: [page object method]

**Pseudocode:**
```
Given user navigates to login page
When user enters valid email: `{{email}}`
And user enters valid password: `{{password}}`
And user clicks "Sign In" button
Then user should see dashboard
And browser should store auth token in localStorage
```

**Fixture Dependencies:**
- Test data: fixtures/users.json → valid user accounts
- Selectors: fixtures/selectors.json → login form elements

**Risk Flags:**
- [Any third-party integrations, timing issues, or environment-specific behaviors]
```

---

### 6. Present for review

Show the user:
1. **Story summary** — Title, description, priority
2. **Extracted AC** — Numbered list
3. **Generated manual test cases** — 1 per AC
4. **Generated automation scripts** — Pseudocode variants
5. **Coverage summary** — How many ACs are covered

Ask:
> _"Do these test cases capture the story's intent? Should I publish them to Azure Test Plans?"_

---

### 7. Publish to Azure Test Plans (optional)

If the user confirms:

1. Create Test Case work items via Azure DevOps REST API:
   ```
   POST https://dev.azure.com/{organization}/{project}/_apis/wit/workitems/$Test%20Case?api-version=7.0
   
   Body:
   {
     "op": "add",
     "path": "/fields/System.Title",
     "value": "[Module] - [AC description]"
   },
   {
     "op": "add",
     "path": "/fields/System.Description",
     "value": "<markdown test steps>"
   },
   {
     "op": "add",
     "path": "/fields/Microsoft.VSTS.TCM.Steps",
     "value": "<steps in ADF format>"
   }
   ```

2. Link each test case to the parent story:
   ```
   Relationship: "Tested by" (Test Case → Story)
   ```

3. Capture returned Test Case IDs and URLs.

---

## Response format

```
✅ Test Cases Generated from STORY-123
📖 Story: "User authentication with SSO integration"
📋 Acceptance Criteria: 4 items extracted
✏️  Manual Test Cases: 4 created
🤖 Automation Scripts: 4 generated (Cypress-ready)

**Generated Test Cases:**
1. TC-1: [TC-1234] - User can log in with valid credentials
2. TC-2: [TC-1235] - User sees error on invalid credentials
3. TC-3: [TC-1236] - Account locks after 5 failed attempts
4. TC-4: [TC-1237] - Password reset email arrives within 2 minutes

**Links:**
- [View in Azure](https://dev.azure.com/{org}/{project}/_testPlans/...)
- [Story](https://dev.azure.com/{org}/{project}/_workitems/edit/123)
```

---

## Safety rules

- **Never publish to Azure without explicit user confirmation.**
- **Always extract AC verbatim** from the story — do not paraphrase or add your own interpretation.
- **Always link test cases back to the parent story** so traceability is preserved.
- **If AC is ambiguous or missing**, ask the user for clarification before generating test cases.
- **Always show the generated test cases** to the user before publishing — never silently create work items.
