# WordPress operations

User-provided operating instructions, recorded September 8, 2026.

## Authentication and API

Credentials live in `~/.wp-parallellabs-credentials` as `WP_URL`, `WP_USER`, and `WP_APP_PASSWORD`. Authenticate with HTTP Basic auth using the user and application password. Never expose their values in output, source files, or commits. The WordPress Application Password is revocable in wp-admin under **Users → Profile**.

Use the standard REST API for page CRUD: `/wp-json/wp/v2/pages` and `/wp-json/wp/v2/pages/{id}`. Send a browser User-Agent header; otherwise Bluehost Mod_Security returns a non-JSON HTTP 406 response. Check HTTP status and response content before assuming JSON.

Upload media through `/wp-json/wp/v2/media` using **multipart form uploads**, never raw binary request bodies. Bluehost Mod_Security rejects raw binary uploads.

## Operations requiring wp-admin

The custom theme-file write endpoint was never registered on the live site. WPCode snippets cannot be written through REST or XML-RPC. These operations require the wp-admin editors.

The user referenced a CodeMirror save workaround in an existing note, but did not include its steps here. Locate and read that note before using the workaround; do not assume that changing the visible editor text saves it.

## Caching and verification

Cloudflare and Bluehost cache anonymous pages for about four hours. Verify fresh renders with a logged-in cookie. After CSS or JavaScript edits to the theme, bump the theme version in `style.css`.

## Product showcase preferences

Capture the running app in **light mode with default accent colors**. Use coherent, customer-relevant fictional companies and sample data; label illustrative metrics as demo data. Never present synthetic results as customer testimonials. Prefer the REST API for creating/updating showcase pages, reusable blocks, and media. Use the browser for actual app screenshots and visual verification.
