# Course File Converter

A three-tab GUI tool for learning designers that handles two conversion workflows:

1. **Padlet → Word** — Takes a curriculum-map Markdown file exported from Padlet and a blank Word template, and produces a fully populated `.docx` course planner.
2. **Word → Brightspace HTML** — Takes a populated `.docx` course file and converts it to clean HTML ready to paste into Brightspace (D2L), with support for single-file or per-module output.

---

## Features at a glance

### Padlet → Word

| Padlet content | Word output |
|---|---|
| `## Module N:` headings | Fills `[module title]` placeholders in the template |
| Module learning objectives (green cards) | Fills `List Paragraph` objective slots |
| Task cards (white cards) | Fills `[task title]` and body placeholders |
| Introduction tasks | Fills the Introduction section |
| Module culminating tasks (yellow cards) | Fills `Module Culminating Task` slots |
| Course culminating task | Fills `Course Culminating Task` slot |
| Course learning objectives | Fills the course-level objective list |
| Resource hyperlinks | Inserted as formatted "Read …" / "Watch …" citations |
| Attached files | Included as hyperlinks below the task body |
| YouTube links | Formatted as "Watch …" citations with optional metadata |
| Course title (`# DEPT ### course planner`) | Reformatted as `DEPT###: Title` and written to the Title placeholder |

Cards with **Purple**, **Red**, **Green**, or **Blue** post colours, and cards explicitly labelled "Progress Update", "Final Report", "Additional Key Resource", "Guiding Resource", "[See Earlier Note]", or "Course Tasks" are automatically skipped.

### Word → Brightspace HTML

| Word feature | HTML output |
|---|---|
| Heading 1–6 | Configurable tag (h2–h6) via the GUI |
| Bullet lists | `<ul><li>` (nested levels supported) |
| Numbered lists | `<ol><li>` (nested levels supported) |
| Bold / Italic / Underline / Strikethrough | `<strong>`, `<em>`, `<u>`, `<del>` |
| Hyperlinks | `<a href="...">` |
| Blockquote (Quote / Block Text style) | `<blockquote><p>` |
| Accordion (single-cell table) | D2L accordion card HTML |
| Embedded images | Extracted to `images/` subfolder; `<img>` tags with relative paths |
| YouTube links (inline hyperlinks) | Converted to responsive `<iframe>` embeds |
| Module headings (`Module 1:`, `Module 2:`, …) | One HTML file per module |
| Tracked changes | Accepted silently before conversion |

---

## Setup (one-time)

You need **Python 3.8+** installed. No other manual setup is required.

When you run the converter for the first time, it will automatically detect any missing packages and show a graphical installer window — just click **Install** and the app will handle the rest before opening.

If you prefer to install manually beforehand:

```bash
pip install python-docx
pip install tkinterdnd2   # optional — adds drag-and-drop support
pip install requests beautifulsoup4   # optional — enables resource metadata fetching
```

---

## Running

```bash
python course_file_converter.py
```

On first launch, if any required packages are missing a **📦 Missing Python Packages** window will appear. It lists what needs to be installed, shows live pip output as packages download, and opens the main app automatically once installation is complete. If a package fails to install, a **Retry** button appears. You can also click **Skip** to proceed without installing (the app may not function correctly if required packages are absent).

The app opens to three tabs: **Word to Brightspace**, **Padlet to Word**, and **Settings**.

---

## Padlet → Word

### Overview

This tab converts a Padlet-exported Markdown curriculum map into a populated Word course planner document. You supply two inputs and one output:

- **Curriculum map (.md)** — the Markdown file exported from your Padlet board
- **Word template (.docx)** — the blank course planner template with placeholder headings and body paragraphs
- **Save output as (.docx)** — where to write the generated document

Click **Generate → .docx** to run the conversion. The Conversion Log below the button reports any warnings (extra task cards that didn't fit available slots, modules in the Markdown that exceeded the template's module count, etc.).

### How the Padlet Markdown is parsed

The converter exports Padlet boards as a Markdown file in which each card becomes a `### N. Card Title` section separated by `---` dividers. Module columns are denoted by `## Module N: Title` headings. The converter reads each card and classifies it by its heading text and post colour:

