# Cypress Folder Structure

When you install Cypress and run `npx cypress open` for the first time,
it scaffolds a **default folder structure** inside your project:

    cypress/
      ├── e2e/             # End-to-end test files (.cy.js / .cy.ts)
      ├── fixtures/        # Mock data (JSON files or static data)
      ├── support/         # Custom commands & global setup
      ├── downloads/       # Downloaded files during tests (optional)
      ├── screenshots/     # Saved screenshots from test failures
      ├── videos/          # Test run videos (when using cypress run)
    cypress.config.js      # Main Cypress configuration file

------------------------------------------------------------------------

## 📂 Folder Explanations

### 1. `cypress/e2e/`

-   Holds your **test files**.
-   Each `.cy.js` or `.cy.ts` file corresponds to a test suite.

**Example:**

``` js
// cypress/e2e/login.cy.js
describe('Login Test', () => {
  it('should login successfully', () => {
    cy.visit('/login');
    cy.get('input[name="username"]').type('user');
    cy.get('input[name="password"]').type('password');
    cy.get('button[type="submit"]').click();
    cy.url().should('include', '/dashboard');
  });
});
```

------------------------------------------------------------------------

### 2. `cypress/fixtures/`

-   Contains **static data or mock data** in JSON or JS format.
-   Useful for stubbing network responses or providing test input.

**Example:**

``` json
// cypress/fixtures/user.json
{
  "username": "testuser",
  "password": "secret123"
}
```

Usage in test:

``` js
cy.fixture('user').then((user) => {
  cy.get('input[name="username"]').type(user.username);
  cy.get('input[name="password"]').type(user.password);
});
```

------------------------------------------------------------------------

### 3. `cypress/support/`

-   Holds **global configuration and reusable logic**.
-   Files:
    -   `commands.js` → define **custom Cypress commands** (e.g.,
        `cy.login()`)
    -   `e2e.js` → runs before every test file (global setup, imports,
        hooks).

**Example:**

``` js
// cypress/support/commands.js
Cypress.Commands.add('login', (username, password) => {
  cy.get('input[name="username"]').type(username);
  cy.get('input[name="password"]').type(password);
  cy.get('button[type="submit"]').click();
});
```

------------------------------------------------------------------------

### 4. `cypress/downloads/`

-   Stores **files downloaded** during tests.
-   Useful when testing export features (e.g., CSV, PDF downloads).

------------------------------------------------------------------------

### 5. `cypress/screenshots/`

-   When a test fails (or when you call `cy.screenshot()`), Cypress
    saves a **screenshot** here.
-   Helps in debugging test failures.

------------------------------------------------------------------------

### 6. `cypress/videos/`

-   When running `npx cypress run`, Cypress records a **video of the
    test execution** and saves it here.
-   Useful for CI/CD debugging.

------------------------------------------------------------------------

## ⚙️ Configuration Files

### `cypress.config.js`

Main Cypress config file.

**Example:**

``` js
const { defineConfig } = require("cypress");

module.exports = defineConfig({
  e2e: {
    baseUrl: "http://localhost:3000",
    setupNodeEvents(on, config) {
      // implement node event listeners here
    },
  },
});
```

------------------------------------------------------------------------

## 📝 Summary

-   **`e2e/`** → Test files\
-   **`fixtures/`** → Static/mock data\
-   **`support/`** → Global setup + custom commands\
-   **`downloads/`** → Stores downloaded files\
-   **`screenshots/`** → Auto screenshots on failure\
-   **`videos/`** → Test execution recordings\
-   **`cypress.config.js`** → Central config file
