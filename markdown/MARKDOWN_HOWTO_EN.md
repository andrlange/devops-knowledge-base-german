# Sprache/Language : [DE](MARKDOWN_HOWTO.md) | [EN](MARKDOWN_HOWTO_EN.md)
# ✍️ Markdown HOWTO

A compact introduction to **Markdown**, the preferred format for this knowledge collection.

---

## 🧾 What is Markdown?

**Markdown** is a simple markup language that allows you to write structured text like headings, lists, code, or tables – readable in plain text and simultaneously displayable as HTML.

---

## 🚀 Use Cases

Markdown is excellent for:

- Documentation in Git repositories (`README.md`, How-Tos, Guides)
- Wikis and internal knowledge collections
- Blog posts, change logs, or API descriptions
- Formatted notes, e.g., in IntelliJ, VS Code, GitHub, GitLab, and many more

---

## 🧪 Examples & Snippets

### Headings

```markdown
# H1
## H2
### H3
```

### Lists
```markdown
- Point 1
- Point 2
    - Sub-point

1. First
2. Second
```

### Code & Commands
Inline: `kubectl get pods`

Code block:
<pre> ```bash 

kubectl get pods -n my-namespace 
``` </pre>

Result:
```bash

kubectl get pods -n my-namespace 
```

### Links & Images
```markdown
[Link text](https://example.com)

![Alt text](images/logo.png)
```

### 🧰 Formatting
| Style           | Markdown Example  | Result     |
| --------------- | ----------------- | ---------- |
| Bold            | `**Text**`        | **Text**   |
| Italic          | `*Text*`          | *Text*     |
| Bold + Italic   | `***Text***`      | ***Text*** |
| Strikethrough   | `~~Text~~`        | ~~Text~~   |
| Quote           | `> Quote`         | > Quote    |
| Divider         | `---` or `***`    | ---        |

### 📊 Tables

| Name       | Language  | Tool      |
|------------|-----------|-----------|
| Service A  | Kotlin    | Spring    |
| UI Frontend| Dart      | Flutter   |


### Highlighting with BlockQuotes for Tips etc.

Blockquote (quote block), and this is how you write it in Markdown:

```markdown
> ✏️ **Tip:** Many Markdown editors (e.g., VS Code, Obsidian) show you a live preview. On GitHub or GitLab, Markdown is automatically rendered.
```
Result:
> ✏️ **Tip:** Many Markdown editors (e.g., VS Code, Obsidian) show you a live preview. On GitHub or GitLab, Markdown is automatically rendered.

💡 Multi-line tips or notes can be written simply with multiple > lines:
```markdown
> ⚠️ **Note:**
> On GitHub, Markdown is automatically rendered.
>
> Tables, emojis, and formatting work reliably there.
```
Result:

> ⚠️ **Note:**
> On GitHub, Markdown is automatically rendered.
>
> Tables, emojis, and formatting work reliably there.


### 🔣 Commonly Used Symbols

| Symbol | Description          | Example                | Unicode Input    |
|--------|----------------------|------------------------|------------------|
| ✅     | Done                 | `✅ Done`              | `U+2705`         |
| ⚠️     | Warning              | `⚠️ Attention`         | `U+26A0 U+FE0F`  |
| 🚧     | Work in Progress     | `🚧 WIP`               | `U+1F6A7`        |
| ❌     | Error / Problem      | `❌ Build failed`      | `U+274C`         |
| 🔍     | Investigation        | `🔍 Debugging`         | `U+1F50D`        |
| 📦     | Package / Deployment | `📦 Released v1.2.0`   | `U+1F4E6`        |
| 📄     | Document             | `📄 Specification.md`  | `U+1F4C4`        |
| 🔐     | Security / Secrets   | `🔐 API key stored`    | `U+1F510`        |

#### Note on Usage:
- Unicode inputs are noted in the form U+XXXX, which describes the official Unicode code point.
- In HTML, you can also write emojis as &#xXXXX; (hexadecimal) or &#DDDD; (decimal).
    - Example: \&#x1F4E6; results in 📦
- Some symbols contain a so-called Variation Selector (e.g., U+FE0F) to ensure that the symbol is displayed as
- an emoji (and not as a black-and-white glyph).


>✏️ Tip: Many Markdown editors (e.g., IntelliJ, VS Code, Obsidian) show you a live preview. On GitHub
> or GitLab, Markdown is automatically rendered.


## 🛠️ Viewers & Editors
Markdown can be written and displayed with many tools:

### 🖥️ Windows & macOS
- IntelliJ (free as Community Edition, or Ultimate as paid version)

- Visual Studio Code (free, with live preview via extension)

- Typora (WYSIWYG editor with preview)

- Obsidian (ideal for knowledge collections, also usable locally)

- MarkText (Open Source Markdown editor)

### 🌐 Web-based Editors
- StackEdit – Online editor with cloud sync

- Dillinger – Simple web editor with export function

- HackMD – Collaborative Markdown editor (ideal for teams)

## Back to Content:
[Back to Starting Point](../README_EN.md)