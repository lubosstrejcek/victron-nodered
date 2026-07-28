# CLAUDE.md

Guidance for Claude Code working in `lubosstrejcek/victron-nodered` (**public** repo).

## What this repo is — and is not

It is a **Node-RED user directory (`/data`) kept under git**, not an application. There is
nothing to build, nothing to install, and no test suite.

| File | What it is |
|---|---|
| `flows.json` | 80 KB, **174 nodes across 8 tabs**. This is the entire program. |
| `settings.js` | 623-line Node-RED runtime config (stock file, a handful of edits) |
| `docker-compose.yml` | one `nodered/node-red:latest` service; repo root bind-mounted to `/data` |

`package.json` is the **stock Node-RED stub** — `name`/`description`/`version`/`private` and
nothing else: no `dependencies`, no `scripts`. `package-lock.json` and `node_modules/` are
gitignored, so `npm ci` cannot work here by design.

**Do not add `npm ci`, `npm test`, or a build step to CI.** There is no lockfile to install
from and no script to run.

## The container writes into the working tree

`docker-compose.yml` mounts `./:/data`. The repo root *is* the Node-RED user directory, so a
running container **rewrites `flows.json` in place** every time you hit Deploy in the editor —
and drops `flows_cred.json`, `.config.*.json`, and (if you install palette nodes) a
`package-lock.json` + `node_modules/` alongside it. That is what the `.gitignore` entries are
for; they are not cosmetic.

Consequences to keep in mind:

- `flowFilePretty: true` in `settings.js` is load-bearing — it is why the committed
  `flows.json` diffs are reviewable instead of one 80 KB line. Do not turn it off.
- Before committing, check whether a container is running. `git diff flows.json` may contain
  edits you made in the browser and forgot about, or a spurious reorder from a Deploy.
- Hand-editing `flows.json` while the container is up will be silently overwritten on Deploy.

## CI (`.github/workflows/ci.yml`)

Static validation only, on push to `main` and every PR:

1. `node --check settings.js` — a syntax error means the container never comes up.
2. A Node script that parses `package.json` and `flows.json` and asserts flows are a top-level
   array, every node has a non-empty string `id` and `type`, ids are unique, and every `wires`
   target resolves to an existing id.
3. `docker compose config --quiet` (offline parse + interpolation, no daemon needed).

Check 2 is the important one. Because the container rewrites `flows.json` in the working tree,
a truncated write or a bad hand-edit is the most likely breakage in this repo, and Node-RED
responds to a corrupt flow file by silently refusing to load it. Nothing else checks this.

## Gotchas that will cost you an hour

1. **Contrib nodes are declared nowhere.** `influxdb` / `influxdb out` come from
   `node-red-contrib-influxdb`, and `ui-base` / `ui-page` / `ui-group` / `ui-theme` / `ui-text`
   from `@flowfuse/node-red-dashboard`. `package.json` lists no dependencies, so a fresh
   `docker compose up -d` imports those as **"unknown node" placeholders**. Install them via
   the editor's palette manager; that writes them into `package.json` in the working tree, and
   that change *is* worth committing.
2. **Keepalive is mandatory.** An inject fires every 50 s into an `mqtt out` publishing
   `R/c0619ab6d055/keepalive`. The Victron GX broker stops publishing `N/…` topics roughly 60 s
   after the last keepalive. If data goes silent, suspect that inject before anything else.
3. **The portal ID is hardcoded in all 30 `mqtt in` topics** (`c0619ab6d055`). It is derived
   from the GX device, so pointing this at a *different* GX means editing 30 topics, not just
   the broker host.
4. **No `credentialSecret` is set** in `settings.js`. Node-RED generates a random one into
   `.config.runtime.json`, which is gitignored — so `flows_cred.json` is not portable between
   machines or clones. Expect to re-enter credentials after a fresh clone.
5. **No `adminAuth`** — the editor on `:1880` is unauthenticated. Fine on a training LAN, not
   fine anywhere reachable.

## README drift (verified 2026-07-28, not yet fixed)

- The "Flow Structure" table lists 6 tabs (`1. MQTT Input` … `6. Errors`). `flows.json`
  actually has 8: `0. System`, `1. MQTT Input`, `2. Processing`, `3. Digital Inputs`,
  `4. Shelly Monitor`, `5. InfluxDB`, `6. Dashboard`, `7. Errors`.
- "InfluxDB connection is pre-configured to `host.docker.internal:8086`" is wrong. The config
  node is `http://influxdb:8086` (database `energy`, InfluxDB 2.0), and `docker-compose.yml`
  defines no `influxdb` service — the `extra_hosts: influxdb:host-gateway` line is commented
  out. Out of the box, InfluxDB writes fail.
- "Only step needed: set your Ekrano GX IP" understates it; see gotcha 3.

## Protocol: MQTT, not Modbus

This repo talks **MQTT** (`N/{portal_id}/…` subscribe, `R/{portal_id}/keepalive` publish) plus
a Shelly HTTP API exposed at `/shelly/{on,off,toggle,status}` on port 1880. `grep -i modbus
flows.json` returns nothing, and that is correct — Modbus TCP :502 unit 100 is a *separate*
Loxone↔Victron control path that has no presence in this repo. Do not conflate them.

Since the repo is public, keep site-specific addresses and any credential out of commits
beyond the training-lab values already here.
