---
description: Build and use custom editors to support your content editors.
---

# Custom Editors

## How to write your own editor

Custom editors are enabling developers to replace the default editors with HTML5 applications so that the editing experience of the Squidex Web App can be customized.

Technically speaking a UI editor lives in a sandboxed iframe,which interacts with the web application through a small SDK using messaging. This SDK is a proxy of the Angular [ControlValueAccessor](https://angular.io/api/forms/ControlValueAccessor), without having the dependencies to Angular itself.

![Define Editor URL](../../.gitbook/assets/custom-editors.png)

Lets see how the code looks like:

```markup
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">

    <!-- Load the editor sdk from the local folder or https://cloud.squidex.io/scripts/editor-sdk.js -->
    <script src="editor-sdk.js"></script>
    <script src="https://cdn.ckeditor.com/ckeditor5/10.0.0/classic/ckeditor.js"></script>

    <style>
        .ck-editor__editable {
            min-height: 250px;
        }
    </style>
</head>

<body>
    <textarea name="content" id="editor"></textarea>

    <script>
        var element = document.getElementById('editor');
        ClassicEditor
            .create(element)
            .catch(error => {
                console.error(error);
            })
            .then(editor => {
                // When the field is instantiated it notified the UI that it has been loaded.
                var field = new SquidexFormField();
                // Handle the value change event and set the text to the editor.
                field.onValueChanged(function (value) {
                    if (value) {
                        editor.setData(value);
                    }
                });
                // Disable the editor when it should be disabled.
                field.onDisabled(function (disabled) {
                    editor.set('isReadOnly', disabled);
                });
                editor.model.document.on('change', function () {
                    var data = editor.getData();
                    // Notify the UI that the value has been changed. Will be used to trigger validation.
                    field.valueChanged(data);
                });
                editor.ui.focusTracker.on('change:isFocused', function (event, name, isFocused) {
                    if (!isFocused) {
                        // Notify the UI that the value has been touched.
                        field.touched();
                    }
                });
            });
    </script>
</body>

</html>
```

You just have to reference the editor SDK and handle the events. You also have to push the current value to the web application whenever it changes. Validation will happen automatically then.

## API

The \``SquidexFormField` class is the entry point to your editor. 

Create a new instance when your editor is initialized.

### Methods

| Name | Description |
| :--- | :--- |
| `editor.getValue()` | Gets the current value of the field. |
| `editor.getContext()` | Gets the current context information. More about that later. |
| `editor.getFormValue()` | Gets the current value of the content form. Can be used to access the values of other fields. |
| `editor.touched()` | Notifies the control container that the editor has been touched, must be called when your custom editor looses the focus. |
| `editor.clean()` | Cleanup the editor. Usually it is not needed to call this method. |
| `editor.onInit(callback)` | Registers the init handler. This callback is invoked once the messaging communication with the management UI is established. After the callback is invoked you get retrieve values with the get methods. The context object will be passed to the callback. |
| `editor.onDisabled(callback)` | Registers the disabled callback. This callback is invoked whenever the editor should either be enabled or disabled. A boolean value will be passed with either `true` \(disabled\) or `false` \(enabled\). |
| `editor.onValueChanged(callback)` | Registers the value changed callback. This callback is invoked whenever the value of the field has changed. The value will be passed to the callback as argument. |
| `editor.onFormValueChanged(callback)` | Registers the value changed callback. This callback is invoked whenever the value of the content form has changed. The value will be passed to the callback as argument. |

### Context

The context object contains application information, such as the username and access token.

Example:

```javascript
{
  "user": {
    "user": {
      "id_token": "TOKEN",
      "session_state": "TOKEN",
      "access_token": "TOKEN", // Access Token
      "token_type": "Bearer",  // Access Token Type
      "scope": "openid profile email squidex-profile role permissions squidex-api",
      "profile": {
        "s_hash": "Wn3eHEjfi65aLx-KioJ53g",
        "sid": "-S7htcpBlnhNKfBXLhl1rg",
        "sub": "5dc32104ebc77a363cca0e0c", // User Id
        "auth_time": 1573240790,
        "idp": "Google",
        "amr": [
          "external"
        ],
        "urn:squidex:name": "USERNAME",
        "urn:squidex:picture": "URL",
        "urn:squidex:permissions": "squidex.admin.*",
        "email": "hello@quidex.io",
        "email_verified": false
      },
      "expires_at": 1573405262
    }
  },
  "apiUrl": "http://localhost:5000/api"
}
```

You can use `apiUrl`, `access_token` and `token_type` to retrieve additional information from the API, for example when you build a special editor to manage references or assets.  
  
Use the sample editor [https://cloud.squidex.io/scripts/context-editor.html](http://localhost:5000/scripts/context-editor.html) to view your context.

## All Examples

Also, we have more example you can use them on your apps.

### 1. Simple CKE Editor

Reference: [https://squidex.github.io/squidex-samples/editors/cke-simple.html](https://squidex.github.io/squidex-samples/editors/cke-simple.html)

![CKE Editor](../../.gitbook/assets/cke.png)

Clone the sample and configure the CKE editor as you need it.

### 2. Country selector

Reference: [https://squidex.github.io/squidex-samples/editors/country-selector.html](https://squidex.github.io/squidex-samples/editors/country-selector.html)

![Country Selector](../../.gitbook/assets/country-selector.gif)

### 3. Product taxonomy

Reference: [https://squidex.github.io/squidex-samples/editors/tags-category.html](https://squidex.github.io/squidex-samples/editors/tags-category.html)

The data format is a list of url like paths for each product category that will be converted to a tree strucuture.

```javascript
[
  "/laptops-and-netbooks/thinkpad-x-series-chromebook-laptops/",
  "/laptops-and-netbooks/thinkpad-edge-laptops/thinkpad-edge-e330/",
  "/laptops-and-netbooks/ideapad-s-series-netbooks/ideapad-s210-notebook/",
  "/tablets/a-series/a2109-tablet/",
  "/servers/thinkserver/rs110/6438/",
  "/desktops-and-all-in-ones/thinkcentre-m-series-desktops/m715q/10m2/",
  "/phones/a-series/a328-smartphone/"
]
```

![Product taxonomy](../../.gitbook/assets/product-taxonomy.gif)

### 4. JSON Tree

Reference: [https://squidex.github.io/squidex-samples/editors/jstree-editor.html](https://squidex.github.io/squidex-samples/editors/jstree-editor.html)

Create a visual tree for a JSON object.

![JSON Tree](../../.gitbook/assets/jstree-editor.png)

