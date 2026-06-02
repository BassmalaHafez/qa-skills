# Clean Code Standards for Cypress

## Spec Files (`cypress/e2e/specs/*.cy.js`)

### ✅ DO

**1. Use descriptive test names**

```javascript
// ✅ GOOD
describe('Login Page', () => {
  it('should display error message for invalid credentials', () => {
    // test body
  });

  it('should redirect to dashboard on successful login', () => {
    // test body
  });
});

// ❌ BAD
describe('Login', () => {
  it('test login', () => {
    // unclear what is being tested
  });
});
```

**2. Organize with arrange-act-assert (AAA)**

```javascript
// ✅ GOOD
it('should apply discount code to cart', () => {
  // Arrange: Set up initial state
  cy.fixture('testdata').then(data => {
    cy.visit('/cart');
    cy.addToCart(data.products.laptop);
  });

  // Act: Perform the action
  cy.get('[data-test="discount-input"]').type('SAVE10');
  cy.get('[data-test="apply-discount-btn"]').click();

  // Assert: Verify the outcome
  cy.get('[data-test="discount-amount"]')
    .should('contain', '$50.00');
  cy.get('[data-test="total-price"]')
    .should('contain', '$450.00');
});
```

**3. Use page objects for element interaction**

```javascript
// ✅ GOOD
const checkoutPage = new CheckoutPage();
it('should complete checkout flow', () => {
  checkoutPage.fillShippingAddress(address);
  checkoutPage.selectShippingMethod('express');
  checkoutPage.fillPaymentInfo(cardDetails);
  checkoutPage.submitOrder();
  checkoutPage.verifyOrderConfirmation();
});

// ❌ BAD
it('should complete checkout flow', () => {
  cy.get('input[name="street"]').type('123 Main St');
  cy.get('input[name="city"]').type('New York');
  cy.get('select[name="shipping"]').select('express');
  // ... etc (logic scattered, hard to maintain)
});
```

**4. Use fixtures for test data**

```javascript
// ✅ GOOD
it('should login with valid credentials', () => {
  cy.fixture('users').then(users => {
    cy.login(users.validUser.email, users.validUser.password);
    cy.get('[data-test="user-greeting"]')
      .should('contain', users.validUser.name);
  });
});

// ❌ BAD
it('should login with valid credentials', () => {
  cy.get('input[name="email"]').type('test@example.com');
  cy.get('input[name="password"]').type('SecurePassword123!');
  // ... hardcoded test data is hard to change globally
});
```

**5. Keep tests focused and independent**

```javascript
// ✅ GOOD
describe('Checkout', () => {
  beforeEach(() => {
    cy.visit('/cart');
    cy.fixture('testdata').then(data => {
      cy.addToCart(data.products.laptop);
    });
  });

  it('should display order summary', () => {
    cy.get('[data-test="order-summary"]').should('be.visible');
  });

  it('should apply discount code', () => {
    cy.applyDiscountCode('SAVE10');
    cy.get('[data-test="discount-amount"]').should('contain', '$50');
  });
});

// ❌ BAD
describe('Checkout', () => {
  it('should display order summary and apply discount and submit', () => {
    // Testing 3 different things in one test
    // Hard to debug when it fails
  });
});
```

### ❌ DON'T

- ❌ Hardcode selectors in specs → use page objects
- ❌ Hardcode test data in specs → use fixtures
- ❌ Test multiple features in one spec → keep specs focused
- ❌ Rely on test execution order → make tests independent
- ❌ Use generic assertions → use specific, meaningful assertions

---

## Page Objects (`cypress/e2e/pages/*.js`)

### ✅ DO

**1. Load selectors from fixtures**

```javascript
// ✅ GOOD
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
  }
}

// ❌ BAD
class LoginPage {
  login(email, password) {
    // Selectors hardcoded in page object
    cy.get('[name="email"]').type(email);
    cy.get('[name="password"]').type(password);
    cy.get('button[type="submit"]').click();
  }
}
```

**2. Expose high-level methods**

