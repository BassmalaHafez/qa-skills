# Manual Test Case Conversion to Cypress Automation

## Overview

This guide shows how to convert a manual test case (written in Gherkin, TestRail format, or plain English) into a Cypress automation spec that follows the framework patterns.

---

## Manual Test Case Structure

A typical manual test case includes:

```
Test Case ID: TC-001
Title: User can log in with valid credentials
Priority: High

Preconditions:
- User is on the login page
- Valid user account exists in the system
- No active session

Steps:
1. Enter email: test@example.com
2. Enter password: password123
3. Click "Sign In" button

Expected Result:
- User is redirected to dashboard
- User name appears in top-right corner
- Session token is stored

Cleanup:
- Log out (optional, depends on next test)
```

---

## Conversion Process

### Step 1: Extract Information from Manual Test Case

Break down the test case into components:

| Component | From Test Case | Example |
| --- | --- | --- |
| **Feature/Module** | Title context | "Login Page" |
| **Test Scenario** | Title | "User can log in with valid credentials" |
| **Preconditions** | Setup requirements | "User on login page, valid account exists" |
| **Test Data** | Specific values in steps | `email: test@example.com`, `password: password123` |
| **Actions** | Each numbered step | Enter email → Enter password → Click sign in |
| **Assertions** | Expected results | Redirected to dashboard, name visible |
| **Cleanup** | Teardown logic | Log out if needed |

**Example extraction:**

```
Feature: Login Page
Scenario: User can log in with valid credentials

Preconditions:
- Navigate to /login
- Valid test account available

Test Data:
- Email: test@example.com
- Password: password123

Actions:
1. Enter email
2. Enter password
3. Click sign in button

Assertions:
- Redirect to /dashboard
- User name displayed
- Auth token in localStorage

Cleanup:
- cy.logout()
```

---

### Step 2: Identify Fixture & Page Object Needs

**Question:** Where should each component live?

```
Test Data (email, password) → fixtures/users.json
Selectors (email field, password field, sign-in button) → fixtures/selectors.json
Element interactions (login flow) → pages/LoginPage.js
Reusable commands (fill form, login) → support/commands.js
Test scenario → specs/login.cy.js
```

**Example breakdown:**

```javascript
// What goes in fixtures/users.json:
{
  "validUser": {
    "email": "test@example.com",
    "password": "password123"
  }
}

// What goes in fixtures/selectors.json:
{
  "loginPage": {
    "emailInput": "[data-test='email-input']",
    "passwordInput": "[data-test='password-input']",
    "signInButton": "button[type='submit']"
  },
  "dashboardPage": {
    "userGreeting": "[data-test='user-greeting']"
  }
}

// What goes in pages/LoginPage.js:
class LoginPage {
  login(email, password) { /* implementation */ }
  getErrorMessage() { /* implementation */ }
}

// What goes in cypress/e2e/specs/login.cy.js:
describe('Login Page', () => {
  it('should log in with valid credentials', () => { /* test */ })
})
```

---

### Step 3: Build the Spec

#### 3a. Create the File Structure

```javascript
// cypress/e2e/specs/login.cy.js

describe('Login Page', () => {
  beforeEach(() => {
    // Setup before each test
    cy.visit('/login');
  });

  it('should log in with valid credentials', () => {
    // Arrange
    // Act
    // Assert
  });

  afterEach(() => {
    // Cleanup after each test (optional)
  });
});
```

#### 3b. Add Preconditions (Arrange)

```javascript
it('should log in with valid credentials', () => {
  // Arrange: Load test data
  cy.fixture('users').then(users => {
    const validUser = users.validUser;
    
    // Note: cy.visit() is already in beforeEach()
    // Verify we're on the correct page
    cy.get('h1').should('contain', 'Sign In');
```

#### 3c. Add Test Steps (Act)

**Option 1: Using Page Objects (Recommended)**

```javascript
    // Act: Perform login using page object
    const loginPage = new LoginPage();
    const dashboardPage = loginPage.login(
      validUser.email,
      validUser.password
    );
```

**Option 2: Using Custom Commands**

```javascript
    // Act: Perform login using custom command
    cy.login(validUser.email, validUser.password);
```

**Option 3: Direct Interaction (Avoid for repeated flows)**

```javascript
    // Act: Manually interact with elements
    cy.fixture('selectors').then(selectors => {
      const sel = selectors.loginPage;
      cy.get(sel.emailInput).type(validUser.email);
      cy.get(sel.passwordInput).type(validUser.password);
      cy.get(sel.signInButton).click();
    });
```

#### 3d. Add Assertions (Assert)

```javascript
    // Assert: Verify login was successful
    cy.url().should('include', '/dashboard');
    cy.get('[data-test="user-greeting"]')
      .should('contain', validUser.name);
    cy.window()
      .its('localStorage')
      .invoke('getItem', 'authToken')
      .should('exist');
  });
});
```

