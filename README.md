# QA Skills - Azure DevOps Edition

A collection of AI agent skills and custom agents for QA automation workflows integrated with **Azure DevOps** — compatible with **GitHub Copilot (VS Code)**, **Claude Code**, and **Cursor AI**.

---

## Repository Structure

```
qa-skills/
├── CLAUDE.md                              # Knowledge wiki configuration for Claude Code
├── README.md                              # This file
│
├── agents/                                # GitHub Copilot agent definitions (VS Code)
│   ├── cypress-automation.agent.md
│   └── QA-Reviewer.agent.md
│
└── skills/                                # Shared skill definitions (VS Code / Claude Code / Cursor)
    ├── ai-rca-manager/
    │   └── SKILL.md
    ├── cypress-automation-framework/
    │   ├── SKILL.md
    │   ├── references/
    │   │   ├── clean-code-standards.md
    │   │   ├── framework-structure.md
    │   │   ├── locator-extraction.md
    │   │   └── manual-test-case-conversion.md
    │   └── scripts/
    │       ├── extract-cypress-locators.cy.js
    │       └── extract-cypress-locators.js
    ├── create-bug-to-azure/
    │   └── SKILL.md
    └── generate-test-cases-from-azure/
        └── SKILL.md
```

---

## Agents

### `Cypress Automation`

**File:** [agents/cypress-automation.agent.md](agents/cypress-automation.agent.md)

Cypress automation specialist agent. Handles converting manual test cases to Cypress automation, and adding, updating, reviewing, refactoring, or debugging Cypress specs, page objects, fixtures, commands, and flaky test diagnostics.

**Triggers:** "manual test case to automation", "Cypress test", "page object", "fixture update", "support command", "flaky E2E", "email validation", "locator extraction", "automation framework cleanup"

---

### `QA-Reviewer`

**File:** [agents/QA-Reviewer.agent.md](agents/QA-Reviewer.agent.md)

Expert review agent for test automation. Reviews locator stability, POM integrity, wait strategies, and data management. Provides constructive feedback with "why" and actionable improvement recommendations.

---

## Skills (`skills/`)

### `ai-rca-manager`

**File:** [skills/ai-rca-manager/SKILL.md](skills/ai-rca-manager/SKILL.md)

AI specialist in SRE and Lean Six Sigma Root Cause Analysis. Applies RCA frameworks (5 Whys, Fishbone/Ishikawa, Barrier Analysis, FMEA). Conducts blameless post-mortems and generates preventive actions.

**Triggers:** "RCA", "root cause", "post-mortem", "incident report", "blameless report", "why did this break", "analyze this bug", or an Azure DevOps Bug/Incident ID.

---

### `cypress-automation-framework`

**File:** [skills/cypress-automation-framework/SKILL.md](skills/cypress-automation-framework/SKILL.md)

Consolidated Cypress framework skill. Covers the full automation lifecycle — converting manual test cases, adding/updating/refactoring specs, extending page objects, managing fixtures and support commands, and debugging flaky tests.

| Reference | Purpose |
| --- | --- |
| [clean-code-standards.md](skills/cypress-automation-framework/references/clean-code-standards.md) | Code quality rules and examples for specs, POMs, commands, and fixtures |
| [framework-structure.md](skills/cypress-automation-framework/references/framework-structure.md) | Folder layout, layer ownership, and naming conventions |
| [locator-extraction.md](skills/cypress-automation-framework/references/locator-extraction.md) | Workflow and script usage for live-site selector discovery |
| [manual-test-case-conversion.md](skills/cypress-automation-framework/references/manual-test-case-conversion.md) | Step-by-step process for converting manual test cases to Cypress automation |

**Triggers:** "Cypress test", "page object", "fixture update", "support command", "flaky E2E", "email validation", "locator extraction", "automation framework cleanup"

---

### `create-bug-to-azure`

**File:** [skills/create-bug-to-azure/SKILL.md](skills/create-bug-to-azure/SKILL.md)

Takes a failed test case, crafts a Bug work item in Azure DevOps with structured description, inferred severity/priority from acceptance criteria, and links to parent stories and test cases.

**Triggers:** A test failure pasted or described by the user, "log this bug", "create an Azure bug", "file a defect", or any failed test result with an Azure project identifier.

---

### `generate-test-cases-from-azure`

**File:** [skills/generate-test-cases-from-azure/SKILL.md](skills/generate-test-cases-from-azure/SKILL.md)

Fetches an Azure DevOps User Story via REST API, extracts acceptance criteria, and generates traceable manual and Azure-ready test cases with optional code/wiki evidence.

**Triggers:** Providing an Azure work item ID, story URL, asking for test cases from a story, or wanting to publish cases back to Azure DevOps.

---

## Installation

**Option 1 — via npx (recommended):**

```bash
npx skills add BassmalaHafez/qa-skills
```

**Option 2 — manually:**

### GitHub Copilot (VS Code)

Copy the `skills/` and `agents/` folders into your project root. VS Code will auto-discover `.agent.md` files and skill `SKILL.md` files when using GitHub Copilot Chat in Agent mode.

### Claude Code

Copy each skill folder into your project under `.claude/skills/<skill-name>/` and ensure each folder contains a `SKILL.md` file. Restart the chat session.

### Cursor AI

Copy each skill folder into your project under `.cursor/skills/<skill-name>/` and ensure each folder contains a `SKILL.md` file. Reload the Cursor window.

---

## Azure DevOps Configuration

These skills integrate with **Azure DevOps** using the REST API. Ensure you have:

1. **Azure DevOps Organization URL** — e.g., `https://dev.azure.com/yourorg`
2. **Personal Access Token (PAT)** — with scopes: `Work Items (Read & Write)`, `Test Management (Read & Write)`
3. **Project Name or ID** — the target Azure project
4. **Area Path** (optional) — for organizing work items

Store your PAT securely as an environment variable or in your CI/CD secrets.

---

## Key Features

✅ **AI-Powered Test Automation** — Convert manual test cases to Cypress specs with intelligent POM generation  
✅ **Root Cause Analysis** — Blameless RCA framework with Fishbone, 5 Whys, and FMEA scoring  
✅ **Azure DevOps Integration** — Create bugs, stories, and test cases directly from AI workflows  
✅ **Framework Consistency** — Bundled references ensure clean code and architectural alignment  
✅ **Multi-IDE Support** — Works in VS Code (Copilot), Claude Code, and Cursor  

---

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository.
2. Create a new branch (`git checkout -b feature-branch`).
3. Make your changes and test them with your AI editor.
4. Commit your changes (`git commit -m 'Add new skill or improvement'`).
5. Push to the branch (`git push origin feature-branch`).
6. Open a pull request with a clear description.

---

## License

MIT

---

## Resources

- [GitHub Copilot Skills Documentation](https://github.com/github-copilot/skills-preview)
- [Azure DevOps REST API](https://learn.microsoft.com/en-us/rest/api/azure/devops)
- [Cypress Documentation](https://docs.cypress.io)
- [Root Cause Analysis - Lean Six Sigma](https://en.wikipedia.org/wiki/Root_cause_analysis)
