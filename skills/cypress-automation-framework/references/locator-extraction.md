# Locator Extraction Workflow

## Overview

This guide covers how to discover, extract, and validate stable CSS selectors from a live website. Use this when you need to add new locators for a page or feature.

---

## When to Extract Locators

✅ Adding automation for a new page or feature  
✅ Updating locators after a UI redesign  
✅ Discovering stable selectors for dynamic content  
✅ Testing multi-language or multi-region UI variations  

---

## Selector Stability Hierarchy (Best to Worst)

| Priority | Selector Type | Stability | Example |
| --- | --- | --- | --- |
| 🟢 **1** | `data-test` attribute | Very High | `[data-test='email-input']` |
| 🟢 **2** | `id` attribute | Very High | `#submit-button` |
| 🟡 **3** | Class name (intentional) | High | `.primary-button` |
| 🟡 **4** | Semantic HTML | High | `button[type='submit']` |
| 🔴 **5** | nth-child selectors | Low | `.form > div:nth-child(2)` |
| 🔴 **6** | XPath | Low | `//button[contains(text(), 'Sign In')]` |
| 🔴 **7** | :contains() pseudo | Very Low | `button:contains('Sign In')` |

**Preference:** Prefer data-test attributes or semantic HTML. Avoid :nth-child and :contains selectors as they break easily.

---

## Step 1: Prepare for Extraction

### 1a. Gather Required Information

Before starting locator discovery, collect:

- **Website URL** — e.g., `https://example.com`
- **Login credentials** — Valid user account for the target environment
- **Login flow** — How to authenticate (email/password, SSO, MFA)
- **Target page** — The page or feature you're extracting locators for
- **Access restrictions** — VPN, IP allowlist, CAPTCHA, etc.

### 1b. Set Up the Environment

```bash
# Clone/navigate to your Cypress project
cd /path/to/cypress-project

# Ensure dependencies are installed
npm install

# Check Cypress is working
npx cypress --version
```

---

## Step 2: Run the Locator Extraction Script

Two options:

### Option A: Node Script (Recommended for Initial Discovery)

```bash
node skills/cypress-automation-framework/scripts/extract-cypress-locators.js
```

**What it does:**
- Opens a browser window
- Logs in with your credentials
- Navigates to the target page
- Scans the DOM for stable selectors
- Outputs candidates to `locators-output.json`

**Interactive prompts:**
```
✓ Website URL: https://example.com
✓ Email: test@example.com
✓ Password: ••••••••••
✓ Login selector (email field): [data-test='email']
✓ Login selector (password field): [data-test='password']
✓ Login selector (submit button): button[type='submit']
✓ Target page URL (after login): https://example.com/dashboard

Extracting selectors...
```

### Option B: Cypress Interactive Mode (For Manual Validation)

```bash
npx cypress run --spec skills/cypress-automation-framework/scripts/extract-cypress-locators.cy.js
```

Or open Cypress UI:

```bash
npx cypress open
# Then select the locator extraction spec
```

---

## Step 3: Review and Filter Output

The script generates `locators-output.json`:

```json
{
  "loginPage": {
    "email": {
      "selectors": [
        "[data-test='email-input']",
        "[name='email']",
        "input[type='email']",
        "#email"
      ],
      "recommended": "[data-test='email-input']",
      "stability": "very_high"
    },
    "submitButton": {
      "selectors": [
        "[data-test='submit']",
        "button[type='submit']",
        ".btn-primary"
      ],
      "recommended": "[data-test='submit']",
      "stability": "very_high"
    }
  }
}
```

**Review each locator:**

✅ **Accept** the recommended selector if it has:
- data-test or id attribute
- High stability rating
- Clear intent (not brittle nth-child or :contains)

⚠️ **Review** if the recommended selector is a class name:
- Check if it's intentional (e.g., `.primary-button`) vs generated (e.g., `.css-j7qwjs`)
- Prefer data-test if available

