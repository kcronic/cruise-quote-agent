# P&P Travel Agent Assistant

This project is a working assistant for a travel agent. Its job: given a client's
trip request, research options across the agent's portals (via a real, logged-in
browser session) and produce a comparison, then draft a client-ready quote.

## How this works

- The user (the travel agent) logs into each portal manually in the Playwright-
  controlled browser window. Never ask for, store, or type in passwords/2FA codes
  on the user's behalf.
- Use the `playwright` MCP browser tools (`browser_navigate`, `browser_snapshot`,
  `browser_click`, `browser_type`, etc.) to search portals per the client
  criteria and extract listings/prices.
- **Read-only by default.** Never submit a booking, payment, or any
  irreversible action in a portal without the user explicitly reviewing and
  confirming that specific step first.
- If a portal throws a CAPTCHA, MFA prompt, or looks like it's blocking
  automated traffic, stop and hand control back to the user rather than trying
  to work around it.

## Workflow

1. Intake a client request (see `templates/intake.md` for the fields to capture).
2. Check `portals.md` for which portal(s) fit this trip type and what's known
   about navigating each.
3. Research each relevant portal, log findings in
   `clients/<client-name>/research.md` as you go (raw notes are fine).
   - For cruise options, capture a ship photo and a stateroom-type photo
     while you're on the portal, and save them to
     `clients/<client-name>/images/` — `<option-name>-ship.jpg` /
     `<option-name>-stateroom.jpg`. **Prefer pulling the real source image
     URL over screenshotting the page** — a full-page screenshot picks up
     nav bars, sidebars, and whatever carousel slide happens to be active,
     which crops badly. Most ship/stateroom photos are CSS
     `background-image`s, not `<img>` tags, so `browser_snapshot` won't
     surface them — instead use `browser_evaluate` to scan for elements
     with a non-`none` `backgroundImage` and pull the URL out of it, e.g.:
     ```
     () => Array.from(document.querySelectorAll('*'))
       .map(el => getComputedStyle(el).backgroundImage)
       .filter(bg => bg !== 'none')
     ```
     then download the clean URL directly (`curl`) rather than
     screenshotting. Only fall back to `browser_take_screenshot` with a
     specific element `target` (not a full-page shot) when no clean source
     URL exists. Note in `research.md` which option each image belongs to
     so the right one lands in the final quote.
   - **Report to the user in the chat as you go.** Every time you research
     sailings for a quote, send the user a message in the conversation
     listing, for each sailing researched or quoted: ship, sailing dates,
     embark/disembark port, the exact day-by-day itinerary (ports with
     arrive/depart times), and every promo/offer code used or considered
     (exact code as the portal shows it, offer name, price, perks, terms,
     expiry). The user tracks these in the chat to book the exact offer
     later, so don't leave them only in `research.md`.
4. Build a comparison table of the viable options (price, dates, inclusions,
   cancellation policy, commission if known). Always check for and call out
   promotions/extras — onboard credit, discounted or free gratuities,
   excursion/spa/dining discounts, drink packages, etc. — and fold them into
   the "Inclusions" the client sees; these often decide which option looks
   best, so don't let them get buried in portal fine print.
5. Draft the client-facing quote in two forms (see `templates/`):
   - `quote-email.md` — plain text ready to paste into an email
   - `quote-branded.html` — branded document (agency letterhead) that can be
     printed/exported to PDF; for cruise quotes, use the ship/stateroom
     photos saved in `clients/<client-name>/images/`
6. **Before calling `quote-branded.html` done, inline every image as a
   base64 `data:` URI** — logos and ship/stateroom photos alike — instead of
   referencing them by file path. A file path (relative like
   `../../assets/logo.png` or absolute like `/Users/<you>/...`) only
   resolves on this machine; the moment the file is emailed to a client or
   opened anywhere else, those images break. It also avoids local-browser
   file-access quirks that can silently fail to render images referenced by
   path even on this machine. A quick way (resolves relative paths against
   the quote's own folder):
   ```
   python3 -c "
   import base64, re, pathlib
   p = pathlib.Path('clients/<client-name>/quote-branded.html')
   html = p.read_text()
   def repl(m):
       src = m.group(1)
       if not src.startswith(('data:', 'http:', 'https:')):
           img = (p.parent / src).resolve()
           mime = 'image/png' if img.suffix.lower() == '.png' else 'image/jpeg'
           b64 = base64.b64encode(img.read_bytes()).decode()
           return m.group(0).replace(src, f'data:{mime};base64,{b64}')
       return m.group(0)
   p.write_text(re.sub(r'<img src=\"([^\"]+)\"', repl, html))
   "
   ```
   If `python3` errors with an Xcode license message, use the Homebrew one
   instead (`/opt/homebrew/bin/python3.12` or whatever `ls
   /opt/homebrew/bin/python3*` shows) — same script, just a different
   interpreter path.
7. Save both into `clients/<client-name>/`.

## Project structure

- `portals.md` — list of portals in use, URLs, what each is good for, and any
  quirks/selectors learned while automating them.
- `templates/intake.md` — client request intake fields.
- `templates/research.md` — per-option research notes template, including
  where to log captured ship/stateroom images.
- `templates/quote-email.md` — email-ready quote template.
- `templates/quote-branded.html` — branded HTML quote template.
- `clients/<client-name>/` — one folder per client: `request.md`,
  `research.md`, `images/` (captured ship/stateroom photos), and the
  generated quotes. **Git-ignored** — client data never goes in the repo.
- `branding.md` — agency/host-agency branding details that fill the
  templates' `{{AGENCY_*}}` / `{{HOST_AGENCY_*}}` placeholders.
  **Git-ignored**; `branding.example.md` is the committed blank version.
  Logo files live in `assets/` (also git-ignored).
- `.mcp.json` — project Playwright MCP server (persistent browser profile in
  `~/.pandp-agent/browser-profile`, outside the repo, so portal logins are
  never committed).
- `playwright-config.json` — passed to the Playwright MCP via `--config`.
  Overrides Playwright's default `--disable-blink-features=AutomationControlled`
  launch flag (which hides automation from sites) with an empty value, so
  the browser doesn't disguise itself. Chrome still shows an "unsupported
  command-line flag" banner for the empty flag — harmless, ignore it.

## Status

All five portals in `portals.md` — six cruise lines: Carnival, MSC, Royal
Caribbean + Celebrity (both CruisingPower), Norwegian, Disney — have had a
pilot search — check each entry for what's
confirmed vs. still to fill in. Branding: check `branding.md`.
