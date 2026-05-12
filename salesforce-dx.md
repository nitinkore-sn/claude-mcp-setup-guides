---
title: Claude Desktop — Salesforce DX MCP Setup
layout: default
---

# Claude Desktop — Salesforce DX MCP Setup

Connect Claude Desktop to any Salesforce org so it can run SOQL (Salesforce Object Query Language) queries and read object data directly from a chat.

Uses Salesforce's official MCP server (`@salesforce/mcp`) via the Salesforce CLI. One-time terminal setup, then fully hands-off.

---

## Before you start — find your values

You'll need two things specific to your org. Look them up once and use them throughout this guide.

| What you need | How to find it | Example |
|---|---|---|
| **Org alias** | Make one up — a short nickname for this org. You'll use it consistently in step 2 and step 3. | `my-company-prod` |
| **My Domain URL** | Open your Salesforce org in a browser. Take the subdomain from the URL bar and append `.my.salesforce.com`. | Browser shows `https://acme.lightning.force.com` → your URL is `https://acme.my.salesforce.com` |

⚠️ **The My Domain URL is not the same as what you see in the browser.** Salesforce displays a Lightning URL (`*.lightning.force.com`) but the CLI needs the My Domain URL (`*.my.salesforce.com`). Using the Lightning URL causes an immediate error.

| What the browser shows | What to use |
|---|---|
| `https://acme.lightning.force.com/lightning/r/Case/...` | `https://acme.my.salesforce.com` |
| `https://mycompany.lightning.force.com/...` | `https://mycompany.my.salesforce.com` |

---

## Setup

### 1. Install the Salesforce CLI

1. Open **Terminal** (⌘ Space → type *Terminal* → Enter).
2. Paste and press Enter:

   ```
   npm install -g @salesforce/cli
   ```

   Wait ~30 seconds. If you get *"command not found: npm"*, install Node.js first from **https://nodejs.org** (LTS version), then retry.

3. Verify:

   ```
   sf --version
   ```

   Expected output: something like `@salesforce/cli/2.133.4 darwin-arm64 node-v24.x.x` (version numbers will vary).

### 2. Authorize your Salesforce org (one-time)

1. In Terminal, run this command — replacing the two values with your own from the table above:

   ```
   sf org login web --alias <your-org-alias> --instance-url https://<your-subdomain>.my.salesforce.com
   ```

   Example:
   ```
   sf org login web --alias acme-prod --instance-url https://acme.my.salesforce.com
   ```

   What each piece means:

   | Piece | What to put here |
   |---|---|
   | `sf org login web` | Fixed — don't change. |
   | `--alias` | Your chosen org alias (e.g. `acme-prod`). You'll use this exact string again in step 3. |
   | `--instance-url` | Your My Domain URL from the table above (e.g. `https://acme.my.salesforce.com`). |

2. A browser tab opens. Log in with your company SSO or Salesforce credentials.
3. Return to Terminal — you'll see:

   ```
   Successfully authorized you@yourcompany.com with org ID 00Dxxxxxxxxxxxx
   ```

4. Close Terminal. **This is the only time you need it.** Auth tokens are stored in `~/.sfdx/` and shared automatically with Claude Desktop.

### 3. Find your Node paths

Claude Desktop doesn't inherit your shell PATH, so you must provide absolute paths to Node. Run this in Terminal:

```
which node && which npx
```

Copy both lines it prints. You'll paste them into the config file next.

Common examples:

| Setup | node path | npx path |
|---|---|---|
| nvm | `/Users/you/.nvm/versions/node/v24.x.x/bin/node` | `/Users/you/.nvm/versions/node/v24.x.x/bin/npx` |
| Homebrew (Intel Mac) | `/usr/local/bin/node` | `/usr/local/bin/npx` |
| Homebrew (Apple Silicon) | `/opt/homebrew/bin/node` | `/opt/homebrew/bin/npx` |

### 4. Add the MCP server to Claude Desktop config

1. Open **Finder**.
2. Press **⌘ ⇧ G** (Go to Folder).
3. Paste this path and press Enter:

   ```
   ~/Library/Application Support/Claude
   ```

4. Find **`claude_desktop_config.json`**. Right-click → **Open With** → **TextEdit**.
   - If the file doesn't exist, create it with that exact name in this folder.

