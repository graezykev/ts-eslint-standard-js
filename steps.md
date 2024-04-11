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
coverage
public
dist
pnpm-lock.yaml
pnpm-workspace.yaml
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
```