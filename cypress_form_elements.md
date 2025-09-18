# 🎯 Cypress Form Elements Cheat Sheet

Interacting with **dropdowns**, **checkboxes**, and **radio buttons** in Cypress.

---

## 🔽 Dropdowns (Select Menus)

Use `.select()` for `<select>` dropdowns.

### Select by Visible Text
```js
cy.get('select#country').select('India');
```

### Select by Value
```js
cy.get('select#country').select('IN');
```

### Multiple Select
```js
cy.get('select[multiple]').select(['IN', 'US']);
```

### Assertion
```js
cy.get('select#country')
  .select('India')
  .should('have.value', 'IN');
```

---

## ☑️ Checkboxes

Use `.check()` and `.uncheck()`.

### Check a Single Checkbox
```js
cy.get('input[type="checkbox"][value="subscribe"]').check();
```

### Uncheck a Checkbox
```js
cy.get('input[type="checkbox"][value="subscribe"]').uncheck();
```

### Check Multiple Checkboxes
```js
cy.get('input[type="checkbox"]').check(['email', 'sms']);
```

### Assertions
```js
cy.get('input[value="email"]').should('be.checked');
cy.get('input[value="sms"]').should('not.be.checked');
```

---

## 🔘 Radio Buttons

Use `.check()` (ensures only one selected at a time).

### Select a Radio Option
```js
cy.get('input[type="radio"][value="male"]').check();
```

### Assertions for Radio Buttons
```js
cy.get('input[type="radio"][value="female"]').check();
cy.get('input[type="radio"][value="female"]').should('be.checked');
cy.get('input[type="radio"][value="male"]').should('not.be.checked');
```

---

## ⚡ Advanced Interactions

### Hidden Elements (Force)
```js
cy.get('input[type="checkbox"]').check({ force: true });
```

### Disabled Dropdowns (Workaround)
```js
cy.get('select#country').invoke('val', 'IN').trigger('change');
```

---

## ✅ Best Practices

- Prefer `.check()` / `.uncheck()` over `.click()` for checkboxes/radio.  
- Always use `.should()` assertions to verify state.  
- Use `{ force: true }` only when necessary (hidden elements).  
- For dynamic dropdowns, wait for options before selecting.

---

# 🔎 Summary

- **Dropdowns** → `.select('Text')` or `.select('Value')`  
- **Checkboxes** → `.check()` / `.uncheck()` (can pass arrays)  
- **Radio buttons** → `.check()` (exclusive selection)  
- **Force option** → `{ force: true }` for hidden elements  
- Combine with `.should()` assertions for reliability  

---