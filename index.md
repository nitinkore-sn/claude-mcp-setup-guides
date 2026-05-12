---
title: Claude Desktop MCP Setup Guides
layout: default
---

# Claude Desktop MCP Setup Guides

Step-by-step guides to connect Claude Desktop to Salesforce and Atlassian Jira using MCP (Model Context Protocol) servers. Once set up, you can read Salesforce cases and create Jira tickets directly from a Claude Desktop chat — no copy-pasting, no browser switching.

---

## Guides

### [Salesforce DX MCP Setup](salesforce-dx)
Connect Claude Desktop to any Salesforce org. Run SOQL queries, read Cases, Accounts, and any Salesforce object directly from chat.

- One-time terminal setup
- Read-only by default — safe for production orgs
- Works with any company's Salesforce instance

---

### [Atlassian Rovo (Jira + Confluence) Setup](atlassian-rovo)
Connect Claude Desktop to Jira and Confluence. Create issues, search projects, update tickets, and read Confluence pages — all from chat.

- Pure GUI setup via claude.ai connectors
- No config files, no terminal
- OAuth-based — no API tokens to manage

---

## Workflow: Salesforce Case → Jira Ticket

With both MCP servers connected, the full workflow is three prompts:

1. **Pull the case** — `Fetch all fields from Case where Id = '500xx000000001' from my-org`
2. **Pick the project** — `List my Jira projects and recommend one for this case`
3. **Preview, then create** — `Show me a preview of the Jira ticket. Don't create yet.` → `Looks good, create it.`
