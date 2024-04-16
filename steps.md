# Steps

## Init Git

```sh
git init && \
echo 'node_modules' >> .gitignore
```

## Init Project

```sh
npm init -y
```

```sh
npm pkg set type="module"
```

## Install Dependencies

- TypeScript
- ESLint
- Lintting configs & plugins in **Standard Style**

```sh
npm install -D \
  typescript@\* \
  eslint@^8.57.0 \
  eslint-plugin-promise@^6.0.0 \
  eslint-plugin-import@^2.25.2 \
  eslint-plugin-n@^15.0.0 \
  @typescript-eslint/eslint-plugin@^7.0.1 \
  eslint-config-love@latest \
  typescript-eslint@^7.6.0 \
  globals@^15.0.0
```

## Init TypeScript

```sh
npx tsc --init
```

## Create ESLint configuration file

`eslint.config.js`:

```js
import globals from 'globals'
import tseslint from 'typescript-eslint'

import path from 'path'
import { fileURLToPath } from 'url'
import { FlatCompat } from '@eslint/eslintrc'
import pluginJs from '@eslint/js'

// mimic CommonJS variables -- not needed if using CommonJS
const __filename = fileURLToPath(import.meta.url)
const __dirname = path.dirname(__filename)
const compat = new FlatCompat({
  baseDirectory: __dirname,
  recommendedConfig: pluginJs.configs.recommended
})

export default [
  { files: ['**/*.js'], languageOptions: { sourceType: 'script' } },
  { languageOptions: { globals: globals.browser } },
  ...compat.extends('love'),
  ...tseslint.configs.recommended
] 
```

## Try to lint js file

```sh
npx eslint eslint.config.js
```

![alt text](images/image-8.png)

## Try to lint a `.ts` file

create a `index.ts`:

```js
const x = {
    a: "b",
    b: 123
};
```

```sh
npx eslint index.ts
```

that also works.

![alt text](images/image-9.png)

## Configure linting commands

edit `package.json`

```diff
{
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
+   "lint": "eslint ."
```

don't forget to exclude some files that should not be lint, create a `.eslintignore`:

```txt
node_modules
test
coverage
public
dist
```

now try the script

```sh
npm run lint
```

all problems in your js & ts files will be shown.

![alt text](images/image-10.png)

## Configure formatting commands

there are some problems can be fixed by the command `eslint --fix`

edit `package.json`

```diff
{
  "scripts": {
+ "format": "eslint --fix ."
```

now try to format it

```sh
npm run format
```

since some problems have been fixed, only those what can't be fixed will show

![alt text](images/image-11.png)

> Once again, formatting codes using ESLint is self-asserting

## VS Code editor setting