| Classification | Detected by |
|---|---|
| Course learning objectives | Heading contains "course learning objectives" |
| Course culminating task | Heading contains "course culminating task" |
| Module learning objectives | Heading contains "learning objectives" |
| Module culminating task | Heading contains "module culminating task", or post colour is Yellow |
| Introduction task | Heading contains "task" and "introduce yourself" |
| Regular task | Heading matches `task \d+` |
| Skipped | Post colour is Purple/Red/Green/Blue, or heading matches known noise patterns |

### How the Word template is filled

The template uses specific placeholder strings and Word styles that the converter targets:

| Template element | What gets written |
|---|---|
| `Title`-style paragraph | Reformatted course name, e.g. `CONT806: The Learning Environment` (requires **Detect and reformat course name** to be enabled) |
| `Heading 1` containing `[module title]` | Replaced with the module title from the Markdown |
| `Normal` paragraph containing "in this module you will have the opportunity to" | The `List Paragraph` items that follow are replaced with module learning objectives |
| `Normal` paragraph containing "in this course you will have the opportunity to" | The `List Paragraph` items that follow are replaced with course learning objectives |
| `Heading 2` containing `[task title]` | Replaced with the task title; body paragraph below it is replaced with task content |
| `Heading 2` containing "Module Culminating Task" | Title and body filled from the module's culminating task card |
| `Heading 2` containing "Course Culminating Task" | Title and body filled from the course culminating task card |
| `Heading 2` containing "Introduction" | Body filled from the introduce-yourself task card |

If a module has more tasks than the template provides slots for, the converter clones the last `[task title]` heading and inserts the overflow tasks before the Module Culminating Task heading. Empty slots with no corresponding task card are removed from the output document.

### Resource formatting

Links found in the body text of a card, and any files listed under `[Attachment N](url)`, are collected and prepended to the task body under a **Resources:** label.

- Regular web links are prefixed with bold **Read** and formatted as `Read [Title] by Author from Site (Year).`
- YouTube links are prefixed with bold **Watch** and formatted as `Watch [Title] (M:SS) from Channel (Year).`
- When two or more consecutive links share the same verb, they are grouped under a single **Read the following:** / **Watch the following:** header with each item indented below it.
- If metadata fetching is disabled (no YouTube API key saved), the original hyperlink display text is used as the title.

### Preview

The right-hand preview panel has three modes, selectable from the toolbar:

- **Rendered** — shows a Word-style layout of the output `.docx` after generation, or of the template as soon as you select it
- **MD Source** — shows the raw Markdown of the curriculum map as soon as you select it
- **Side by Side** — shows both panes simultaneously

The MD Source pane loads immediately when you pick a `.md` file, before any generation has occurred.

---

## Word → Brightspace

### Single File mode

Click the file area (or drag and drop a `.docx`) to pick your source document, adjust settings if needed, then click **Convert → .html**. The converted HTML is saved alongside the source file by default.

### Batch mode

Switch to **Batch** mode using the pill toggle at the top of the tab to convert an entire folder of `.docx` files at once.

**Workflow:**

1. The folder bar defaults to the current working directory when you open the app. Click **Browse…** to pick a different folder — all `.docx` files in it are listed automatically.
2. Each file has a checkbox. Use the **Select All / None** links to quickly check or uncheck everything, then uncheck any files you want to skip.
3. Click **Convert All →**. Files are converted one at a time so the UI stays responsive. A progress bar tracks the run.

**Status indicators** per file:

| Badge | Meaning |
|---|---|
| ⏳ | Currently converting |
| ✅ | Converted successfully (single file) |
| ✅ ×N | Converted successfully — N module files saved |
| ❌ | Conversion failed — hover the filename for the error |

Successful filenames turn green; failed ones turn red. Click any filename after conversion to load it into the preview panel.

### Brightspace Preview

The **Rendered** preview panel is styled to match a real Brightspace content page:

- A dark charcoal navbar at the top showing a **Course Home › Content › filename** breadcrumb, matching D2L's standard chrome
- A grey page shell with a white content card floating inside it, matching D2L's content page layout
- Lato typeface, D2L's default link blue (`#006fbf`), and heading styles consistent with D2L's defaults

When you use **Open in Browser**, the exported HTML uses the same stylesheet, so the browser view is consistent with the in-app preview.

> The preview approximates D2L's default appearance before institution-level theming is applied. If your institution has a heavily customised theme, you can load your own `.css` file in **Settings → Preview CSS** to override the defaults.

