# 📂 File Upload in Cypress

Cypress does not have a built-in `upload()` command, so file upload can
be handled using plugins or native browser APIs.

------------------------------------------------------------------------

## 1. Using `cypress-file-upload` Plugin (Recommended)

1.  Install the plugin:

``` bash
npm install --save-dev cypress-file-upload
```

2.  Import it in `cypress/support/commands.js`:

``` js
import 'cypress-file-upload';
```

3.  Usage in test:

``` js
// Upload a file from fixtures folder
cy.get('input[type="file"]').attachFile('example.json');

// Upload with a custom name
cy.get('input[type="file"]').attachFile({ filePath: 'example.json', fileName: 'myFile.json' });

// Upload multiple files
cy.get('input[type="file"]').attachFile(['file1.json', 'file2.json']);
```

👉 By default, Cypress looks in the `cypress/fixtures/` folder for the
file.

------------------------------------------------------------------------

## 2. Without Plugin (Native Approach)

If you don't want to use a plugin, you can create a `File` object and
assign it to the input.

``` js
const fileName = 'example.json';

cy.fixture(fileName).then(fileContent => {
  cy.get('input[type="file"]').then(input => {
    const blob = new Blob([JSON.stringify(fileContent)], { type: 'application/json' });
    const file = new File([blob], fileName, { type: 'application/json' });
    const dataTransfer = new DataTransfer();
    dataTransfer.items.add(file);
    input[0].files = dataTransfer.files;
    input[0].dispatchEvent(new Event('change', { bubbles: true }));
  });
});
```

✅ Works without any dependency, but more verbose.

------------------------------------------------------------------------

## 3. Drag and Drop Uploads

Some apps use drag-and-drop instead of `<input type="file">`. You can
simulate this:

``` js
cy.fixture('example.json').then(fileContent => {
  cy.get('#dropzone').upload(
    { fileContent, fileName: 'example.json', mimeType: 'application/json' },
    { subjectType: 'drag-n-drop' }
  );
});
```

*(Requires `cypress-file-upload` as well).*

------------------------------------------------------------------------

## 📝 Summary

  -------------------------------------------------------------------------
  Method                    Pros                              Cons
  ------------------------- --------------------------------- -------------
  **cypress-file-upload**   ✅ Easy, concise, supports        ❌ Needs
                            multiple files                    plugin

  **Native fixture + File   ✅ No external dependency         ❌ Verbose
  API**                                                       

  **Drag & drop (plugin)**  ✅ Handles modern UIs             ❌ Needs
                                                              plugin
  -------------------------------------------------------------------------

------------------------------------------------------------------------

⚡ **Rule of Thumb**\
- Use `cypress-file-upload` for most cases.\
- Use native approach if you want **zero dependencies**.
