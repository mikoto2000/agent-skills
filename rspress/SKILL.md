---
name: rspress
description: Work with Rspress v2 projects. Use when creating or editing Rspress documentation, especially when deciding where files belong or writing Markdown/MDX content.
---

# Rspress v2

This skill assumes Rspress v2.

It defines only:

* common Rspress v2 directory structure
* Markdown/MDX conventions used by Rspress v2

Do not use this skill for plugin development, build troubleshooting, theme development, or Rsbuild configuration.

# Directory structure

A typical Rspress v2 project may look like:

```text
.
├── package.json
├── rspress.config.ts
├── docs/
│   ├── index.md
│   ├── guide/
│   │   └── index.md
│   └── public/
└── theme/
```

The actual documentation root is defined by the project configuration.

Do not assume `docs/` if the repository uses another root.

## `rspress.config.ts`

Rspress configuration is typically defined in:

```text
rspress.config.ts
```

Do not place documentation content in this file.

## Documentation root

Markdown and MDX documentation pages belong under the configured documentation root.

A common example is:

```text
docs/
```

Example:

```text
docs/
├── index.md
├── getting-started.md
├── guide/
│   ├── index.md
│   └── configuration.md
└── reference/
    └── api.md
```

Directory structure generally corresponds to URL structure.

For example:

```text
docs/index.md
docs/guide/index.md
docs/guide/configuration.md
```

typically represents pages conceptually corresponding to:

```text
/
/guide/
/guide/configuration
```

Follow the existing project structure rather than reorganizing documents unnecessarily.

## `public`

Static files that should be served directly can be placed under the documentation root's `public` directory.

Typical example:

```text
docs/
└── public/
    ├── logo.svg
    ├── images/
    │   └── architecture.png
    └── files/
        └── example.zip
```

Files under `public` are referenced from the site root.

For example:

```markdown
![Architecture](/images/architecture.png)
```

Do not include `public` itself in the URL.

Use the project's existing asset convention when one already exists.

## `theme`

Project-level theme customizations may be placed in:

```text
theme/
```

Do not place ordinary documentation pages under `theme/`.

Theme implementation is outside the scope of this skill.

# Markdown and MDX

Rspress v2 documentation is written primarily in Markdown and MDX.

Use `.md` for ordinary documentation where possible.

Use `.mdx` when JSX or React components are required.

## Headings

Use Markdown headings normally.

```markdown
# Page title

## Section

### Subsection
```

Prefer one top-level `#` heading for a page unless the existing project uses another convention.

Do not skip heading levels unnecessarily.

Prefer:

```markdown
## Section

### Detail
```

over:

```markdown
## Section

#### Detail
```

## Paragraphs

Separate paragraphs with a blank line.

```markdown
First paragraph.

Second paragraph.
```

Do not rely on source line breaks alone to create separate paragraphs.

## Emphasis

```markdown
**bold**

*italic*

`inline code`
```

Use inline code for:

* commands
* filenames
* configuration keys
* class names
* function names
* package names

when appropriate.

## Lists

Unordered list:

```markdown
- Alpha
- Beta
- Gamma
```

Ordered list:

```markdown
1. First
2. Second
3. Third
```

Nested lists must be indented consistently.

```markdown
- Parent
  - Child
  - Child
```

## Links (Anchor tags)

Use normal Markdown links.

```markdown
[Rspress](https://rspress.rs/)
```

For documentation pages within the same site, prefer relative links when practical.

```markdown
[Configuration](./configuration.md)
```

or:

```markdown
[Configuration](../guide/configuration.md)
```

Follow the existing project's link convention.

## Images

Use normal Markdown image syntax when no special component is required.

```markdown
![Description](./image.png)
```

For assets under `public`:

```markdown
![Description](/images/image.png)
```

Always provide meaningful alt text unless the image is purely decorative.

## Code blocks

Use fenced code blocks.