Accordion cards in the Rendered view are interactive — click a card header to expand or collapse its body, just as they behave in Brightspace.

### Image Extraction

Embedded images in your `.docx` are automatically extracted and saved to an `images/` subfolder alongside your HTML output. The converted HTML references them with relative paths:

```html
<img src="images/image1.png" alt="image1.png" style="max-width:100%;">
```

**What gets extracted:** any image inserted via Word's Insert → Pictures, including inline and floating images. Duplicate images (the same file embedded multiple times) are de-duplicated automatically.

**The status bar** reports how many images were extracted after each conversion, e.g. `✅ Saved: MyDoc.html (3 images → images/)`.

**In the Rendered preview**, images appear as a placeholder showing the filename: `🖼️ [image1.png]`. Use **Open in Browser** to see the actual images rendered.

**Image link annotations:** If images in your Word document have a URL attached via a Word comment, the converter extracts those URL-to-image mappings and saves them to a `_image_links.txt` file alongside the HTML output, grouped by module. This is useful when images in the source document serve as clickable links in Brightspace.

#### Uploading to Brightspace

After converting, your output folder will look like:

```
MyDoc_module_01.html
MyDoc_module_02.html
images/
  image1.png
  image2.png
```

Upload both the HTML files and the `images/` folder to the **same location** in Brightspace's Manage Files. As long as the folder structure is preserved, the relative `src="images/..."` paths will resolve correctly. If Brightspace flattens the folder on upload, the image paths will break — in that case you will need to re-point the `src` attributes manually or use a zip import that preserves folder structure.

### YouTube Embeds

Hyperlinks to YouTube videos found in your `.docx` are automatically converted to responsive `<iframe>` embeds in the HTML output. The embed uses `youtube-nocookie.com` for privacy. A CSS block for the responsive wrapper is injected once at the top of the output when at least one video is present.

In the Rendered preview, YouTube links appear as placeholder text. Use **Open in Browser** to see the actual iframes.

### Module Splitting

When the **Split into one file per Module** option is enabled (default: on), the converter detects headings that match the pattern **"Module N:"** — where N is any number from 1 to 99 — and splits the document into one HTML file per module.

**Example heading patterns detected:**
- `Module 1: Introduction`
- `Module 12: Advanced Topics`

**Output filenames** follow the pattern `MyDoc_module_01.html`, `MyDoc_module_02.html`, etc.

If no Module headings are found in the document, a single `.html` file is produced as normal.

#### Module Preview Navigation

When modules are detected, a **◀ Module N of M ▶** navigation bar appears in the preview toolbar. Click the arrows to step through each module's content without re-converting.

The preview always reflects the currently selected module. Switching between **HTML Source** and **Rendered** modes also respects the current module.

### Accordion Support

To create accordion output, place your content inside a **single-cell Word table**. The converter detects these automatically — no special styles or plugins required.

| Element | How to create it |
|---|---|
| **Card title** | Use a **Heading 4** paragraph inside the cell |
| **Card body** | Use normal body paragraphs after the Heading 4 |

Each Heading 4 paragraph inside the cell starts a new accordion card. The body paragraphs that follow it (up to the next Heading 4, or the end of the cell) become that card's content. Multiple Heading 4 / body pairs in the same cell produce multiple cards inside a single `<div class="accordion">` wrapper.

**Example structure (all inside one single-cell table):**

```
Heading 4 → "What is Brightspace?"
Normal paragraph → "Brightspace is a learning management system..."

Heading 4 → "How do I enroll?"
Normal paragraph → "Contact your institution's registrar..."
```

**Output:**

```html
<div class="accordion">
  <div class="card">
    <div class="card-header">
      <h2 class="card-title">What is Brightspace?</h2>
    </div>
    <div class="collapse">
      <div class="card-body">
        <p>Brightspace is a learning management system...</p>
      </div>
    </div>
  </div>
  ...
</div>
```

**Changing the trigger heading**

By default, **Heading 4** is used as the card title style. You can change this in **Settings → Accordion Card Title Style** if your document uses a different heading level.

> **Note:** A table is only treated as an accordion if it is a single-cell table containing at least one paragraph using the accordion heading style. All other tables are converted as standard `<table>` HTML.

### Copy HTML

Click **Copy HTML 📋** in the bottom-left of the preview panel, or press **Ctrl+Shift+C**, to copy the converted HTML directly to your clipboard.