```javascript
// ✅ GOOD
class CheckoutPage {
  completeOrder(address, paymentInfo) {
    this.fillShippingAddress(address);
    this.fillPaymentInfo(paymentInfo);
    this.submitOrder();
    return new OrderConfirmationPage();
  }

  private fillShippingAddress(address) {
    cy.get(this.selectors.streetInput).type(address.street);
    // ...
  }
}

// ❌ BAD
class CheckoutPage {
  // Only low-level getters; spec has to orchestrate the flow
  getStreetInput() { return cy.get('[data-test="street"]'); }
  getPaymentInput() { return cy.get('[data-test="payment"]'); }
}
```

**3. Return the next page object for chaining**

```javascript
// ✅ GOOD
const dashboardPage = new LoginPage()
  .login(email, password);

dashboardPage.verifyUserGreeting(name);

// Enables fluent, readable test code
```

**4. Use descriptive method names**

```javascript
// ✅ GOOD
page.fillCheckoutForm(...);
page.verifyErrorMessage(...);
page.getCartTotal();
page.clickApplyDiscountButton();

// ❌ BAD
page.fill(...);
page.verify(...);
page.getTotal();
page.click();
```

### ❌ DON'T

- ❌ Hardcode selectors → load from fixtures
- ❌ Put test data in page objects → inject as parameters
- ❌ Create spec-specific methods → generalize for reuse
- ❌ Mix selectors and test logic → keep concerns separated

---

## Fixtures (`cypress/fixtures/*.json`)

### ✅ DO

**1. Keep selectors organized by page/component**

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
    "discountInput": "[data-test='discount-code']"
  }
}
```

**2. Use meaningful variable names**

```json
{
  "validUser": {
    "email": "test@example.com",
    "password": "SecurePassword123!"
  },
  "expiredCard": {
    "number": "4242424242424242",
    "expiry": "01/20"
  }
}
```

**3. Group related data**

```json
{
  "products": {
    "laptop": { "sku": "LAP-001", "name": "Dell XPS", "price": 999.99 },
    "mouse": { "sku": "MOU-002", "name": "Logitech", "price": 29.99 }
  },
  "addresses": {
    "homeAddress": { "street": "123 Main St", "city": "New York" },
    "workAddress": { "street": "456 Work Ave", "city": "Boston" }
  }
}
```

### ❌ DON'T

- ❌ Use overly generic names → be specific about what each fixture represents
- ❌ Mix unrelated data → organize by feature/page
- ❌ Hardcode absolute values that change → use relative or parameterized values

---

## Support Commands (`cypress/support/commands.js`)

### ✅ DO

**1. Create commands for repeated multi-step flows**

```javascript
// ✅ GOOD
Cypress.Commands.add('login', (email, password) => {
  cy.visit('/login');
  cy.get('[data-test="email"]').type(email);
  cy.get('[data-test="password"]').type(password);
  cy.get('[data-test="sign-in"]').click();
  cy.get('[data-test="dashboard"]').should('be.visible');
});

// Usage in specs:
cy.login(users.validUser.email, users.validUser.password);
```

**2. Add JSDoc documentation**

```javascript
// ✅ GOOD
/**
 * Logs in a user with email and password
 * @param {string} email - User email address
 * @param {string} password - User password
 * @returns {void}
 */
Cypress.Commands.add('login', (email, password) => {
  // implementation
});
```

### ❌ DON'T

- ❌ Create commands for single-action operations
- ❌ Hardcode test data in commands → accept as parameters
- ❌ Create overly complex commands → keep focused

---

## General Principles

1. **DRY (Don't Repeat Yourself)** — Extract common patterns into page objects or commands
2. **KISS (Keep It Simple, Stupid)** — Use the simplest approach; avoid over-engineering
3. **Readability First** — Spec code should read like a business user's journey, not technical jargon
4. **Maintainability** — When the UI changes, only update selectors in fixtures, not in dozens of specs
5. **Isolation** — Tests should not depend on execution order or external state

