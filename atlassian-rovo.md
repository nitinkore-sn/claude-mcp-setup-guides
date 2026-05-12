---
title: Claude Desktop — Atlassian Rovo (Jira + Confluence) Setup
layout: default
---

# Claude Desktop — Atlassian Rovo (Jira + Confluence) Setup

GUI-first guide. Connect Claude Desktop to your Atlassian site so it can read/create Jira issues and Confluence pages directly from a chat.

Uses Atlassian's official remote MCP server at `https://mcp.atlassian.com/v1/mcp` via OAuth — no API tokens to manage, no JSON to edit.

---

## Setup

### 1. Connect on claude.ai

1. Open **https://claude.ai** in your browser and sign in with the **same account** you use in Claude Desktop.
2. Click your **profile avatar** (bottom-left) → **Settings**.
3. In the left sidebar, choose **Connectors**.
4. Find **Atlassian** in the list and click **Connect**.
5. A browser tab opens for Atlassian login. Sign in with your SambaNova SSO.
6. Approve the requested permissions (Jira read/write, Confluence read/write, etc.).
7. If you have multiple Atlassian sites, pick the one you want — for SambaNova: `sambanova.atlassian.net`.
8. The connector status flips to **Connected**.

### 2. Pick up in Claude Desktop

1. **Fully quit** Claude Desktop: press **⌘ Q**. Closing the window is not enough on macOS.
2. Relaunch Claude Desktop from Applications or Spotlight.
3. Open any chat. The Atlassian tools are now loaded — you'll see them listed when you open the **🔌 connectors / tools menu**.

### 3. Smoke test

In a chat, try:

> List my Jira projects.

Or:

> Show me Jira issues assigned to me that are still open.

If projects/issues come back populated, you're done.

---

## What you can do once connected

A non-exhaustive list of the most-used Atlassian Rovo tools and what to say to trigger them:

| What you want | Try saying |
|---|---|
| See your sites / cloud IDs | "List my accessible Atlassian resources." |
| List Jira projects you can write to | "List my visible Jira projects." |
| Find existing issues | "Search Jira for issues in CUSTEI with status In Progress assigned to me." (uses JQL under the hood) |
| Read an issue in detail | "Get Jira issue CUSTEI-1234." |
| Discover issue types for a project | "What issue types are available in CUSTEI?" |
| Discover required fields for a type | "What fields are required when creating a Customer Enhancement in CUSTEI?" |
| Create an issue (preview first!) | "Show me a preview of a Jira ticket you'd create in CUSTEI for …. Don't create yet." → "Looks good, create it." |
| Update an issue | "Update CUSTEI-1234 — set priority to High and add a comment with this link …" |
| Link two issues | "Link CUSTEI-1234 as 'relates to' CLOUD-9876." |
| Add a worklog | "Log 2 hours on CUSTEI-1234 with comment 'investigation'." |
| Transition status | "Move CUSTEI-1234 to In Review." |
| Search Confluence | "Search Confluence for pages about 'endpoint logging access controls'." |
| Read a Confluence page | "Get the Confluence page titled 'Endpoint OpenSearch Logging Policy'." |
| Create / update Confluence | "Create a Confluence page in the SUPPORT space titled '…' with this content …" |

For Jira creation, **always ask for a preview before committing** — Jira deletions require admin privileges, so the cost of a wrong ticket is real.

---

## Multi-site handling

If you belong to more than one Atlassian site, the first time you ask for projects/issues you may see a list of cloud IDs returned. Pick one and either:

- Mention the site name in subsequent prompts ("…in my `sambanova.atlassian.net` site"), or
- Ask Claude to remember it for the session ("From now on use the sambanova.atlassian.net cloud ID").

---

## Verifying everything works

1. In Claude Desktop, in any chat, open the **🔌 tools / connectors menu**.
2. You should see entries prefixed with **Atlassian** (Rovo).
3. Run the smoke test:

   > Tell me which Atlassian account I'm connected as, and list my visible Jira projects.

If both come back populated, you're set.

---

## Troubleshooting

| Symptom | What to do |
|---|---|
| Connector says "Connect" instead of "Connected" on claude.ai | Re-click Connect, complete OAuth in the popup. Make sure popups aren't blocked. |
| After install, Atlassian tools don't appear in Claude Desktop | You closed the window instead of fully quitting. Press **⌘ Q**, then relaunch. |
| Tools appear but every call returns 401 / "not authenticated" | OAuth token expired. Go to claude.ai → Settings → Connectors → Atlassian → **Reconnect**. |
| "No accessible resources" / empty project list | Your Atlassian account doesn't have access to any Jira/Confluence projects on the site you authorized. Confirm in the Atlassian web UI that you can see projects there. |
| Wrong site selected | Disconnect on claude.ai → Connectors → Atlassian, reconnect, and pick the correct site. |
| You changed your Atlassian password / SSO / left the org | Disconnect and reconnect on claude.ai. |

---

## What's where on disk (reference)

You don't need to touch these, but useful to know:

- **Atlassian OAuth tokens:** managed by claude.ai — not stored locally on your Mac
- **Logs:** `~/Library/Logs/Claude/` (look at the newest `mcp*.log`)

---

## Security quick notes

- The Atlassian connector has **only the scopes you approved** during OAuth — review the consent screen.
- It can read/write everything **your Atlassian account** can read/write. There's no separate, narrower permission boundary. If your account is a site admin, the connector inherits admin powers.
- Treat Jira *creation* and *deletion* as production write operations. Always preview-then-create.
- Revoking access: claude.ai → Settings → Connectors → Atlassian → **Disconnect**, *and* in Atlassian → Profile → **Connected apps** revoke the Claude OAuth grant.
s