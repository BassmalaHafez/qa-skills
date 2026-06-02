# Framework Structure

## Folder Layout

```
cypress/
├── e2e/
│   ├── specs/              # Test scenarios (one feature per file)
│   │   ├── login.cy.js
│   │   ├── checkout.cy.js
│   │   └── payment.cy.js
│   │
│   └── pages/              # Page Object Models
│       ├── LoginPage.js
│       ├── DashboardPage.js
│       ├── CheckoutPage.js
│       └── BasePage.js     # Common methods
│
├── fixtures/               # Selectors, test data, localized text
│   ├── selectors.json      # UI element locators (CSS, XPath, data-test)
│   ├── users.json          # Test user accounts
│   ├── testdata.json       # Purchase orders, addresses, etc.
│   └── locales/
│       ├── en.json         # English UI labels
│       └── es.json         # Spanish UI labels
│
├── support/                # Shared utilities and custom commands
│   ├── commands.js         # cy.login(), cy.fillForm(), cy.verifyAlert(), etc.
│   └── e2e.js             # Global hooks (before, after, beforeEach)
│
├── plugins/                # Cypress plugins (email, database, file handling)
│   └── index.js
│
└── cypress.config.js       # Configuration (environment, base URL, timeouts)
```

## Layer Ownership

### `specs/` — Test Scenarios

**Responsibility:** Define test scenarios and expected outcomes.

**DO:**
- Describe the user journey in plain language
- Use page object methods (e.g., `loginPage.login(email, password)`)
- Use fixtures for test data (e.g., `cy.fixture('users').then(users => ...)`)
- Organize by feature: one file per feature/module

**DON'T:**
- Hardcode selectors or UI copy
- Mix setup logic with assertions
- Create long, linear test blocks — break into smaller, focused tests

### `pages/` — Page Objects

**Responsibility:** Encapsulate element access and page-specific verification logic.

**DO:**
- Load selectors from fixtures (`cy.fixture('selectors').then(...)`)
- Expose high-level methods: `login()`, `getErrorMessage()`, `verifyCheckoutTotal()`
- Keep methods focused and reusable
- Use helper methods for repeated patterns (e.g., `fillInput()`, `selectDropdown()`)

**DON'T:**
- Put test data into page objects
- Hardcode selectors
- Create spec-specific methods — generalize for reuse

**Example:**
```javascript
class LoginPage extends BasePage {
  constructor() {
    super();
    cy.fixture('selectors').then(sel => {
      this.selectors = sel.loginPage;
    });
  }

  login(email, password) {
    cy.get(this.selectors.emailInput).type(email);
    cy.get(this.selectors.passwordInput).type(password);
    cy.get(this.selectors.signInButton).click();
    return new DashboardPage(); // Return next page
  }

  getErrorMessage() {
    return cy.get(this.selectors.errorAlert).invoke('text');
  }
}
```

### `fixtures/` — Selectors, Test Data, Localization

**Responsibility:** Store all external data — UI selectors, test accounts, labels, and configuration.

**Structure:**

**selectors.json:**
```json
{
  "loginPage": {
    "emailInput": "[data-test='email-input']",
    "passwordInput": "[data-test='password-input']",
    "signInButton": "button:contains('Sign In')",
    "errorAlert": ".alert-danger"
  },
  "checkoutPage": {
    "cartTotal": "[data-test='cart-total']",
    "paymentButton": "button[type='submit']"
  }
}
```

**users.json:**
```json
{
  "validUser": {
    "email": "test@example.com",
    "password": "SecurePassword123!",
    "name": "Test User"
  },
  "invalidUser": {
    "email": "invalid@example.com",
    "password": "WrongPassword"
  }
}
```

**locales/en.json:**
```json
{
  "loginPage": {
    "heading": "Sign In",
    "forgotPasswordLink": "Forgot Password?",
    "errorMessage": "Invalid credentials"
  }
}
```

### `support/commands.js` — Reusable Commands

**Responsibility:** Define custom commands for repeated multi-step flows.

**DO:**
- Create commands for common flows: `cy.login()`, `cy.fillCheckout()`, `cy.verifyOrder()`
- Use commands in specs to keep test code clean
- Document each command with JSDoc comments

**DON'T:**
- Create commands that do too much (keep them focused)
- Hardcode test data or selectors

**Example:**
```javascript
Cypress.Commands.add('login', (email, password) => {
  const loginPage = new LoginPage();
  loginPage.login(email, password);
});

Cypress.Commands.add('fillCheckout', (address, paymentMethod) => {
  const checkoutPage = new CheckoutPage();
  checkoutPage.fillAddress(address);
  checkoutPage.selectPayment(paymentMethod);
});
```

### `cypress.config.js` — Configuration

**Responsibility:** Set up environments, timeouts, base URLs, and plugins.

**Example:**
```javascript
module.exports = {
  e2e: {
    baseUrl: process.env.BASE_URL || 'http://localhost:3000',
    viewportWidth: 1280,
    viewportHeight: 720,
    defaultCommandTimeout: 5000,
    requestTimeout: 10000,
    env: {
      environment: 'staging',
      MAILOSAUR_API_KEY: process.env.MAILOSAUR_API_KEY,
    },
  },
};
```

---

## Naming Conventions

| Element | Naming Pattern | Example |
| --- | --- | --- |
| Spec files | `{feature}.cy.js` | `login.cy.js`, `checkout.cy.js` |
| Page object files | `{Page}Page.js` | `LoginPage.js`, `CheckoutPage.js` |
| Page object classes | PascalCase | `class LoginPage`, `class CheckoutPage` |
| Page object methods | camelCase, verb-first | `login()`, `fillEmail()`, `getErrorMessage()` |
| Custom commands | camelCase, verb-first | `cy.login()`, `cy.fillCheckout()` |
| Fixture files | snake_case | `selectors.json`, `test_users.json` |
| Selector keys in fixtures | camelCase | `emailInput`, `signInButton` |

---

## Best Practices

1. **One Feature Per Spec File** — Keep specs focused. Don't mix login tests with checkout tests.
2. **Page Objects Own Selectors** — Never hardcode selectors in specs.
3. **Fixtures Own Test Data** — Never hardcode test accounts or purchase data in specs.
4. **Commands Own Reusable Flows** — Don't repeat multi-step actions in multiple specs.
5. **Organize by Feature, Not by Layer** — Group related tests together, not all specs in one file.
6. **Use Locale Fixtures for i18n** — When testing multiple languages, load locale-specific labels from fixtures.

