# Creating and Publishing Plugins for A.I Desk

This guide explains how to create, package, and publish plugins for A.I Desk. Each plugin consists of an `index.js` file (the plugin logic) and a `package.json` file (the plugin metadata and configuration).

---

## 1. Plugin Structure

Each plugin must include:

- `index.js`: The main JavaScript file implementing your plugin logic.
- `package.json`: Metadata, dependencies, and configuration for your plugin.

### Writing `package.json`

The `package.json` file describes your plugin and its requirements. It should include:
- `name`: The unique name of your plugin.
- `author`: Your name or handle.
- `description`: A short summary of what your plugin does.
- `version`: The plugin version (use [semver](https://semver.org/)).
- `dependencies`: Any npm packages your plugin needs.
- `parameters`: An array of user-configurable settings. Each parameter should have:
  - `id`: Unique identifier for the parameter.
  - `name`: Display name.
  - `description`: What the parameter is for.
  - `type`: The HTML input type (e.g., `"password"`, `"text"`, `"email"`, `"number"`, `"tel"`, `"url"`).

### Example: `package.json`

```json
{
    "name": "ChatGPTFree",
    "author": "Florestan",
    "description": "A plugin to interact with the ChatGPT without an API token.",
    "version": "1.0.0",
    "dependencies": {
        "openai-free": "^1.0.1"
    },
    "parameters": [
        {
            "id": "username",
            "name": "User name",
            "description": "Login name to ChatGpt",
            "type": "password"
        },
        {
            "id": "password",
            "name": "Password",
            "description": "Your secret password",
            "type": "password"
        }
    ]
}
```


### Writing `index.js`

The `index.js` file contains your plugin's logic. It should export functions that A.I Desk can call. The functions are:

- **`prompt(settings, conversationHistory, model)`**:  
  Sends a prompt to an AI model and returns the response.  
  - **settings**: An object where each key is the `id` of a parameter defined in your `package.json` and each value is what the user entered for that parameter. For example, if your `package.json` defines a parameter with `"id": "username"`, then `settings.username` will contain the user's input for that field.  
  - **conversationHistory**: An array of objects representing the conversation so far. Each object has a `role` property (`'user'` or `'assistant'`) and a `content` property (the message text). Example:  
    ```js
    [
      { role: 'user', content: 'Hello!' },
      { role: 'assistant', content: 'Hi, how can I help you?' }
    ]
    ```
  - **model**: The ID of the model to use (as a string). This is one of the models returned by your `listModels` function.

- **`listModels(settings)`**:  
  Returns a list of available models for your plugin, using the provided settings.  
  - **Return Value**: An array of objects, where each object represents a model. Each object consist of:
    - `id`: A unique identifier for the model (required).
    - `name`: A human-readable name for the model (optional).  
  Example return value:  
  ```json
  [
      { "id": "gpt-4", "name": "GPT-4" },
      { "id": "gpt-3.5", "name": "GPT-3.5 Turbo" }
  ]
  ```

- **`getDefaultModuleID()`** (optional):  
  Returns the string ID of the default model. This function is useful if your plugin has a preferred model that should appear at the beginning of the list of models and have "(Default)" appended to its name.  
  - **Return Value**: A string representing the default model ID.  
  Example return value:  
  ```json
  "gpt-4o"
  ```

### Hints

- You can use any npm packages declared in your `package.json` dependencies.
- `axios` is already included for making HTTP requests.
- Use the `settings` object to access user-provided parameters, such as API tokens or other configuration values.

---

### Example: `index.js`

```javascript
const axios = require('axios');

module.exports = {
    async prompt(settings, conversationHistory, model = "gpt-4") {
        try {
            const response = await axios.post(
                'https://api.openai.com/v1/chat/completions',
                {
                    model: model,
                    messages: conversationHistory,
                },
                {
                    headers: {
                        'Authorization': `Bearer ${settings.apiToken}`,
                        'Content-Type': 'application/json',
                    },
                }
            );
            return response.data.choices[0].message.content;
        } catch (error) {
            throw new Error(
                "Failed to generate a response from ChatGPT: " +
                (error?.response?.data?.error?.message || error?.message || "Unknown error")
            );
        }
    },

    async listModels(settings) {
        try {
            const response = await axios.get(
                'https://api.openai.com/v1/models',
                {
                    headers: {
                        'Authorization': `Bearer ${settings.apiToken}`,
                    },
                }
            );
            return response.data.data.map(model => ({
                id: model.id,
                name: model.name || model.id // Use id as fallback if name is not provided
            }));
        } catch (error) {
            throw new Error("Failed to fetch models from OpenAI API.");
        }
    },

    getDefaultModuleID() {
        return "gpt-4o";
    }
};
```

---

## 2. Packaging Your Plugin

1. Place your `index.js` and `package.json` in the same folder.
2. Select both files and compress them into a `.zip` archive.
3. Rename the archive, changing the extension from `.zip` to `.ai` (for example, `myplugin.ai`).

---

## 3. Publishing Your Plugin

1. **Clone the Repository**

   ```sh
   git clone <repository-url>
   cd ai-deck-store
   ```

2. **Add Your Plugin**

   - Copy your `.ai` file to the `plugins/community` folder.

3. **Update `pluginlist.json`**

   - Open `pluginlist.json`.
   - Add an entry for your plugin, for example:

     ```json
     {
         "name": "ChatGPTFree",
         "author": "Florestan",
         "description": "A plugin to interact with the ChatGPT without an API token.",
         "version": "1.0.0",
         "filename": "chatgptfree.ai"
     }
     ```

---

## 4. Committing and Pushing Your Changes

```sh
git add plugins/community/myplugin.ai pluginlist.json
git commit -m "Add MyPlugin plugin"
git push
```

---

## 5. Submitting a Pull Request

- Open a pull request on GitHub to propose your changes for review and inclusion.

---