````markdown
```java
public class Example {
}
```
````

Specify the language when known.

Common examples:

````markdown
```ts
const value = 1;
```

```json
{
  "enabled": true
}
```

```bash
npm run build
```
````

Do not indent ordinary fenced code unnecessarily.

## Showing Markdown syntax literally

When documenting fenced code blocks themselves, use a longer outer fence.

Example:

`````markdown
````markdown
```plantuml
Alice -> Bob
```
````
`````

The outer fence must contain more backticks than the inner fence.

## Tables

Standard Markdown tables can be used.

```markdown
| Name | Description |
| --- | --- |
| Alpha | First item |
| Beta | Second item |
```

Keep tables reasonably narrow.

For complex content, prefer normal document sections over very wide tables.

## Blockquotes

```markdown
> This is a quote.
```

Do not use blockquotes merely for visual indentation.

## Horizontal rules

Use:

```markdown
---
```

only when a structural separator is genuinely useful.

Do not overuse horizontal rules between ordinary sections.

# Frontmatter

Rspress pages may contain YAML frontmatter at the beginning of the file.

Example:

```markdown
---
title: Getting Started
---

# Getting Started
```

Frontmatter must appear before page content.

Preserve existing frontmatter fields when editing a document.

Do not invent project-specific frontmatter fields unless their use is already established in the repository.

## Common page title behavior

If the page already uses frontmatter for the title, follow the project's existing convention regarding whether an explicit `#` heading is also present.

Do not automatically duplicate or remove titles.

# MDX

Use `.mdx` when the page needs JSX or React components.

Example:

```mdx
<MyComponent />

## Section

Normal Markdown can continue here.
```

MDX supports JavaScript expressions using `{}`.

```mdx
The result is {value}.
```

Because MDX treats JSX and JavaScript specially, characters that are harmless in plain Markdown may have different meaning in MDX.

Do not convert `.md` files to `.mdx` without a concrete need.

## JSX components

React components can appear directly in MDX.

```mdx
<Example foo="bar" />
```

Use the component conventions already established by the project.

Do not invent component names or imports without checking the existing codebase.

## HTML-like syntax

In MDX, tags may be interpreted as JSX.

For example:

```mdx
<div>Content</div>
```

Treat such constructs as JSX-compatible markup.

Do not assume arbitrary HTML syntax accepted elsewhere will necessarily compile unchanged in MDX.

# Escaping and literal examples

Documentation frequently needs to show syntax that Rspress or MDX would otherwise interpret.

Use fenced code blocks for literal examples.

Prefer:

````markdown
```text
<Component />
```
````

over trying to escape every character inline.

This is especially important when documenting:

* Markdown
* MDX
* JSX
* directives
* PlantUML
* configuration files

# File naming

Follow the existing repository convention.

Prefer predictable lowercase names unless the project uses another style.

Examples:

```text
getting-started.md
configuration.md
api-reference.md
```

Avoid gratuitous renames of existing pages.

For section landing pages, use the convention already present in the repository, commonly:

```text
guide/index.md
```

# Creating new pages

When adding a new documentation page:

1. identify the correct location under the documentation root
2. follow neighboring filenames
3. follow neighboring frontmatter conventions
4. follow neighboring heading style
5. use relative links consistent with nearby documents
6. place related static assets according to the existing project convention

Do not create new top-level directories when an appropriate existing section already exists.

# Editing existing pages

When editing documentation:

* preserve unrelated content
* preserve frontmatter
* preserve existing link style
* preserve local terminology
* preserve the established heading structure where possible
* avoid reformatting unrelated sections

Keep changes scoped to the requested documentation change.

# Rspress-specific extensions

A project may provide additional Markdown syntax through plugins.

Examples may include:

* reStructuredText-style directives
* PlantUML fences
* custom components
* additional remark or rehype processing

Do not infer their syntax from this skill.

Use the corresponding project-specific skill or existing documentation for those extensions.
