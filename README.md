# Grist Search Widget

A custom Grist widget: pick a table, search it, click a result to see every
field, and click any Reference / Reference List field to drill straight into
the linked record — recursively, with a breadcrumb trail back.

## Install it in Grist

Grist loads custom widgets from a URL, so `index.html` needs to be hosted
somewhere reachable over HTTPS. Easiest options:

1. **GitHub Pages** — push this file to a repo, enable Pages, use the
   resulting URL.
2. **Netlify / Vercel drop** — drag the file onto their free static hosting.
3. Any other static host (S3 bucket, Cloudflare Pages, etc).

Then in Grist:
1. Add a new page/widget → **Custom** → **URL**.
2. Paste the hosted URL.
3. When prompted, grant it **Full document access** — this is required so
   it can read tables other than the one it's attached to (needed to follow
   reference links).

## How it works

- On load it reads Grist's own metadata tables (`_grist_Tables`,
  `_grist_Tables_column`) to discover every table, every column, and which
  columns are `Ref:OtherTable` / `RefList:OtherTable` links.
- The table dropdown lets you choose which table to search.
- Typing in the search box filters that table's rows across its text/choice
  columns.
- Clicking a result opens a detail panel listing every field. Reference
  fields render as clickable pills; clicking one fetches and opens the
  linked record in the same panel, pushing it onto a breadcrumb stack so you
  can navigate back up.

This directly supports your example: search **Projects** for "Muhlenberg
High School," open it, see all of its fields, then click its **School
Board** reference pill to jump into that School Board's own record (and so
on, however deep the links go).

## Customizing for your scenario

Everything you're likely to want to change lives in the `CONFIG` object near
the top of the `<script>` block:

```js
const CONFIG = {
  // Which column is the "title" shown in search results / headers, per table.
  titleFields: {
    Projects: 'Name',
    SchoolBoard: 'BoardName',
  },

  // Restrict which columns are searched, per table (optional).
  searchableFields: {
    Projects: ['Name', 'Location'],
  },

  // Hide specific columns from the detail view, per table (optional).
  hiddenFields: {
    Projects: ['InternalNotes'],
  },

  // Fully override how a table's detail panel renders, per table (optional).
  customRenderers: {
    Projects: (record, columns, helpers) => {
      // helpers.fieldRow(label, valueHtml), helpers.refPill(tableId, rowId),
      // helpers.escapeHtml(str) are available.
      return `<div>${helpers.escapeHtml(record.Name)}</div>`;
    },
  },

  // Table IDs to hide entirely from the table picker.
  excludeTables: [],
};
```

If you leave `titleFields`/`searchableFields` empty, the widget guesses
reasonably (first Text column as the title, all Text/Choice/numeric columns
as searchable) — so it works out of the box on any table, but naming the
title field explicitly gives cleaner results.

For deeper changes (styling, result list layout, how reference lists are
displayed, etc.), the CSS variables at the top of `<style>` and the
`render*` functions in the script are the places to edit — everything is in
that one file, no build step required.
