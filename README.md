# Travel Agent Cruise Quote Assistant

A [Claude Code](https://claude.com/claude-code) project that works as a
research assistant for a travel agent. Give it a client's trip intake requests and
it searches the cruise lines' travel-agent portals in a real browser, logs
what it finds, compares the options (including promos and perks), and
drafts a client-ready quote as both a paste-ready email and a branded
printable document.

It's built around `CLAUDE.md` (the agent's working instructions) and
`portals.md` (hard-won navigation notes for each portal), so it gets
more reliable with every search instead of re-learning each site.

## Supported portals

| Cruise line | Portal |
|---|---|
| Carnival | GoCCL Navigator |
| MSC | MSC Book |
| Royal Caribbean, Celebrity, Silversea | CruisingPower |
| Norwegian | Norwegian Central → Quest |
| Disney | Disney Travel Agents |

You need your own travel-agent login for each portal you want to use.

## How it works

- **You log in, it browses.** The agent drives a Chrome window through the
  [Playwright MCP server](https://github.com/microsoft/playwright-mcp). You
  sign in to each portal yourself in that window. It never asks for, stores
  or types passwords or 2FA codes.
- **Read-only.** It never books, pays or holds a cabin without you
  confirming that specific step.
- **No bot-evasion.** If a portal shows a CAPTCHA or blocks automation, it
  stops and hands control back to you. 
- **You track the details.** After researching, it posts each sailing's
  exact day-by-day itinerary and the promo codes it found in the chat.

## Setup

1. Install [Claude Code](https://claude.com/claude-code) and Node.js
   (for `npx`), and have Google Chrome installed.
2. Clone this repo and open it in Claude Code. Approve the `playwright`
   MCP server from `.mcp.json` when prompted. It keeps a persistent browser
   profile in `~/.pandp-agent/browser-profile`, outside the repo, so your
   portal logins stay on your machine.
3. Copy `branding.example.md` to `branding.md` and fill in your agency's
   name, contact details and colors. Put your logo files in `assets/`.
4. Ask Claude to open a portal, log in when the browser window appears,
   and give it a client request (see `templates/intake.md` for what to
   capture).

Chrome shows an "unsupported command-line flag" banner at the top of the
agent's window. That's expected and harmless (see `CLAUDE.md`).

## What's in here

| Path | Purpose |
|---|---|
| `CLAUDE.md` | The agent's workflow and rules |
| `portals.md` | Per-portal navigation notes, quirks and pricing gotchas |
| `templates/` | Intake form, research notes, email quote, branded HTML quote |
| `branding.example.md` | Blank branding details — copy to `branding.md` |
| `.mcp.json`, `playwright-config.json` | Browser automation setup |
| `clients/` | One folder per client (git-ignored — client data never gets committed) |

## Privacy

`clients/`, `branding.md`, `assets/` and `.playwright-mcp/` (the browser
tool's page captures, which can include logged-in portal pages) are all
git-ignored. Check `git status` before committing anything new.
