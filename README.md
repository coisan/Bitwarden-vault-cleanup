# Vault tidy

A lightweight, browser-based cleanup tool for unencrypted Bitwarden JSON exports. Review proposed changes, compare duplicate logins side by side, and download a new export containing your approved updates.

The app is a single HTML file with embedded JavaScript, CSS, and domain-suffix data. No installation, build step, account, or backend is required.

## Getting started

1. Download `vault-cleanup.html` from this repository and open it in a modern browser.
2. Choose or drag in an **unencrypted Bitwarden JSON export**.
3. Review the cleanup categories and approve the changes you want.
4. Select **Download updated JSON** to save `<original-name>-cleaned.json`.

All selections start unchecked. The original file is never overwritten. Use **Clear vault from this page** to discard the loaded data and selections.

## Cleanup categories

| Category | Proposed changes | Approval |
| --- | --- | --- |
| **Name cleanup** | Removes every occurrence of `www.` from names, regardless of capitalization. | Per name, or Select all / Deselect all. |
| **URI cleanup** | Uses `https://`, removes a leading `www.`, and removes paths, query strings, and fragments. Keeps other subdomains and explicit non-default ports. | Per URI, or Select all / Deselect all. |
| **Subdomain cleanup** | For login items containing a subdomain URI, renames the item to the root domain and simplifies matching URIs to `https://root-domain`. | One checkbox per item approves its displayed NAME and URI changes together. Select all / Deselect all is available. |
| **Duplicate logins** | Groups logins by root domain and exact username, including related subdomains. Shows NAME, URI, USERNAME, and PASSWORD in side-by-side columns. | Select individual items to remove. No bulk removal control. |
| **Duplicate passwords** | Lists login items sharing exactly the same nonempty password, across domains and usernames. | Informative only; no update selections. |

### URI cleanup example

```text
Before: http://www.example.com/account/login?redirect=home#form
After:  https://example.com
```

Ordinary URI cleanup preserves subdomains: `https://users.example.com/login` becomes `https://users.example.com`. Removing `users.` requires Subdomain cleanup or a qualifying duplicate removal.

Non-web schemes such as `androidapp://` and `mailto:` are skipped. URIs containing embedded username/password credentials are also skipped. URI matching settings and other URI-object properties are retained.

### Subdomain cleanup

For an item with `https://fantasy.premierleague.com/login`, approving the grouped suggestion produces:

```text
NAME: premierleague.com
URI:  https://premierleague.com
```

The name is replaced with the root domain, including when the original name is a custom label. When an item has several applicable root domains, the name joins them with ` / `. Matching URIs retain explicit non-default ports; URIs belonging to other roots remain unchanged by this action.

Root-domain detection uses an embedded [Public Suffix List](https://publicsuffix.org/list/), including private suffixes. It handles suffixes such as `co.uk` and keeps separately owned hosted domains such as `one.github.io` and `two.github.io` apart. IP addresses and local hostnames are not reduced to domain suffixes.

### Duplicate logins

These URIs are potential duplicates when their usernames match exactly:

```text
fantasy.premierleague.com
users.premierleague.com
```

Different usernames stay in separate groups. Username matching is case-sensitive; empty-string usernames can match each other, while missing or non-string usernames are excluded.

Selecting a removal deletes the **entire selected item** from the new export. At least one item must remain in every duplicate group, including overlapping groups. Data from removed items is not merged into kept items.

For a group containing subdomains, selecting a removal also updates the kept entries' names and matching URIs to the root domain. The comparison previews these changes. Unchecking the removal reverses changes triggered by that selection; separately approved Subdomain cleanup changes still apply.

Shared roots indicate potential duplicates, not necessarily interchangeable logins. Review the displayed fields before removing an entry.

## How approvals interact

Changes are applied in this order:

1. Approved Name and URI cleanup changes.
2. Approved grouped Subdomain cleanup changes.
3. Root-domain changes triggered by duplicate removals.
4. Removal of selected duplicate items.

Later changes take precedence when they target the same field. Unchanged properties, passwords, passkeys, custom fields, folders, and other export data are preserved for retained items. The download preserves the JSON data structure, but reformats its whitespace.

## Password and passkey totals

The summary shows separate totals from the **original uploaded export**:

- **Passwords:** number of login items containing a nonempty password string. Repeated passwords count separately.
- **Passkeys:** number of credential objects in login items' `fido2Credentials` arrays. A login can contain multiple passkeys.

These totals do not change as removals are selected. Duplicate-password findings also reflect the original export.

## Privacy and handling exports

- File contents are processed in memory within the browser tab.
- The app makes no network requests and uses no analytics, external scripts, cookies, or browser storage for vault data.
- Domain-suffix data is bundled with the HTML, so the app works offline.
- A restrictive Content Security Policy blocks network connections.
- Passwords are displayed in plain text in comparison reports. The generated JSON also contains unencrypted secrets.

Keep vault exports and cleaned downloads out of Git commits, issues, screenshots, and public hosting directories. Publish only the application source and documentation.

## Scope and limitations

- Accepts unencrypted Bitwarden-style JSON with an `items` array; encrypted exports and CSV files are not supported.
- Generates an updated file; it does not connect to or modify a live Bitwarden vault.
- Importing a cleaned export is not a synchronization or deletion operation against existing vault entries. Keep a backup and review your import workflow separately.
- Root-domain cleanup can change the scope of URI matching. URI matching modes themselves are preserved.
- The embedded suffix list is a snapshot and does not update automatically.
- Large exports are processed synchronously and may temporarily slow the page.

## Development

The application lives entirely in `vault-cleanup.html`. Edit it directly, reopen or refresh the browser, and use synthetic exports to check behavior. There are no runtime package dependencies or build commands.

Use a modern browser supporting `URL`, `File.text()`, `structuredClone`, and Blob downloads. If publishing a static copy, the HTML can be served as-is or renamed to `index.html`.

The embedded Public Suffix List is distributed under the [Mozilla Public License 2.0](https://mozilla.org/MPL/2.0/). Its source attribution is included in the application.