❌ **Reject** selectors that are:
- nth-child or :contains based
- Using generated/hashed class names (e.g., `.css-12345`)
- Too general (e.g., `div`, `span`)

---

## Step 4: Validate Selectors in a Test

Create a quick validation spec to ensure selectors work:

```javascript
// cypress/e2e/specs/locator-validation.cy.js

describe('Locator Validation', () => {
  it('should find all extracted locators on the page', () => {
    cy.visit('/');
    
    // Login
    cy.get('[data-test="email-input"]').should('exist');
    cy.get('[data-test="password-input"]').should('exist');
    cy.get('button[type="submit"]').should('exist');
    
    // Perform login
    cy.get('[data-test="email-input"]').type('test@example.com');
    cy.get('[data-test="password-input"]').type('password');
    cy.get('button[type="submit"]').click();
    
    // Verify we're on the target page
    cy.url().should('include', '/dashboard');
    cy.get('[data-test="user-greeting"]').should('contain', 'Welcome');
  });
});
```

Run the validation:

```bash
npx cypress run --spec cypress/e2e/specs/locator-validation.cy.js
```

---

## Step 5: Update the Selectors Fixture

Once validated, add the selectors to your fixture file:

**cypress/fixtures/selectors.json:**

```json
{
  "loginPage": {
    "emailInput": "[data-test='email-input']",
    "passwordInput": "[data-test='password-input']",
    "signInButton": "button[type='submit']",
    "errorAlert": ".alert-danger"
  },
  "dashboardPage": {
    "userGreeting": "[data-test='user-greeting']",
    "logoutButton": "[data-test='logout']",
    "profileMenu": "[data-test='profile-dropdown']"
  }
}
```

---

## Step 6: Create or Update Page Objects

Add the new page object or extend an existing one:

```javascript
// cypress/e2e/pages/LoginPage.js

class LoginPage {
  constructor() {
    cy.fixture('selectors').then(sel => {
      this.selectors = sel.loginPage;
    });
  }

  login(email, password) {
    cy.get(this.selectors.emailInput).type(email);
    cy.get(this.selectors.passwordInput).type(password);
    cy.get(this.selectors.signInButton).click();
    return new DashboardPage();
  }

  getErrorMessage() {
    return cy.get(this.selectors.errorAlert).invoke('text');
  }
}

module.exports = LoginPage;
```

---

## Common Challenges & Solutions

### Issue: "Selector not found" errors

**Cause:** Element is dynamically rendered or behind login wall

**Solution:**
1. Ensure login credentials are correct
2. Add waits for dynamic content: `cy.get(selector, { timeout: 10000 })`
3. Check if element is inside an iframe: use `cy.iframe()` if needed

### Issue: Selector works in Chrome but not in Firefox

**Cause:** Browser-specific CSS or JavaScript behavior

**Solution:**
1. Test the selector in both browsers
2. Use semantic HTML selectors (e.g., `button[type='submit']`) over browser-specific ones
3. Add browser-specific fixtures if necessary

### Issue: Selector is fragile (changes often)

**Cause:** Selector relies on dynamic classes or nth-child

**Solution:**
1. Ask developers to add `data-test` attributes
2. Use semantic HTML: `button:not(.secondary)`
3. Use parent container + more specific child: `form.login-form input[type='email']`

---

## Best Practices

1. **Prefer data-test attributes** — They're explicit and intentional
2. **Use semantic selectors** — `button[type='submit']` is more stable than `.btn-primary`
3. **Validate in multiple browsers** — Don't assume Chrome-only behavior
4. **Document tricky selectors** — Leave a comment if a selector is fragile or environment-specific
5. **Review regularly** — Selectors can break after UI updates; keep them current

---

## Selector Validation Checklist

- [ ] Selector is unique on the page
- [ ] Selector works in all supported browsers
- [ ] Selector has high stability (data-test, id, or semantic HTML)
- [ ] Selector is documented in the fixture
- [ ] Page object method encapsulates the selector
- [ ] Validation test passes
