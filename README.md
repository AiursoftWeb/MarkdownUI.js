# MarkdownUI.js

[![Pipeline](https://badge-proxy.aiursoft.com/aiursoft/MarkdownUI.js/badges/master/pipeline.svg)](https://gitlab.aiursoft.com/aiursoft/MarkdownUI.js/-/pipelines) [![npm](https://img.shields.io/npm/v/@aiursoft/uistack-markdown-ui)](https://www.npmjs.com/package/@aiursoft/uistack-markdown-ui)

Shared Markdown rendering, editing and reading components.

## Install

```bash
npm install @aiursoft/uistack-markdown-ui
```

## Usage

```js
import { renderMarkdown } from '@aiursoft/uistack-markdown-ui';

preview.innerHTML = renderMarkdown('# Hello');
```

See [usage and API](docs/usage.md) for editors, readers, image uploads and print integration.

## Development

```bash
npm ci
npm run check
npm pack
```