- The button briefly changes to **✓ Copied!** as confirmation, then resets.
- When modules are detected, only the **currently displayed module** is copied — since each module is pasted into its own Brightspace content page separately.
- When no modules are present, the full document body is copied.
- The clipboard always receives a **bare HTML snippet** (no `<!DOCTYPE>` wrapper), regardless of the "Wrap in full HTML document" setting — Brightspace's HTML editor expects a fragment, not a full document.

### Save as ZIP

Click **Save as ZIP 📦** to package all HTML files and the `images/` folder into a single `.zip` archive for easy transfer or upload. The ZIP preserves the same file structure as the folder-based output.

### Open in Browser

Click **Open in Browser 🌐** to open the current conversion in your default browser, rendered with the full preview stylesheet. Images are resolved from their relative paths, so embedded images display correctly in the browser view.

### Search (Ctrl+F)

Press **Ctrl+F** while the app is focused to open a search bar at the top of the preview panel. It works in both **HTML Source** and **Rendered** modes.

- Type to search incrementally — all matches are highlighted in amber.
- The current match is highlighted in orange and scrolled into view.
- Press **Enter** / **▼** to jump to the next match; **Shift+Enter** / **▲** for the previous.
- Press **Escape** or click **✕** to close the search bar.

---

## Settings

The Settings tab is divided into two sections: one for the Word → Brightspace converter and one for Padlet → Word.

### Word → Brightspace settings

#### Heading Map

Remap each Word heading level to any HTML heading tag. For example, if your Brightspace page already has an `<h1>` title, you might want:

- Heading 1 → `h2`
- Heading 2 → `h3`
- Heading 3 → `h4`
- and so on.

Choose `(skip)` to drop that heading level from the output entirely.

#### List Transform

Force bullet lists to always output as `<ul>` or `<ol>`, and the same for numbered lists. Useful when your Word document uses numbered list style for items you want as bullets in Brightspace.

#### Blockquote Transform

Control how Word's `Quote`, `Block Text`, and `Intense Quote` styles are output. Options are `<blockquote>`, `<p>`, `<div class="callout">`, or `(skip)`. You can also enable **Add `<hr>` dividers inside blockquote** to wrap each blockquote in horizontal rules.

#### Accordion Card Title Style

Choose which Word heading level (Heading 1–6) triggers accordion card detection inside single-cell tables. Default is Heading 4.

#### Preview CSS (optional)

Load a `.css` file to style the preview panel and the full-document HTML output. Loading your Brightspace theme's stylesheet gives you an accurate preview of how the converted page will look to students. CSS is only embedded in the output when **Wrap in full HTML document** is enabled.

#### Paragraph Font Size

Adjust the font size for body paragraph text in the Rendered preview. Range: 10–36 px. Does not affect the exported HTML.

#### Output Options

- **Wrap in full HTML document** — adds `<!DOCTYPE html>`, `<html>`, `<head>`, and `<body>` tags. Useful for previewing in a browser. Leave unchecked for a snippet you paste directly into Brightspace's HTML editor.
- **Save output alongside source** — saves `MyDoc.html` (or `MyDoc_module_01.html` etc.) next to the source `.docx`. Uncheck to choose a custom save location each time you convert.
- **Split into one file per Module** — when Module headings are detected, saves each module as a separate file. Uncheck to always produce a single combined file.
- **Strip inline `style=""` attributes** — removes all `style="..."` attributes from the output HTML. Useful when Brightspace's editor applies its own styles and you don't want conflicts.

#### Profiles (Presets)

The **Settings** tab includes a **Profiles** dropdown that lets you save, load, rename, and delete named sets of settings.

- **First launch:** a default **Profile 1** is created automatically.
- **Saving:** click **Save…**, enter a name (or pick an existing one to overwrite), and confirm.
- **Loading:** select a profile from the dropdown — settings apply immediately.
- **Persistence:** the last-used profile is remembered across sessions and restored automatically when you reopen the app.
- **Import / Export:** use the **Export profiles…** and **Import profiles…** buttons to share settings across machines via a `.json` file.

Profiles store: heading map, list transforms, blockquote style, accordion heading, and CSS file path.

### Padlet → Word settings

#### Remove Padlet Instructions

When enabled, any text enclosed in square brackets — such as `[placeholder]`, `[see earlier note]`, or `[optional]` — is stripped from the Markdown source before parsing. Useful for keeping internal Padlet annotations out of the final Word document.

