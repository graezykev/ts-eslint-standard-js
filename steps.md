# Steps

## Init project

```sh
pnpm init -y
```

## Install dependencies

- TypeScript
- ESLint
- Lintting configs & plugins in **Standard Style**

```sh
pnpm add -D \
  typescript@\* \
  eslint@^8.0.1 \
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
pnpm exec tsc --init
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

## Try lintting js file

```sh
pnpm exec eslint eslint.config.js
```

## Try lintting ts file

`index.ts`:

```js
const x = {
    a: "b",
    b: 123
};
```

```sh
pnpm exec eslint index.ts
```

## Configure lintting commands

`package.json`

```json
{
  "scripts": {
    "lint": "eslint ."
  }
}
```

`.eslintignore`

```txt
node_modules
test
coverage
public
dist
```

```sh
pnpm eslint
```

## Configure formatting commands

`package.json`

```json
{
  "scripts": {
    "format": "eslint --fix ."
  }
}
```

```sh
pnpm format
```

## VS Code editor setting


```sh
mkdir .vscode && touch .vscode/settings.json
```

`.vscode/settings.json`:

```json
{
  "editor.tabSize": 2,
  "editor.insertSpaces": true,
  "editor.detectIndentation": false
}
```

# VS Code ESLint extension

search and install `dbaeumer.vscode-eslint` on the Extensions pannel

show error on editing

## Auto format on save

`.vscode/settings.json`:

```json
{
  "eslint.enable": true,
  "eslint.format.enable": true,
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "dbaeumer.vscode-eslint"
}
```

## Git ignored files

```sh
touch .gitignore
```

```txt
node_modules
dist
```

## Linting React and React Hooks

```sh
pnpm add react react-dom
```

```sh
pnpm add @types/react @types/react-dom
```

create `react-code-1.tsx`:

![alt text](image.png)

edit `tsconfig.json`:

```diff
-    // "jsx": "preserve",                                /* Specify what JSX code is generated. */
+    "jsx": "react",                                /* Specify what JSX code is generated. */
```

The error above will disappear.

now lint the `tsx` file:

```sh
npx eslint react-code-1.tsx
```

you'll find 14 problems:

![alt text](image-1.png)

create `react-code-2.jsx` with the same code:

now lint the `jsx` file:

```sh
npx eslint react-code-2.jsx
```

![alt text](image-2.png)

because `.jsx` file is not define in the ESLint config, let's configure it in `eslint.config.js`:

```diff
export default [
-  { files: ['**/*.js'], languageOptions: { sourceType: 'script' } },
+  { files: ['**/*.{js,ts,jsx,tsx}'], languageOptions: { sourceType: 'script' } },
  { languageOptions: { globals: globals.browser } },
```

lint it again and you'll get 14 errors same to `react-code-1.tsx`.

![alt text](image-3.png)

```sh
pnpm add -D eslint-plugin-react eslint-plugin-react-hooks
```

```diff
export default [
  { files: ['**/*.{js,ts,jsx,tsx}'], languageOptions: { sourceType: 'script' } },
  { languageOptions: { globals: globals.browser } },
+  ...compat.extends('plugin:react/recommended'),
```

run `npx eslint react-code-1.tsx` again you'll find 15 problems, in contrast with the previous 14 problems, because 1 more rule is being applied in this file.

![alt text](image-4.png)

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

run `npx eslint react-code-1.tsx`, again you'll get 1 more problems which is define by you.

![alt text](image-5.png)

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

now lint it you'll find some errors but none related with react hooks.

![alt text](image-6.png)

edit the ESLint config:

```diff
  ...compat.extends('plugin:react/recommended'),
+  ...compat.extends('plugin:react-hooks/recommended'),
  ...tseslint.configs.recommended,
```

lint again now you'll get error related to react hooks.

![alt text](image-7.png)