Configure your VS Code editor, to auto insert 2 spaces after you click the `Tab` key, instead of inserting a real `Tab`, following the rule defined by [Standard JS](https://standardjs.com/rules)

create the configure file

```sh
mkdir .vscode && touch .vscode/settings.json
```

edit `.vscode/settings.json`:

```json
{
  "editor.tabSize": 2,
  "editor.insertSpaces": true,
  "editor.detectIndentation": false
}
```

## VS Code ESLint extension

Using `npm run lint` and `npm run format` to check and format your code can be a very laborious job, but you can do it by integrating with ESLint VS Code extension.

search `dbaeumer.vscode-eslint` on the Extensions pannel and install it.

> At the time I write this, I'm using the v3.0.5 (pre-release) of this extension. Other versions may have some unknown issues.

edit `.vscode/settings.json` to make sure it works:

```diff
{
  "editor.tabSize": 2,
  "editor.insertSpaces": true,
- "editor.detectIndentation": false
+ "editor.detectIndentation": false,
+ "eslint.enable": true
}
```

Reload your VS Code window, you'll see errors detected by ESLint showing on the editor while you are editing the code, with **wavy line** highlighting the errors, hovering on the wavy line will show the pop-up of the error details.

![alt text](images/image-00.png)

![alt text](images/image-01.png)

## Auto format on save

this extension can also fix your code linting issues automatically.

edit `.vscode/settings.json`:

```diff
- "eslint.enable": true
+ "eslint.enable": true,
+  "eslint.format.enable": true,
+  "editor.formatOnSave": true,
+  "editor.defaultFormatter": "dbaeumer.vscode-eslint"
}
```

Now every time you edit your code, and pres `command + s` to save it, those problems can be automatically fixed will be fixed.

![alt text](images/image-02.png)

the semicolon will disappear after you press `command + s` to save.

## Linting React and React Hooks

```sh
npm install react react-dom
```

```sh
npm install -D @types/react @types/react-dom
```

create `react-code-1.tsx`:

```js
import React, { useContext } from "react";

const SignupButton = () => {
  const handleSignup = () => {
    alert("Sign up successful!");
  };

  return (
    <button onClick={handleSignup} className="signup-button">
      Sign Up
    </button>
  );
};

const MyComponent = (props) => {
  return (<div id={props.id} />)
};
```

There is some error in the editor.

![alt text](images/image.png)

That's because JSX syntax is not allowed yet in Typescript configuration, edit `tsconfig.json`:

```diff
-    // "jsx": "preserve",                                /* Specify what JSX code is generated. */
+    "jsx": "react",                                /* Specify what JSX code is generated. */
```

The error above will disappear.

Now lint the `tsx` file with cli:

```sh
npx eslint react-code-1.tsx
```

you'll find 14 problems:

![alt text](images/image-1.png)

create `react-code-2.jsx` with the same code as `react-code-1.tsx`, and lint it:

```sh
npx eslint react-code-2.jsx
```

you'll see some problems but not related to real code problems.

![alt text](images/image-2.png)

that's because `.jsx` file is not specified in the ESLint configuration, let's configure it in `eslint.config.js`:

```diff
export default [
-  { files: ['**/*.js'], languageOptions: { sourceType: 'script' } },
+  { files: ['**/*.{js,ts,jsx,tsx}'], languageOptions: { sourceType: 'script' } },
  { languageOptions: { globals: globals.browser } },
```

lint it again and you'll get **14 errors** same as `react-code-1.tsx`.

![alt text](images/image-3.png)

these errors as only JS problems not specific to best practise of React and React Hooks, we need to do more.

## Install React & React Hooks ESLint plugins

```sh
npm install -D eslint-plugin-react eslint-plugin-react-hooks
```

## config React ESLint plugin

first, config `eslint-plugin-react` in `eslint.config.js` with a new line:

```diff
export default [
  { files: ['**/*.{js,ts,jsx,tsx}'], languageOptions: { sourceType: 'script' } },
  { languageOptions: { globals: globals.browser } },
+  ...compat.extends('plugin:react/recommended'),
```

now run `npx eslint react-code-1.tsx` you'll find 15 problems, in contrast with the previous 14 problems, because 1 more issue related to React which is stipulated in the plugin we just added, has been detected.

![alt text](images/image-4.png)

## defined or edit your own rule(s)

Ok let take one further step, edit `eslint.config.js` to extend your self-customed rule:

```diff
export default [
  { files: ['**/*.{js,ts,jsx,tsx}'], languageOptions: { sourceType: 'script' } },
  { languageOptions: { globals: globals.browser } },
  ...compat.extends('plugin:react/recommended'),
  ...tseslint.configs.recommended,
  ...compat.extends('love'),
+  {
+    rules: {
+      'react/destructuring-assignment': ['warn', 'always']
+    }
+  }
]
```

run `npx eslint react-code-1.tsx`, you'll get 1 more problem which is define by you.

you can defined or modify any rule as much as you can to tailor your team's rules here.

![alt text](images/image-5.png)

## config React Hooks ESLint plugin

let's now integrate the best best practise rules of React Hooks.

edit `react-code-1.tsx`:

```diff
-import React from "react";
+import React, { useContext } from "react";

...

const MyComponent = (props) => {

+  if (true) {
+    const theme = useContext(ThemeContext);
+  }

  return (<div id={props.id} />)
};
```

now lint it, and you'll find some errors, but none are related to React Hooks.

![alt text](images/image-6.png)

so you need to edit the ESLint config with 1 more line:

```diff
  ...compat.extends('plugin:react/recommended'),
+  ...compat.extends('plugin:react-hooks/recommended'),
  ...tseslint.configs.recommended,
```

lint `react-code-1.tsx` again you'll get errors related to React Hooks.

![alt text](images/image-7.png)

## Conclusion

Now you have a somewhat robust toolchain of linting and formatting your code.

Your Codes of:

**TypeScript and JavaScript**

**React and React Hooks**

will all be ensured by:

**Checking and Formatting CLI scripts**

**Checking and Formatting editor tools**
