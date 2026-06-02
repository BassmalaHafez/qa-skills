---
name: create-bug-to-azure
description: Takes a failed test case, crafts a short descriptive Bug summary, builds a structured Azure DevOps description with Preconditions / Steps / Expected Result / Actual Result, and creates the Bug work item via Azure DevOps REST API.
user-invocable: true
---

# Create Bug to Azure DevOps from Failed Test Case

## When to use

- A test case has failed and the user wants to file a Bug in Azure DevOps
- The user pastes a failed test result, error log, or describes what went wrong

## Required inputs

| Input | Notes |
| --- | --- |
| Failed test case | Title, steps, expected vs actual result — paste or describe |
| Azure project | Project name or ID where the bug should be logged |
| Organization URL | e.g., `https://dev.azure.com/myorg` |

## Optional inputs

- Severity override — if user specifies (e.g., Critical, High)
- Priority override — if user specifies (e.g., Highest, High)
- Area Path — for organizing within the project
- Tags — for classification (e.g., `automated-test`, `regression`)
- Assigned to — user name or ID; use Azure DevOps REST API to resolve
- Linked work item key — will be linked via "Relates to" link type

---

## Workflow

```
Task Progress:
- [ ] Gather failed test input and project details
- [ ] Confirm Bug work item type exists in project
- [ ] Fetch linked story/acceptance criteria (if available)
- [ ] Infer severity and priority from AC + bug impact
- [ ] Draft summary and description; present for approval
- [ ] Create Bug work item via Azure DevOps REST API
- [ ] Link to parent/story if requested
- [ ] Report created work item ID and URL to user
```

---

### 1. Gather inputs

Ask for anything missing before proceeding:

1. If no project is given, ask: _"Which Azure DevOps project should I log this bug under?"_
2. If organization URL is not provided, ask: _"What is your Azure DevOps organization URL? (e.g., https://dev.azure.com/myorg)"_
3. If neither test steps nor error output is provided, ask: _"Can you paste the test case steps, expected result, and what actually happened?"_
4. Do **not** ask about fields that can be inferred or are optional.

---

### 2. Confirm Bug work item type

1. Call Azure DevOps Work Item Types API:
   ```
   GET https://dev.azure.com/{organization}/{project}/_apis/wit/workitemtypes?api-version=7.0
   ```
2. Locate the work item type named **"Bug"** (case-insensitive). Store its name.
3. Verify that Bug can be created in this project.

---

### 3. Fetch linked story and extract acceptance criteria

This step runs **before** drafting. Its output feeds directly into severity and priority inference.

1. If the user provides a linked story / parent work item ID, call:
   ```
   GET https://dev.azure.com/{organization}/{project}/_apis/wit/workitems/{id}?api-version=7.0
   ```
2. Extract the following fields from the response:
   - **Acceptance criteria** — look in Description or custom fields containing "acceptance", "AC", or "criteria".
   - **Priority** — read the Priority field of the parent story.
   - **Tags / Area Path** — may signal business criticality (e.g., `payments`, `auth`, `core-flow`).
3. If no story key is provided, skip this step and infer from the bug description alone.
4. Store the extracted AC text for use in severity/priority inference.

> **If the API call fails or the story has no AC:** log a note "AC unavailable — inferring from bug description only" and continue.

---

### 4. Draft the ticket — present before creating

#### 4a. Summary (title)

Write a **short, specific, actionable** summary following this pattern:

```
[Module/Feature] — <Observable failure in plain language>
```

Rules:

- Maximum ~80 characters
- Start with the module or feature name in brackets if identifiable
- Use present tense: _"fails"_, _"returns"_, _"displays"_ — not _"failed"_ or _"returned"_
- Include the key discriminating detail (e.g., specific input value, role, environment)
- Do **not** start with generic words like _"Bug:"_, _"Issue:"_, _"Test:"_

Good examples:

- `[Login] — "Forgot Password" link returns 404 for SSO users`
- `[Cart] — Discount code applies twice when user refreshes checkout page`
- `[API /orders] — POST returns 500 when item quantity is 0`

#### 4b. Infer Severity and Priority

Use **both** the linked story's acceptance criteria (fetched in step 3) **and** the bug description to determine Severity and Priority. Present them to the user as part of the draft review.

##### Step 1 — Read the acceptance criteria signals

Scan the AC text for indicators that raise or lower severity:

| AC signal (look for these phrases or patterns) | Implication |
| --- | --- |
| _"must"_, _"shall"_, _"critical path"_, _"required"_ | Failing this AC → at least **High** |
| _"payment"_, _"checkout"_, _"login"_, _"authentication"_, _"security"_ | Business-critical context → **High** or **Critical** |
| _"should"_, _"expected to"_, _"ideally"_ | Softer requirement → consider **Medium** |
| _"nice to have"_, _"optionally"_, _"cosmetic"_, _"may"_ | Non-blocking → **Low** |
| AC is absent or story has no AC | Fall back to bug-description inference only |

**Also check:**

