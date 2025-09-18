# 📸 How To Capture Screenshots & Videos on Test Failures (Cypress)

Cypress can automatically **save screenshots on failures** and **record videos** of spec runs. Here’s a practical, CI‑ready setup with examples.

---

## ✅ TL;DR
- **Automatic screenshots on failure**: enabled by default during `cypress run` (not in open mode).
- **Videos**: recorded by default during `cypress run` (can be disabled).
- **Manual screenshots**: `cy.screenshot('name', { options })`.
- **Only on failure**: use an `afterEach` hook to grab extra context when a test fails.

---

## ⚙️ Configure in `cypress.config.js` (v10+)

```js
// cypress.config.js
const { defineConfig } = require('cypress');

module.exports = defineConfig({
  // Top-level recording options
  video: true,                 // record videos during `cypress run`
  videosFolder: 'cypress/videos',
  screenshotsFolder: 'cypress/screenshots',
  screenshotOnRunFailure: true, // auto-screenshot when an assertion fails during `run`

  // (Optional) tweak video behavior
  // videoCompression: 32,      // 32 is default; 0 = no compression
  // videoUploadOnPasses: false // skip uploading/keeping videos for passing specs in some CI providers

  e2e: {
    baseUrl: 'http://localhost:3000',
    setupNodeEvents(on, config) {
      // node event listeners here if needed
      return config;
    },
  },
});
```

> 📝 **Notes**
> - Auto **screenshots/videos** are produced in **`cypress run`** (headless/CI). In interactive `cypress open`, videos are not recorded and auto-screenshots aren’t written unless you call `cy.screenshot()` yourself.
> - Folder names above are the Cypress defaults—customize if you like.

---

## 🧪 CLI Examples

```bash
# Record videos + screenshots on failures (default behavior)
npx cypress run

# Disable video for a run (still keeps screenshots on failure)
npx cypress run --config video=false

# Force fresh run ignoring browser cache (useful while debugging reload behavior)
npx cypress run --browser chrome
```

---

## 🎯 Manual Screenshots (any mode)

```js
// Full-page or viewport capture
cy.visit('/dashboard');

// default: viewport
cy.screenshot('dashboard-viewport');

// full page
cy.screenshot('dashboard-full', { capture: 'fullPage' });

// area clip (x, y, width, height)
cy.screenshot('crop', { clip: { x: 0, y: 0, width: 800, height: 600 } });

// after an action
cy.get('[data-cy="save"]').click();
cy.screenshot('after-save');
```

**Common `capture` options**: `'viewport'` (default), `'fullPage'`, `'runner'` (the Cypress runner UI).

---

## 🚨 Take Extra Screenshot Only When a Test Fails

Sometimes the built‑in “on-failure” screenshot isn’t enough. Add your own **failure‑only** capture:

```js
// In a spec file or a shared hook (function syntax required for `this`)
afterEach(function () {
  if (this.currentTest?.state === 'failed') {
    const name = `FAILED - ${this.currentTest.title}`;
    cy.screenshot(name, { capture: 'fullPage' });
  }
});
```

> Tip: Use fullPage for maximum debugging detail, or add `clip` to focus on a widget.

---

## 🗂️ Where Files Are Saved

- **Screenshots**: `cypress/screenshots/<spec path>/<test name> (failed).png`
- **Videos**: `cypress/videos/<spec name>.mp4`

Cypress auto‑names files by **spec path** and **test title** for easy traceability.

---

## 🧰 Bonus: Helpful Patterns

### Name artifacts consistently
```js
const stamp = () => new Date().toISOString().replace(/[:.]/g, '-');

cy.screenshot(`checkout-${stamp()}`);
```

### Combine with network aliases
```js
cy.intercept('POST', '/api/checkout').as('checkout');
cy.dataCy('pay').click();
cy.wait('@checkout').its('response.statusCode').should('eq', 200);
cy.screenshot('after-checkout');
```

### Disable screenshots/videos for a noisy spec
```js
// At runtime per spec
Cypress.config('screenshotOnRunFailure', false);
Cypress.config('video', false);
```

---

## ✅ Quick Checklist
- [ ] Set `video: true` and `screenshotOnRunFailure: true` in `cypress.config.js`.
- [ ] Run tests in CI with `npx cypress run` to produce artifacts.
- [ ] Add an `afterEach` failure hook for extra context when needed.
- [ ] Use `cy.screenshot('name', { capture: 'fullPage' })` for manual captures.
- [ ] Organize artifacts by spec/test names for easy debugging.

Happy debugging! 🧪📽️
