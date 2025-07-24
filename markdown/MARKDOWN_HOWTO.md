# ✍️ Markdown HOWTO

Ein kompakter Einstieg in **Markdown**, das bevorzugte Format für diese Wissenssammlung.

---

## 🧾 Was ist Markdown?

**Markdown** ist eine einfache Auszeichnungssprache, mit der sich strukturierter Text wie Überschriften, Listen, Code oder Tabellen schreiben lässt – lesbar im Klartext und gleichzeitig darstellbar als HTML.

---

## 🚀 Einsatz

Markdown eignet sich hervorragend für:

- Dokumentation in Git-Repositories (`README.md`, How-Tos, Guides)
- Wikis und interne Wissenssammlungen
- Blogposts, Change Logs oder API-Beschreibungen
- Formatierte Notizen, z. B. in IntelliJ, VS Code, GutHub, GitLab  und vielen mehr

---

## 🧪 Beispiele & Snippets

### Überschriften

```markdown
# H1
## H2
### H3
```

### Listen
```markdown
- Punkt 1
- Punkt 2
    - Unterpunkt

1. Erstens
2. Zweitens
```

### Code & Befehle
Inline: `kubectl get pods`

Codeblock:
<pre> ```bash 

kubectl get pods -n my-namespace 
``` </pre>

Ergebnis:
```bash

kubectl get pods -n my-namespace 
```

### Links & Bilder
```markdown
[Linktext](https://example.com)

![Alt-Text](images/logo.png)
```

### 🧰 Formatierung
| Stil            | Markdown-Beispiel | Ergebnis   |
| --------------- | ----------------- | ---------- |
| Fett            | `**Text**`        | **Text**   |
| Kursiv          | `*Text*`          | *Text*     |
| Fett + Kursiv   | `***Text***`      | ***Text*** |
| Durchgestrichen | `~~Text~~`        | ~~Text~~   |
| Zitat           | `> Zitat`         | > Zitat    |
| Trennlinie      | `---` oder `***`  | ---        |

### 📊 Tabellen

| Name       | Sprache   | Tool      |
|------------|-----------|-----------|
| Service A  | Kotlin    | Spring    |
| UI Frontend| Dart      | Flutter   |


### Markierung mit BlockQuotes für z.B. Tips etc.

Blockquote (Zitatblock), und so schreibst du ihn in Markdown:

```markdown
> ✏️ **Tipp:** Viele Markdown-Editoren (z. B. VS Code, Obsidian) zeigen dir direkt eine Vorschau an. Bei GitHub oder GitLab wird Markdown automatisch gerendert.
```
Ergebnis:
> ✏️ **Tipp:** Viele Markdown-Editoren (z. B. VS Code, Obsidian) zeigen dir direkt eine Vorschau an. Bei GitHub oder GitLab wird Markdown automatisch gerendert.

💡 Mehrzeilige Tipps oder Notizen kannst du einfach durch mehrere > Zeilen schreiben:
```markdown
> ⚠️ **Hinweis:**
> Bei GitHub wird Markdown automatisch gerendert.
>
> Tabellen, Emojis und Formatierungen funktionieren dort zuverlässig.
```
Ergebnis:

> ⚠️ **Hinweis:**
> Bei GitHub wird Markdown automatisch gerendert.
> 
> Tabellen, Emojis und Formatierungen funktionieren dort zuverlässig.


### 🔣 Häufig genutzte Symbole
| Symbol | Beschreibung         | Beispiel                 |
| ------ | -------------------- | ------------------------ |
| ✅      | Erledigt             | `✅ Done`                 |
| ⚠️     | Warnung              | `⚠️ Achtung`             |
| 🚧     | In Arbeit            | `🚧 WIP`                 |
| ❌      | Fehler / Problem     | `❌ Build failed`         |
| 🔍     | Untersuchung läuft   | `🔍 Debugging`           |
| 📦     | Paket / Deployment   | `📦 Released v1.2.0`     |
| 📄     | Dokument             | `📄 Spezifikation.md`    |
| 🔐     | Sicherheit / Secrets | `🔐 API-Key gespeichert` |

✏️ Tipp: Viele Markdown-Editoren (z. B. VS Code, Obsidian) zeigen dir direkt eine Vorschau an. Bei GitHub oder GitLab wird Markdown automatisch gerendert.


## 🛠️ Viewer & Editoren
Markdown lässt sich mit vielen Tools schreiben und anzeigen:

### 🖥️ Windows & macOS
- IntelliJ (konstenlos als Community Edition, oder als Ultimate kostenpflichtig)

- Visual Studio Code (kostenlos, mit Live Preview via Erweiterung)

- Typora (WYSIWYG-Editor mit Vorschau)

- Obsidian (ideal für Wissenssammlungen, auch lokal nutzbar)

-  MarkText (Open Source Markdown-Editor)

### 🌐 Webbasierte Editoren
- StackEdit – Online-Editor mit Cloud-Sync

- Dillinger – Einfacher Web-Editor mit Exportfunktion

- HackMD – Kollaborativer Markdown-Editor (ideal für Teams)