---

### Step 4: Complete Spec Example

```javascript
// cypress/e2e/specs/login.cy.js

import LoginPage from '../pages/LoginPage';
import DashboardPage from '../pages/DashboardPage';

describe('Login Page', () => {
  beforeEach(() => {
    cy.visit('/login');
  });

  it('should display login form on page load', () => {
    // Arrange: Nothing special, page already loaded
    
    // Act: Verify form elements are visible
    cy.get('[data-test="email-input"]').should('be.visible');
    cy.get('[data-test="password-input"]').should('be.visible');
    cy.get('button[type="submit"]').should('be.visible');
  });

  it('should log in with valid credentials', () => {
    // Arrange: Load test user
    cy.fixture('users').then(users => {
      const validUser = users.validUser;
      
      // Act: Login using page object
      const loginPage = new LoginPage();
      const dashboardPage = loginPage.login(
        validUser.email,
        validUser.password
      );
      
      // Assert: Verify successful login
      cy.url().should('include', '/dashboard');
      dashboardPage.verifyUserGreeting(validUser.name);
    });
  });

  it('should display error for invalid credentials', () => {
    // Arrange: Load invalid credentials
    cy.fixture('users').then(users => {
      const invalidUser = users.invalidUser;
      
      // Act: Attempt login with wrong password
      const loginPage = new LoginPage();
      loginPage.login(invalidUser.email, invalidUser.password);
      
      // Assert: Verify error message
      loginPage.getErrorMessage()
        .should('contain', 'Invalid credentials');
    });
  });

  it('should lock account after 5 failed login attempts', () => {
    // Arrange
    cy.fixture('users').then(users => {
      const invalidUser = users.invalidUser;
      const loginPage = new LoginPage();
      
      // Act: Attempt login 5 times
      for (let i = 0; i < 5; i++) {
        loginPage.login(invalidUser.email, 'wrongPassword');
        cy.get('[data-test="email-input"]').clear();
      }
      
      // Assert: Account is locked
      loginPage.getErrorMessage()
        .should('contain', 'Account locked');
    });
  });
});
```

---

## Conversion Checklist

- [ ] Test case title → describe/it block
- [ ] Preconditions → beforeEach() or test setup
- [ ] Test data → fixtures (users.json, testdata.json)
- [ ] Selectors → fixtures (selectors.json)
- [ ] Element interactions → page objects
- [ ] Repeated flows → custom commands
- [ ] Test steps → page object methods or cy.commands
- [ ] Assertions → .should() statements
- [ ] Cleanup → afterEach() if needed
- [ ] Tests are independent (no execution order dependency)

---

## Common Patterns

### Pattern 1: Happy Path Test

```javascript
it('should complete checkout successfully', () => {
  cy.fixture('users').then(users => {
    cy.fixture('testdata').then(data => {
      // Arrange: User logged in
      cy.login(users.validUser.email, users.validUser.password);
      
      // Act: Add item to cart and checkout
      const checkoutPage = new CheckoutPage();
      checkoutPage.addToCart(data.products.laptop);
      checkoutPage.proceedToCheckout();
      checkoutPage.fillShippingAddress(data.addresses.home);
      checkoutPage.fillPaymentInfo(data.paymentMethods.visa);
      const confirmationPage = checkoutPage.submitOrder();
      
      // Assert: Order confirmation
      confirmationPage.verifyOrderNumber();
      confirmationPage.verifyOrderTotal(data.products.laptop.price);
    });
  });
});
```

### Pattern 2: Edge Case Test

```javascript
it('should not allow checkout with expired card', () => {
  cy.fixture('users').then(users => {
    cy.fixture('testdata').then(data => {
      // Arrange
      cy.login(users.validUser.email, users.validUser.password);
      const checkoutPage = new CheckoutPage();
      checkoutPage.addToCart(data.products.laptop);
      checkoutPage.proceedToCheckout();
      
      // Act: Attempt checkout with expired card
      checkoutPage.fillPaymentInfo(data.paymentMethods.expiredCard);
      checkoutPage.submitOrder();
      
      // Assert: Error message
      checkoutPage.getPaymentError()
        .should('contain', 'Card has expired');
    });
  });
});
```

---

## Tips for Effective Conversion

1. **Start with happy path tests** — Convert the primary success scenario first
2. **Then add edge cases** — Invalid input, timeouts, permission errors
3. **Keep tests focused** — One test should verify one behavior
4. **Use descriptive test names** — Read like a user story
5. **Extract common flows into page objects** — Don't repeat login logic
6. **Use fixtures for all external data** — Makes tests maintainable
7. **Validate selectors before writing specs** — Use the locator extraction guide
8. **Review framework standards** — Ensure code matches clean code standards

