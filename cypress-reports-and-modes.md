# 📊 Generate HTML Reports in Cypress & Run in Headed/Headless Mode

This guide shows **practical ways to produce human‑friendly HTML reports** from Cypress runs and how to execute tests in both **headless** and **headed** modes—locally and in CI.

---

## ✅ TL;DR

- **Headless** (default in `cypress run`):  
  ```bash
  npx cypress run                   # Electron headless (default)
  npx cypress run --browser chrome  # Chrome headless
  ```

- **Headed** (visible browser):  
  ```bash
  npx cypress run --headed --browser chrome
  npx cypress open                  # Interactive Test Runner
  ```

- **HTML Reports (popular)**:  
  - **Mochawesome** (HTML + JSON + screenshots embedding)
  - **JUnit** (XML for CI tools; convert to HTML if desired)
  - **Allure** (rich graphs, history, flaky tests – requires extra setup)

---

## 1) Headless vs Headed – When & How

### Headless
- Faster; ideal for **CI** and large suites.
- Default for `npx cypress run`.
- Produces **videos & screenshots** by default (configurable).

```bash
# default (Electron)
npx cypress run

# headless Chrome/Edge/Firefox
npx cypress run --browser chrome
npx cypress run --browser firefox
```

### Headed
- Shows the real browser UI, helpful for **debugging**.
```bash
npx cypress run --headed --browser chrome
# or open interactive runner to pick specs
npx cypress open
```

**Tip:** You can mix with spec filtering:
```bash
npx cypress run --spec "cypress/e2e/auth/**/*.cy.js" --browser chrome --headed
```

---

## 2) Mochawesome HTML Reports (Recommended)

### Install
```bash
npm i -D mochawesome mochawesome-merge marge
```

### Configure reporter
Add this to `cypress.config.js` (v10+):
```js
// cypress.config.js
const { defineConfig } = require('cypress');

module.exports = defineConfig({
  reporter: 'mochawesome',
  reporterOptions: {
    reportDir: 'cypress/reports/mocha',
    overwrite: false,
    html: false,      // we'll generate the final HTML after merge
    json: true
  },
  e2e: {
    setupNodeEvents(on, config) {
      return config;
    },
  },
});
```

### Run & Merge JSON → Generate HTML
```bash
# 1) run tests (headless or headed)
npx cypress run

# 2) merge all mochawesome JSON
npx mochawesome-merge cypress/reports/mocha/*.json > cypress/reports/mocha/merged.json

# 3) create pretty HTML
npx marge cypress/reports/mocha/merged.json --reportDir cypress/reports/mocha --inline
```

**Screenshots/Videos integration:** If you keep Cypress defaults, Mochawesome links failed test artifacts (screenshots/videos). Use `--inline` to inline assets for a single self-contained HTML.

---

## 3) JUnit (XML) + Converters

JUnit is useful for CI dashboards (GitLab/GitHub/Azure) and can be converted to HTML.

### Use JUnit Reporter
```bash
npx cypress run --reporter junit --reporter-options "mochaFile=cypress/reports/junit/results-[hash].xml,toConsole=true"
```

(Or put the same options in `cypress.config.js` `reporter` + `reporterOptions`.)

### Convert XML → HTML (optional)
Use any MOCHA/JUNIT XML to HTML converter (CLI or CI step) to produce an HTML summary if your CI does not render XML natively.

---

## 4) Allure Reports (Optional, Advanced)

Allure gives rich dashboards (history, flakes, retries). Setup varies by plugin—typical pattern:

```bash
npm i -D @shelex/cypress-allure-plugin allure-commandline
```

Then integrate plugin in `cypress/support/e2e.js` & `cypress.config.js`, run tests, and generate:
```bash
# after run generates allure-results/
npx allure generate allure-results --clean -o allure-report
npx allure open allure-report
```

> Use Allure if you need trend charts and advanced reporting. For quick HTML, Mochawesome is simpler.

---

## 5) Sample NPM Scripts

Add to `package.json` for one-liners:

```jsonc
{
  "scripts": {
    "cy:run": "cypress run",
    "cy:run:chrome": "cypress run --browser chrome",
    "cy:run:headed": "cypress run --headed --browser chrome",
    "report:merge": "mochawesome-merge cypress/reports/mocha/*.json > cypress/reports/mocha/merged.json",
    "report:html": "marge cypress/reports/mocha/merged.json --reportDir cypress/reports/mocha --inline",
    "report": "npm run report:merge && npm run report:html",
    "test:report": "npm run cy:run && npm run report"
  }
}
```

Run:
```bash
npm run test:report              # run tests then build HTML
npm run cy:run:headed            # headed Chrome
```

---

## 6) GitHub Actions CI Example (Headless + HTML)

```yaml
name: e2e
on: [push, pull_request]

jobs:
  cypress:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      # Run headless tests and produce mochawesome JSON
      - run: npx cypress run
      # Merge and build HTML report
      - run: npx mochawesome-merge cypress/reports/mocha/*.json > cypress/reports/mocha/merged.json
      - run: npx marge cypress/reports/mocha/merged.json --reportDir cypress/reports/mocha --inline
      # Upload HTML as artifact
      - uses: actions/upload-artifact@v4
        with:
          name: mochawesome-report
          path: cypress/reports/mocha
```

Artifacts in the workflow will contain the **HTML** file (`mochawesome.html`) and assets.

---

## 7) Tips & Troubleshooting

- If **no JSON files** appear, ensure `reporterOptions.json=true` and **do not** overwrite files (`overwrite: false`).
- For **parallel runs**, you still merge all JSONs before generating the single HTML.
- To **embed screenshots** directly, use `--inline` with `marge`.
- In **headed** mode via `cypress run --headed`, reporting works the same as headless; `cypress open` is primarily for local debugging and does not auto-generate run reports.
- Combine with failure hooks for extra screenshots: see your `cypress-screenshots-videos.md` doc.

---

## ✅ Quick Commands

```bash
# Headless (default) / Headed
npx cypress run
npx cypress run --headed --browser chrome

# Mochawesome HTML
npx cypress run
npx mochawesome-merge cypress/reports/mocha/*.json > cypress/reports/mocha/merged.json
npx marge cypress/reports/mocha/merged.json --reportDir cypress/reports/mocha --inline
```

Happy reporting! 📑✨