- Does the **failed AC item** cover the happy/core path, or an edge case?
  - Core / happy path failure → raise severity by one level if not already Critical.
  - Edge case failure → keep or lower by one level.
- Does the **parent story's priority** indicate business urgency?
  - Story priority High/Highest → floor the bug priority at **High** unless severity is Low.
  - Story priority Low → cap the bug priority at **Medium** unless severity is Critical.

##### Step 2 — Apply severity table

##### Severity — how badly does this break the system? (combine AC signals + bug impact)

| Severity | Criteria (match any one) |
| --- | --- |
| **Critical** | System crash / data loss / security vulnerability / complete feature outage with no workaround |
| **High** | Core business flow broken, no reasonable workaround, significant portion of users affected |
| **Medium** | Feature partially works, workaround exists, moderate user impact |
| **Low** | Cosmetic defect, typo, minor UI misalignment, edge-case with negligible user impact |

##### Step 3 — Apply priority table

##### Priority — how urgently must this be fixed? (combine AC signals + story priority)

| Priority | Criteria (match any one) |
| --- | --- |
| **Highest** | Blocks release / blocks other team members' work / production is down right now |
| **High** | Affects a key user-facing flow for many users; should land in the current sprint |
| **Medium** | Notable impact but can be scheduled in the next sprint without serious consequence |
| **Low** | Minor / cosmetic; can be deferred to backlog |

**Decision rules:**

- If Severity = Critical → Priority defaults to **Highest** (unless overridden by user).
- If Severity = High → Priority defaults to **High**.
- If Severity = Medium → Priority defaults to **Medium**.
- If Severity = Low → Priority defaults to **Low**.
- Parent story priority acts as a **floor for Priority** when Severity ≥ High, and as a **cap** when Severity ≤ Medium (see Step 1 above).
- Always let an explicit user override win over the inferred value.
- If there is genuine ambiguity, pick the **lower** severity/priority and explain your reasoning in the draft.

##### Step 4 — Cite your reasoning

In the draft confirmation message, include a one-line rationale, e.g.:

> _Severity: High — AC item #2 ("User must be able to complete checkout") covers the core payment path and the bug fully breaks it. Priority: High — parent story covers critical user flow and affects current sprint._

This gives the user enough context to agree or correct the inference.

---

#### 4c. Description

Use this exact template:

```markdown
## Preconditions

- [List all setup requirements: environment, user role, data state, feature flags, etc.]

## Steps to Reproduce

1. [First action — be specific, include exact values used]
2. [Second action]
3. [Continue until the failure occurs]

## Expected Result

- [What should happen according to requirements or the test case design]

## Actual Result

- [What actually happened — include error messages, status codes, or observable UI behavior verbatim]

## Test Evidence

- Test case: [Name or ID of the failed test case if available]
- Environment: [e.g. Staging, Production, Local — state if unknown]
- Build / Version: [if available]
- Attachment: None

## Additional Context

[Any logs, stack traces, or relevant notes. Remove section if empty.]
```

Fill every section from the user's input. If a section is genuinely unknown, write `Unknown — to be investigated` rather than leaving it blank.

#### 4d. Present draft

Show the complete Summary, Severity, Priority, and Description to the user before creating anything. Ask:

> _"Does this look correct? Should I create the bug in Azure DevOps?"_

Do not create the work item until the user explicitly confirms.

---

### 5. Create the Bug work item

After user confirmation, call Azure DevOps Work Items API:

```
POST https://dev.azure.com/{organization}/{project}/_apis/wit/workitems/$Bug?api-version=7.0

Body:
{
  "op": "add",
  "path": "/fields/System.Title",
  "value": "<drafted summary>"
},
{
  "op": "add",
  "path": "/fields/System.Description",
  "value": "<markdown description>"
},
{
  "op": "add",
  "path": "/fields/Microsoft.VSTS.Common.Severity",
  "value": "<Severity value>"
},
{
  "op": "add",
  "path": "/fields/System.Priority",
  "value": "<Priority value>"
}
```

Capture the returned work item ID and URL.

---

### 6. Link to parent issue (optional)

If the user provides a related story or parent work item ID:

1. Call Azure DevOps Link Types API to find the appropriate link type (prefer "Relates To").
2. Link the new Bug to the parent work item.

---

## Response format

Return a concise confirmation message:

```
Bug created: [BUG-123](https://dev.azure.com/{org}/{project}/_workitems/edit/123)
Summary:  [Login] — "Forgot Password" link returns 404 for SSO users
Severity: High
Priority: High
Linked to: STORY-456 (Relates to)       ← or omit if none
```

Then ask if anything else needs to be updated on the work item.

---

## Safety rules

- **Never create the work item without explicit user confirmation** of the draft.
- Never guess the project — always confirm with the user if ambiguous.
- Never invent steps or failure details not provided by the user.
- Always show the inferred Severity and Priority to the user during draft review — never silently apply them.
- If the inferred severity/priority seems high (Critical/Highest), briefly explain the reasoning so the user can correct it.