5. Paste this block — replacing the three placeholders with your actual values:

   ```json
   {
     "mcpServers": {
       "Salesforce DX": {
         "command": "<absolute-path-to-node>",
         "args": [
           "<absolute-path-to-npx>",
           "-y",
           "@salesforce/mcp",
           "--orgs",
           "<your-org-alias>",
           "--toolsets",
           "data,orgs"
         ],
         "env": {
           "PATH": "<folder-containing-node>:/usr/bin:/bin:/usr/sbin:/sbin"
         }
       }
     }
   }
   ```

   What each placeholder is:

   | Placeholder | Replace with | How to get it |
   |---|---|---|
   | `<absolute-path-to-node>` | e.g. `/Users/you/.nvm/versions/node/v24.x.x/bin/node` | First line of `which node && which npx` from step 3 |
   | `<absolute-path-to-npx>` | e.g. `/Users/you/.nvm/versions/node/v24.x.x/bin/npx` | Second line of `which node && which npx` from step 3 |
   | `<your-org-alias>` | e.g. `acme-prod` | The alias you chose in step 2 |
   | `<folder-containing-node>` | e.g. `/Users/you/.nvm/versions/node/v24.x.x/bin` | Your node path minus the trailing `/node` |

   Filled-in example (nvm setup):
   ```json
   {
     "mcpServers": {
       "Salesforce DX": {
         "command": "/Users/jane/.nvm/versions/node/v24.12.0/bin/node",
         "args": [
           "/Users/jane/.nvm/versions/node/v24.12.0/bin/npx",
           "-y",
           "@salesforce/mcp",
           "--orgs",
           "acme-prod",
           "--toolsets",
           "data,orgs"
         ],
         "env": {
           "PATH": "/Users/jane/.nvm/versions/node/v24.12.0/bin:/usr/bin:/bin:/usr/sbin:/sbin"
         }
       }
     }
   }
   ```

   If you already have other entries in `"mcpServers"`, add `"Salesforce DX"` alongside them — don't replace existing entries.

6. **Save the file** (⌘ S).

### 5. Restart Claude Desktop

1. Press **⌘ Q** to fully quit. Closing the window is not enough.
2. Relaunch from Applications or Spotlight.

### 6. Smoke test

In any chat — replace with your own alias and a real object ID from your org:

> Run a SOQL query against my `acme-prod` Salesforce org to fetch all fields from the Case where Id = '500xx0000000001'.

You should see all case fields returned as a structured response.

---

## What you can do once connected

| What you want | Try saying |
|---|---|
| Read a record by ID | "Fetch all fields from Case where Id = '500xx000000001' from `acme-prod`." |
| Read specific fields only | "Get Subject, Description, Priority, Status, ContactEmail from Case 500xx000000001." |
| Search records by condition | "Find all open Cases created in the last 7 days in `acme-prod`." |
| Query any standard object | "Run SOQL: SELECT Id, Name, Industry FROM Account LIMIT 20" |
| Query a custom object | "Run SOQL: SELECT Id, Name, Status__c FROM My_Custom_Object__c WHERE Status__c = 'Active'" |
| List authorized orgs | "List all my authorized Salesforce orgs." |
| Check connected username | "Which Salesforce username am I connected as?" |

**SOQL tips:**
- `SELECT *` doesn't exist in SOQL. Use `FIELDS(ALL)` with `LIMIT ≤ 200` to get every field, or list fields explicitly for faster results.
- Custom fields end in `__c` (e.g. `My_Field__c`).
- Custom objects also end in `__c` (e.g. `My_Object__c`).

---

## Verifying everything works

1. In Claude Desktop, open the **🔌 tools / connectors menu** in any chat.
2. You should see **Salesforce DX** tools listed.
3. Run the smoke test prompt from step 6.

If records come back, you're set.

---

## Troubleshooting

| Symptom | What to do |
|---|---|
| Salesforce tools don't appear after relaunch | Closed the window instead of fully quitting. Press **⌘ Q**, then relaunch. |
| Tools still missing after ⌘ Q | Open the config file — check for invalid JSON (missing comma between entries, unbalanced braces). Validate: open Terminal, run `python3 -m json.tool ~/Library/Application\ Support/Claude/claude_desktop_config.json` |
| `"Invalid instance URL"` during `sf org login` | You used the `.lightning.force.com` URL. Use `.my.salesforce.com` instead — see the table at the top of this guide. |
| `"command not found: sf"` in Terminal | Salesforce CLI not installed. Re-run `npm install -g @salesforce/cli`. |
| Query returns 0 results / auth error | Token expired. Re-run `sf org login web --alias <your-alias> --instance-url https://<your-subdomain>.my.salesforce.com` in Terminal. |
| `node: command not found` in Desktop logs | The node/npx paths in the config don't match your machine. Run `which node` and update `command`, `args[0]`, and `env.PATH`. |
| Moving to a new machine | Run `which node` and `which npx` — update all three path fields. Re-run `sf org login web` to re-authorize the org. |

Logs: `~/Library/Logs/Claude/` — open the newest `mcp*.log` for raw startup errors.

---

## What's where on disk (reference)

- **Claude Desktop config:** `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Salesforce auth tokens:** `~/.sfdx/` — shared with all tools that use the SF CLI (IDE plugins, terminal, Claude Desktop)
- **Logs:** `~/Library/Logs/Claude/`

---

## Security quick notes

- `--orgs <your-alias>` whitelists only that one alias. Claude Desktop cannot accidentally query other Salesforce orgs you've authorized on your machine.
- `~/.sfdx/` holds your Salesforce refresh tokens — protect it like SSH keys.
- Don't share your `claude_desktop_config.json` with teammates — the absolute `/Users/<you>/` paths in it leak your username. Each person should follow this guide and generate their own paths using `which node` / `which npx`.
- To revoke access: run `sf org logout --target-org <your-alias>` in Terminal.
