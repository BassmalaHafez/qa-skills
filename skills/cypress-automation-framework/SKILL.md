---
name: cypress-automation-framework
description: >
  Consolidated Cypress framework skill. Covers the full automation lifecycle — converting manual test cases, adding/updating/refactoring specs, extending page objects, managing fixtures and support commands, and debugging flaky tests. Includes bundled references for clean code standards, framework structure, locator extraction, and manual test case conversion.
user-invocable: true
---

# Cypress Automation Framework — Skill Definition

This skill covers the **complete Cypress automation lifecycle** for your project. Use it when you need to convert manual test cases, write or refactor specs, extend page objects, manage fixtures, debug flaky tests, or extract selectors from live sites.

---

## Two Entry Points

| Entry point | Context | Best for |
| --- | --- | --- |
| **Skill** | Claude Code · Cursor AI · any LLM that loads skills from the project tree | Broader use beyond VS Code |
| **Agent** | [agents/cypress-automation.agent.md](../../agents/cypress-automation.agent.md) | GitHub Copilot Chat (VS Code) in **Agent** mode with tool access |

Both share the same scope, constraints, and working rules.

---

## Scope

You own the full Cypress automation stack:

- **Specs** (`cypress/e2e/specs/*.cy.js`) — Test scenarios and coverage
- **Page Objects** (`cypress/e2e/pages/*.js`) — Element access and verification helpers
- **Fixtures** (`cypress/fixtures/*.json`) — Selectors, test data, and localized UI copy
- **Support Commands** (`cypress/support/commands.js`) — Reusable multi-step flows
- **Config** (`cypress.config.js`) — Environment, plugins, custom tasks
- **Scripts** (tools for locator extraction, test data generation)

---

## When to Use This Skill

✅ **Convert manual test cases to Cypress automation**  
✅ **Write new specs** for an existing feature or flow  
✅ **Update page objects** to add new locators or verification helpers  
✅ **Manage fixtures** for selectors, test data, or localized text  
✅ **Add/update support commands** for reusable flow logic  
✅ **Debug flaky tests** — race conditions, timing, wait strategies  
✅ **Extract selectors** from a live website  
✅ **Refactor framework code** for readability and maintainability  
✅ **Review code quality** against framework standards  

---

## Required Input

Before starting work on a **live-site task** or **locator discovery**, ask the user for:

1. The **website URL** to open
2. **Valid credentials** for the target environment
3. The **login selectors or steps** — login pages are project-specific
4. The **target page or flow** to inspect after login
5. Any **MFA, CAPTCHA, VPN, or IP restrictions** that could block automation

**Do not start locator extraction or live-site work until you have the URL and login details.**

---

## Constraints

- ❌ **DO NOT** introduce raw selectors into specs when a page object or fixture should own them
- ❌ **DO NOT** hardcode localized UI copy in test logic when the fixture should own that text
- ❌ **DO NOT** bypass the project's shared constants or config layer for environment URLs, schemas, routes, or language-aware navigation
- ❌ **DO NOT** add broad refactors, renames, or framework changes unless the task explicitly requires them
- ✅ **ONLY** run the narrowest useful Cypress command unless the user asks for a wider run

---

## Working Rules

1. **Preserve the existing page-object plus fixture pattern** when the framework uses one.
   - Page classes should load shared selector or text data and expose getters or verification helpers.

2. **Prefer reusable `Cypress.Commands.add(...)`** helpers for repeated flows over duplicating steps across specs.

3. **Use the data factory** for generated form data instead of inventing ad hoc inline test data.

4. **Keep locale-specific coverage explicit** when behavior differs across supported languages or regions.

5. **Follow the existing email-provider and `cy.task(...)`** patterns for email and database validation.

6. **Read the related files together** before editing:
   - Relevant spec
   - Page object
   - Fixture
   - Support command
   - Config file

   This ensures behavior stays aligned.