Card heading lines (the `### N. Card Title` lines) are never stripped, because bracket content in headings controls classification logic (e.g. `[See Earlier Note]`).

#### Attached Files

When enabled (default), attachment URLs listed under each Padlet card (`[Attachment N](url)`) are included as hyperlinks in the task body. Disable to omit all attached files from the output.

#### Detect and Reformat Course Name

When enabled, the converter reads the Padlet document title (e.g. `The Learning Environment CONT 806 course planner`) and writes a reformatted heading to the `Title`-style paragraph in the template.

Format: `CONT806: The Learning Environment` — the course code (e.g. `CONT806`) is extracted and placed at the front, followed by the descriptive title. Noise words like "course planner", "course outline", and "course map" are removed.

#### Validate Links

When enabled, the converter makes an HTTP request for each resource link during conversion and attaches a Word comment to any that are flagged. Comment text describes the issue:

- **Broken link** — the URL returned an error; search for a replacement before publishing
- **Redirected** — the URL permanently redirects; update to the final destination URL
- **Access restricted (403)** — verify manually in a browser before publishing
- **Could not verify** — network timeout or other inconclusive result; check manually
- **Tracking parameters** — URL contains `utm_*` or similar tracking parameters; consider removing

This adds time to conversion (one HTTP request per link) and requires an internet connection. PDF URLs that return 403 are treated as accessible, since many PDF servers block scripted requests.

#### Resource Metadata (YouTube API Key)

When a YouTube Data API v3 key is saved, the converter fetches metadata for every resource link during conversion and formats citations automatically:

- **YouTube links:** `Watch [Title] (M:SS) from Channel (Year).`
- **Web links:** `Read [Title] by Author from Site (Year).`

Metadata is retrieved from the YouTube oEmbed API (title and channel name, no key required), and from YouTube Data API v3 (duration and publish year, requires key) for videos. For web pages, title, author, site name, and year are extracted from Open Graph tags, JSON-LD structured data, and standard meta tags.

To save the key, paste it into the **YouTube API Key** field and click **Save Key**. The key is stored locally in your config file and is only transmitted to the YouTube API. Leave the field blank to disable metadata fetching.

---

## Conversion Log

A **Conversion Log** panel sits below the main controls on both the Word → Brightspace tab and the Padlet → Word tab.

**What it shows:**

For Word → Brightspace:
- A summary of every element type converted — headings, paragraphs, list items, blockquotes, tables, accordions
- Images extracted and their filenames
- Elements skipped because a style is set to `(skip)` in Settings
- Unrecognised Word styles that were treated as plain `<p>` (useful for spotting styles you may want to remap)
- File paths and sizes of every HTML file written to disk
- The path of the `images/` folder when images are saved

For Padlet → Word:
- Warnings about task cards that had nowhere to go (appeared before the first module heading, or the module they belong to exceeded the template's slot count)
- Any modules in the Markdown that the template didn't have room for

**Severity levels** are colour-coded:

| Symbol | Colour | Meaning |
|---|---|---|
| `·` | Muted | Info — everything converted as expected |
| `⚠` | Amber | Warning — something was skipped or fell back to a default |
| `✕` | Red | Error — a file couldn't be written or a conversion failed |

The header badge (`OK`, `2 warnings`, `1 error`) gives an at-a-glance summary. The panel collapses automatically after a clean run and expands automatically when there are warnings or errors.

---

## Drag and Drop

Drag a `.docx` file from your file manager and drop it anywhere on the application window (Word → Brightspace tab only). The drop zone highlights as soon as you bring a file over the app. Requires the `tkinterdnd2` package, which the automatic installer will offer to install on first launch.

---

## Building an Executable

A `build.py` script is included to compile the converter into a standalone `BrightspaceConverter.exe` using PyInstaller. All dependencies are bundled into the executable — end users do not need Python or any packages installed.

```bash
python build.py           # standard build
python build.py --fast    # skip cleaning, rebuild only changed files
python build.py --debug   # build with console window for troubleshooting
```

The finished executable is written to `dist/BrightspaceConverter.exe`. The dependency installer dialog does not appear in the built executable, since all packages are already bundled inside it.

**Prerequisites for building:**

```bash
pip install pyinstaller python-docx tkinterdnd2
```
