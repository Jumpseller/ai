---
name: jumpseller-cli
description: "Use when using the Jumpseller CLI to manage store credentials or do local theme development (export, import, watch, apply, list, rename, delete themes)."
---
# Jumpseller CLI

Use this skill when using the **Jumpseller CLI** (`jumpseller`) — the official command-line tool for managing store credentials and doing local theme development against a Jumpseller store.

Source project: [`Jumpseller/jumpseller-cli`](https://github.com/Jumpseller/jumpseller-cli). It is in early development (v0.1.0) — when in doubt, confirm command names with `--help` and treat the repo README as the source of truth.

## Installation

```bash
npm i -g @jumpseller/cli      # install globally (requires Node >= 18)

jumpseller --version          # -v also works
jumpseller --help
```

Development setup (working on the CLI itself):

```bash
git clone ssh://github.com/Jumpseller/jumpseller-cli
cd jumpseller-cli
yarn install
npm link                      # makes `jumpseller` available globally
```

## Two command families

The CLI currently exposes exactly two groups of commands:

| Command group | Purpose |
|---|---|
| `jumpseller access` | Manage API credentials and which store commands act on |
| `jumpseller theme`  | Local theme development — list, apply, rename, delete, export, import, watch |

Run any command or subcommand with `--help` for usage.

## Same credentials across the API, MCP server, and CLI

There is **one set of credentials per store + account**: a **Login key** and an **Auth Token**. The exact same pair is used by all three Jumpseller interfaces — you do not generate separate keys for each:

| Interface | How the credentials are supplied |
|---|---|
| **REST API** (`jumpseller-api`) | HTTP Basic auth: `curl -u LOGIN_KEY:AUTH_TOKEN …` |
| **MCP server** (`jumpseller-mcp`) | Headers `X-LOGIN-KEY` / `X-AUTH-TOKEN` (from env vars `JUMPSELLER_LOGIN_KEY` / `JUMPSELLER_AUTH_TOKEN`) |
| **CLI** (this skill) | Stored in `~/.config/jumpseller/credentials` as `domain login:authtoken`; sent to the API as Basic auth |

Retrieve the pair once from **Admin Panel → Account Settings → API Tokens** (also reachable at `https://<store>.jumpseller.com/admin/accounts`). The same values work for `curl`, the MCP `.mcp.json`, and `jumpseller access add` interchangeably.

## How the CLI picks which store to act on

Every command resolves a **current store** using this precedence (first match wins):

1. **`-s, --store <store>`** flag on the command line — overrides everything for that one command.
2. **Local default** — the nearest `.jumpseller-store` file, found by walking **up** from the current directory toward your home folder.
3. **Global default** — `~/.config/jumpseller/store`.

If none resolve, commands that need a store fail with *"No store selected…"* and tell you to run `jumpseller access`.

### Store references are flexible

You can pass a store as a short code or a full domain — they normalize to the same canonical domain:

| You type | Resolves to |
|---|---|
| `alejandrotest` | `alejandrotest.jumpseller.com` |
| `alejandrotest.jumpseller.com` | `alejandrotest.jumpseller.com` |
| `https://alejandrotest.jumpseller.com/` | `alejandrotest.jumpseller.com` (scheme/trailing slash stripped) |
| `test` | `test.localhost` (local development) |

## Configuration files

The CLI keeps state in plain files — useful to know for debugging and for CI:

| File | Contents |
|---|---|
| `~/.config/jumpseller/credentials` | One line per store: `domain login:authtoken`. `#` starts a comment. Login key and auth token are each 32–50 hex chars. |
| `~/.config/jumpseller/store` | The global default store domain. |
| `.jumpseller-store` (per folder) | The local default store domain for that theme folder. **Bind a theme to a store.** Safe to commit (it's only a domain) — but never put credentials here. |

Writes are atomic (written to a `.tmp` file then renamed), so a crash won't corrupt these files.

## `jumpseller access` — credentials & default store

| Command | What it does |
|---|---|
| `jumpseller access` | If no credentials exist yet, prompts to set up a first store and makes it the global default. Otherwise lists stored credentials. |
| `jumpseller access list` | Table of all stored credentials, the verified account for each (calls `/v1/whoami`), and which is the local/global default. ✔ = valid, ✖ = problem. |
| `jumpseller access current` | Print the currently-resolved store. |
| `jumpseller access add [store]` | Add or update credentials for a store. Prompts for **Login key** and **Auth Token**, then verifies them against the API. |
| `jumpseller access remove <store>` | Remove stored credentials for a store. |
| `jumpseller access default <store>` | Set the **global** default store (`~/.config/jumpseller/store`). |
| `jumpseller access local <store>` | Set a **local** default in the current directory (writes `.jumpseller-store`). |

> Find a store's credentials in the Admin Panel under **Account → Preferences → {your account}**, or at `https://<store>.jumpseller.com/admin/accounts`. Each store + account pair has its own Login key / Auth Token.

## `jumpseller theme` — local theme development

Themes are referenced by their **integer id**, visible in the editor URL: `/admin/themes/editor/654321` → id `654321`.

| Command | What it does |
|---|---|
| `jumpseller theme list` | Table of all themes: id, name, status (`active` if in use), parent, version, last updated, installed date, author, language. |
| `jumpseller theme apply <theme-id>` | Set a theme as the active/live theme. |
| `jumpseller theme rename <theme-id> [name...]` | Rename a theme (max 65 characters). |
| `jumpseller theme delete <theme-id...>` | Delete one or more themes (space-separated ids). |
| `jumpseller theme export <theme-id> [folder]` | Download a theme. If the target ends in `.zip`, saves a zip; otherwise extracts into a folder. Default name: `theme-<id>.zip`. |
| `jumpseller theme import <folder>` | Zip a local theme folder (including hidden files) and import it into the store as a new theme. |
| `jumpseller theme watch <theme-id> [folder]` | Live-sync: mirror local file edits to an installed theme. Folder defaults to `.`. See below. |

### Theme folder structure

A theme folder (e.g. exported from a store) looks like this:

```
theme/
├── .jumpseller-store   # binds this folder to a store (domain only)
├── assets/             # CSS, JS, images, fonts (assets/library/ is NOT synced by watch)
├── components/         # configurable sections — each is a .liquid + .json pair
├── config/             # theme settings
├── locales/            # translation strings
├── partials/           # reusable fragments
└── templates/          # page-level templates
```

### `theme watch` — live sync (read this before using)

`watch` listens for file-write events and pushes each change to the installed theme via the schema API (add/change → write, delete → unlink). Important behaviors verified from the source:

- **Only certain paths are synced.** By default: `partials/`, `components/`, `templates/`, `assets/`, `config/`. Excluded: `assets/library/`. Everything else is skipped.
- **`components/*.json` writes are blocked by default** because they can be destructive. Use `--unsafe` to allow them, or `--allow`/`--block` for fine-grained control.
- **It is git-aware and will exit on git activity.** If it detects a branch switch, a new/changed stash, or a `.git/index.lock`, it stops to avoid an unintended bulk overwrite of the live theme. So **don't `git switch`/`stash`/`pull` while watching** — stop the watcher first.

| Option | Effect |
|---|---|
| `--allow <pattern>` | Allowlist glob (quote it). Force-include paths. Repeatable. |
| `--block <pattern>` | Blocklist glob (quote it). Force-exclude paths. Repeatable. |
| `--unsafe` | Shorthand to allow all known unsafe patterns (`components/*.json`). |

Stop the watcher with `Ctrl-C` (SIGINT) — it flushes pending writes before exiting.

## Common workflows

### Set up the CLI on a new machine

```bash
npm i -g @jumpseller/cli
jumpseller access                 # prompts for store + Login key + Auth Token, sets global default
jumpseller access current         # confirm
```

### Pull a live theme down, edit, and sync changes back

```bash
jumpseller theme list                       # find the theme id
jumpseller theme export 654321 ./mytheme    # download + extract into ./mytheme
cd mytheme
jumpseller access local alejandrotest       # bind this folder to the store (writes .jumpseller-store)
jumpseller theme watch 654321               # live-sync edits as you save files
```

### Work on a duplicate so you don't touch the live theme

```bash
jumpseller theme export 654321 ./draft      # get the current theme
cd draft
jumpseller theme import .                    # creates a NEW theme in the store
jumpseller theme list                        # note the new theme's id
jumpseller theme watch <new-id>              # iterate safely, then `apply` when ready
```

### Switch which store you act on, for one command

```bash
jumpseller -s klikmuebles theme list         # ignore defaults, target klikmuebles this once
```

## Relationship to the rest of the toolkit

- Themes you edit here are written in **Liquid** — see the `jumpseller-liquid` skill for templating, objects, and the `components/` `.liquid` + `.json` pattern.
- The CLI talks to the same **REST API** documented in the `jumpseller-api` skill (Basic auth, `api.jumpseller.com`), specifically the `/v1/themes/*` and `/v1/whoami` endpoints.
