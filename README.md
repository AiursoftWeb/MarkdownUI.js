<a name="markdownuijs"></a>

# Aiursoft MarkdownUI.js

[![Pipeline](https://badge-proxy.aiursoft.com/aiursoft/MarkdownUI.js/badges/master/pipeline.svg)](https://gitlab.aiursoft.com/aiursoft/MarkdownUI.js/-/pipelines) [![npm](https://img.shields.io/npm/v/@aiursoft/uistack-markdown-ui)](https://www.npmjs.com/package/@aiursoft/uistack-markdown-ui)

Shared Markdown rendering, editing and reading components.

## Overview

### What is MarkdownUI.js?

Shared Markdown rendering, editing and reading components.

<a name="install"></a>

## Installation

Install the package in your project:

```bash
npm install @aiursoft/uistack-markdown-ui
```

The package provides ESM, CommonJS, TypeScript declarations and a browser IIFE build (`AiursoftMarkdownUi`). Browser editing and reading require the DOM; `renderMarkdown` can also be used from a Node.js module. Monaco, Mermaid and MathJax are supplied by the application when those integrations are needed.

## Usage

Save the following as `index.html` beside `node_modules/` and serve that directory over HTTP with your existing static-file server:

```html
<!doctype html>
<html lang="en">
  <meta charset="utf-8">
  <title>MarkdownUI example</title>
  <body>
    <div id="preview"></div>
    <script src="./node_modules/@aiursoft/uistack-markdown-ui/dist/index.global.js"></script>
    <script>
      const preview = document.getElementById('preview');
      preview.innerHTML = AiursoftMarkdownUi.renderMarkdown('# Hello');
    </script>
  </body>
</html>
```

**Bundler import**

In an application bundle, create or select the preview element before assigning the rendered HTML:

```js
import { renderMarkdown } from '@aiursoft/uistack-markdown-ui';

const preview = document.getElementById('preview');
preview.innerHTML = renderMarkdown('# Hello');
```

See [usage and API](docs/usage.md) for editors, readers, image uploads and print integration.

### Complete Markdown editor

`createMarkdownEditor` owns the common editor behavior while the application remains
responsible for loading Monaco. Passing a loader instead of importing Monaco keeps it
out of application bundles and works with the standard Monaco AMD loader.

```js
import { createMarkdownEditor, loadMonacoFromAmd } from "@aiursoft/uistack-markdown-ui";

const markdownEditor = await createMarkdownEditor({
  editorContainer: document.querySelector("#markdown-editor"),
  textarea: document.querySelector("#markdown-source"),
  previewContainer: document.querySelector("#markdown-preview"),
  editorPane: document.querySelector("#editor-pane"),
  previewPane: document.querySelector("#preview-pane"),
  form: document.querySelector("form"),
  loadMonaco: () => loadMonacoFromAmd(),
  theme: "vs-dark",
  uploadUrl: "/api/files/upload",
  initialViewMode: "split",
  viewModeStorageKey: "markdown-view-mode",
  viewModeControls: [
    { element: document.querySelector("#edit-mode"), mode: "editor" },
    { element: document.querySelector("#split-mode"), mode: "split" },
    { element: document.querySelector("#preview-mode"), mode: "preview" }
  ],
  hljs,
  mermaid,
  MathJax,
  onSave: markdown => saveMarkdown(markdown),
  onInitializationError: error => console.error("Monaco failed to initialize", error),
  onPreviewError: error => console.error("Markdown preview failed", error)
});

await markdownEditor.setViewMode("preview");
markdownEditor.syncTextarea();
markdownEditor.setValue("# Updated");
markdownEditor.focus();
markdownEditor.dispose();
```

The controller initializes Monaco for Markdown, keeps the original textarea synchronized,
debounces secure preview rendering, enhances highlight.js/Mermaid/MathJax, supports image
paste and drop, and applies consistent professional Monaco defaults (14px Cascadia Code
font stack, ligatures, semantic highlighting and comfortable vertical padding). Applications
can still override individual Monaco settings through `editorOptions`.

It also registers consistent Markdown shortcuts:

- <kbd>Ctrl/Cmd+B</kbd>: bold
- <kbd>Ctrl/Cmd+I</kbd>: italic
- <kbd>Ctrl/Cmd+K</kbd>: inline or fenced code
- <kbd>Ctrl/Cmd+L</kbd>: link
- <kbd>Ctrl/Cmd+1</kbd> through <kbd>Ctrl/Cmd+6</kbd>: headings
- <kbd>Ctrl/Cmd+S</kbd>: `onSave`
- <kbd>Enter</kbd>: continue bullet and ordered lists

If Monaco cannot be loaded, the original textarea becomes visible and continues to drive
the same preview. Applications therefore use one fallback policy instead of implementing
their own. Preview enhancement errors are reported through `onPreviewError` and never
replace a working Monaco editor. `onError` remains as a deprecated compatibility callback
for both error categories.

### Reading pages

```js
import {
  initializeMarkdownReader,
  printMarkdown
} from "@aiursoft/uistack-markdown-ui";

await initializeMarkdownReader({
  container: ".markdown-body",
  hljs,
  mermaid,
  MathJax
});

printButton.addEventListener("click", () =>
  printMarkdown({ container: ".markdown-body", hljs, mermaid, MathJax }));
```

Only code blocks inside the supplied container are highlighted. Mermaid is always
initialized in strict security mode. The reader also applies the shared responsive
Bootstrap table treatment and opens rendered links with `noopener noreferrer`.

### Image paste and drop

```js
import { attachImageUpload } from "@aiursoft/uistack-markdown-ui";

const uploads = attachImageUpload({
  editor,
  eventTarget: editorContainer.parentElement,
  uploadUrl: "/api/files/upload",
  onError: (error, file) => console.error(`Failed to upload ${file.name}`, error)
});

// Call uploads.dispose() when the editor is destroyed.
```

The default queue uploads three files concurrently and retries HTTP 429 responses up
to five times. Clipboard images are accepted from both `clipboardData.files` and
`clipboardData.items`, and are captured before Monaco consumes the paste event. Use
the parent of Monaco's host container as `eventTarget`; `createMarkdownEditor()` does
this automatically. Use `getImageUrl` when an endpoint does not return
`internetPath` or `InternetPath`.

### Aiursoft application contract

MarkToHtml, MoongladeV2, EmployeeCenter and Kanban use the global IIFE build and the
same `createMarkdownEditor` contract. Monaco, live preview, highlight.js, Mermaid,
MathJax, Markdown shortcuts, image paste/drop, editor/split/preview modes and the
textarea fallback are required capabilities for these full editors.

DocsViewer and HowToCookViewer are reader-only consumers. Their Markdown containers
use `initializeMarkdownReader` for highlighting, Mermaid, MathJax, responsive tables
and safe external-link behavior.

Translate is an intentional product-level exception. It keeps its lightweight
CodeMirror Markdown input because translation does not need code highlighting,
Mermaid, MathJax, uploads or the full editing toolbar. It must not be treated as an
incomplete migration.

## Configuration

Configuration is passed as JavaScript options; the package has no configuration file or environment variables. The complete editor example above shows the application-owned DOM elements, Monaco loader, view controls, upload endpoint and callbacks.

**Rendering**

`renderMarkdown(markdown, options)` and `createMarkdownIt(options)` accept `breaks` (default `false`), `linkify` (`true`) and `typographer` (`true`). Native HTML is always disabled. Mermaid processing uses strict security mode. Math and diagram rendering require the corresponding application-supplied integrations; syntax highlighting is included in the renderer.

**Editor options and lifecycle**

`editorContainer`, `textarea` and `previewContainer` are required. Supply `monaco` or `loadMonaco` to create the editor; initialization failure reveals the textarea fallback. `theme` defaults to `vs`, `debounceMs` to `200`, and `initialViewMode` to `split`. `editorOptions` overrides Monaco editor settings. `form` defaults to the textarea's form; `editorPane` and `previewPane` default to their containers. `viewModeControls` defaults to an empty list; when `viewModeStorageKey` is supplied, a valid saved view mode overrides the initial mode. `markdownOptions`, `mermaidTheme`, `hljs`, `mermaid` and `MathJax` configure preview integration.

`onSave`, `onChange` and `onPreviewRendered` report edits and rendering. Use `onInitializationError` and `onPreviewError` for their separate failure categories; deprecated `onError` remains a compatibility callback. An optional `uploadUrl` enables image uploading, with `imageUploadOptions` for its settings. Call the returned controller's `dispose()` when the editor is removed.

**Reader options**

`initializeMarkdownReader` and `printMarkdown` take a required `container` selector, element or iterable, plus optional highlighter, Mermaid and MathJax dependencies (otherwise resolved from globals). `decorateTables` and `openLinksInNewTab` default to enabled. Tables use `table`, `table-striped` and `table-bordered` classes with a `table-responsive` wrapper; override `tableClassNames` or `tableWrapperClassName` when needed. Links opened in a new tab receive `noopener noreferrer`. `mermaidTheme` selects the diagram theme.

**Image upload options**

`editor` and `uploadUrl` are required for `attachImageUpload`. `eventTarget` defaults to the editor DOM node when available; without either target no DOM paste/drop listeners are attached. A Monaco host parent captures paste events before the editor. `fieldName` defaults to `file`, `concurrency` to `3` and `maxRetries` to `5`. Only HTTP 429 is retried, using `Retry-After` when present. Supply `fetch` to replace the fetch implementation, `getImageUrl` to customize response parsing (defaults to `internetPath` or `InternetPath`), and `onError` to report failures. Dispose the upload controller with the editor. The host provides and secures the upload endpoint.

## Development

Build with Node.js 22.18+ (22.x), 24.11+ (24.x), or 26+. The current build is verified with Node.js 24.

From the repository root, install the locked dependencies, run the existing checks and create a package:

```bash
npm ci
npm run check
npm pack
```

The existing checks run TypeScript checking, Vitest coverage tests and the production build. `npm pack` creates a local archive; it does not publish a release.

## Documentation

See [usage and API](docs/usage.md) for editors, readers, image uploads and print integration. Exported option types and lifecycle contracts are in [src/index.ts](src/index.ts).

## How to contribute

There are many ways to contribute to the project: logging bugs, submitting pull requests, reporting issues, and creating suggestions.

Even if you have push rights on the repository, you should create a personal fork and create feature branches there when you need them. This keeps the main repository clean and your workflow cruft out of sight.

We're also interested in your feedback on the future of this project. You can submit a suggestion or feature request through the issue tracker. To make this process more effective, we're asking that these include more information to help define them more clearly.

## License

Licensed under the [MIT License](LICENSE).