7. **When starting from a manual test case**, convert each precondition, action, and expected result into framework-owned automation steps instead of mirroring the manual script line by line in one large spec.

---

## Bundled References

These documents are loaded on demand — **you do not need to reference them manually**:

| Reference | When it is loaded |
| --- | --- |
| [framework-structure.md](./references/framework-structure.md) | Deciding where new code belongs or understanding the project layout |
| [clean-code-standards.md](./references/clean-code-standards.md) | Reviewing, writing, or refactoring any Cypress code |
| [locator-extraction.md](./references/locator-extraction.md) | Live-site selector discovery |
| [manual-test-case-conversion.md](./references/manual-test-case-conversion.md) | Converting manual test cases or business flows to automation |

---

## Procedure

### For Manual Test Case Conversion

1. Extract **preconditions**, **navigation steps**, **test data**, **assertions**, and **cleanup needs** from the manual test case before writing code.
2. Inspect the **spec**, **page object**, **fixture**, **support command**, and **config** files together.
3. Extend the **smallest layer** that owns the behavior:
   - Fixture → for selectors or localized text
   - Page object → for element access or verification helpers
   - Support command → for reusable flow logic
   - Spec → for scenario coverage
4. Validate with the **narrowest useful Cypress command** (usually a single spec or environment-specific run).
5. Report the **files changed**, **validation performed**, **environment prerequisites**, and **remaining risks**.

### For Live-Site Locator Extraction

1. Ask for the **website URL**, **credentials**, **login sequence**, **target page**, and any **access restrictions**.
2. Run the locator extraction script: `node skills/cypress-automation-framework/scripts/extract-cypress-locators.js`
3. Review the output for **stable selectors** — prefer data-test attributes or class names over dynamic content.
4. Create or update the **fixture** file with the extracted selectors.
5. Validate selectors in a **quick test run** on the actual page.

### For Code Review

1. Read the provided code file(s) in context of the framework.
2. Check against **clean-code-standards.md** and **framework-structure.md**.
3. Identify **risks** and **improvements** in three tiers:
   - 🔴 Critical (blocks reliability or breaks patterns)
   - 🟡 Medium (improves maintainability)
   - 🟢 Low (nice-to-have polish)
4. Provide **examples** and **actionable recommendations** for each finding.

---

## Output

Return:

- The **files changed** (with line numbers or code snippets)
- The **Cypress command(s) run**, if any (e.g., `npx cypress run --spec cypress/e2e/specs/login.cy.js`)
- Any **prerequisites** such as email test access, database credentials, or environment variables
- **Remaining risks** or follow-up work needed

---

## Quick-Start Examples

### Convert a manual test case

```
Convert this manual test case to Cypress:
  Given I am on the login page
  When I enter valid credentials
  Then I should see the dashboard
```

### Add a new spec for an existing flow

```
Add a spec for the "forgot password" flow in the staging environment
```

### Extract locators from a live page

```
Find selectors for the checkout page at https://example.com
```

> The skill will ask for the URL, credentials, and login sequence before starting.

### Review code quality

```
Is this page object following clean code standards?
```

### Debug a flaky test

```
This spec is flaky — cypress/e2e/specs/checkout.cy.js
```

---

## Layer Ownership Cheat-Sheet

| What you are changing | Owning layer |
| --- | --- |
| Selectors or localized UI text | `cypress/fixtures` |
| Element access or verification helpers | `cypress/e2e/pages` (page object) |
| Reusable multi-step flows | `cypress/support/commands.js` |
| Scenario coverage | `cypress/e2e/specs` |
| DB tasks or email helpers | `cypress/plugins` / `cypress.config.js` tasks |

---

## Validation Commands

```bash
# Full headless suite
npm test

# Interactive Cypress runner
npm run cy:open

# Single spec
npx cypress run --spec cypress/e2e/specs/<name>.cy.js

# Environment-specific run
npx cypress run --env environment=staging --spec cypress/e2e/specs/<name>.cy.js
```
