# Portals

One entry per portal. Fill in URL and notes as we pilot each one — the goal is
that after a couple of runs, Claude can navigate each portal reliably without
re-discovering it from scratch every time.

## Which portal for which cruise line

| Cruise line | Portal | Status |
|---|---|---|
| Carnival Cruise Line | GoCCL Navigator | Piloted 2026-09-26 |
| MSC Cruises | MSC Book | Piloted |
| Royal Caribbean | CruisingPower | Piloted |
| Celebrity Cruises | CruisingPower (same login — pick "Celebrity" in the Brand dropdown) | Piloted 2026-09-26 |
| Silversea | CruisingPower (Brand dropdown) | Not yet searched |
| Norwegian Cruise Line | Norwegian Central → Quest (second sign-in) | Piloted 2026-09-26 (promos/taxes step not yet reached) |
| Disney Cruise Line | Disney Travel Agents (disneytravelagents.com) | Piloted 2026-09-26 (day-by-day itinerary not yet captured) |

## Template for a new entry

### <Portal name>
- **URL:**
- **Login:** manual, by the user, in the Playwright browser window.
- **Best for:** (e.g. Caribbean cruises, all-inclusive resorts, air+hotel packages)
- **Search flow notes:** (steps to get from homepage to a results page —
  which fields matter, any quirks like date pickers or multi-step forms)
- **Results extraction notes:** (where price/cabin/inclusions live on the
  results page, anything that trips up automated reading e.g. lazy-loaded
  content, iframes)
- **Known issues:** (CAPTCHAs, rate limits, session timeouts)

---

### GoCCL Navigator (Carnival Cruise Line trade portal)
- **URL:** https://www.goccl.com
- **Login:** manual, by the user, in the Playwright browser window. Username
  field id'd as "Username:", password as "Password:". Note: a "Privacy
  Notice" cookie dialog can sit on top of the form on first load and steal
  clicks/keystrokes — close it (button "Close cookie policy") before
  filling in the form if login seems to silently fail.
- **Best for:** Carnival Cruise Line sailings — individual staterooms and
  group bookings.
- **Search flow notes:** After login, lands on `/` (Home). Main nav tabs:
  Booking, Ships & Sailings, Marketing Tools, Booked Clients, Knowledge &
  Training, Agent Rewards & Programs. To start a new individual search:
  "Individual Stateroom" link → `/app/bookingengine`. Group bookings via
  `/BookingEngine/Groups/SailingSearch.aspx`. Can also jump straight to
  results with query params, e.g.
  `/app/bookingengine/search-results?amountOfGuests=2&embarkationPorts=MIA&sailingDateStart=112026&shipCode=&...`
  — `sailingDateStart` is `MMYYYY` (112026 = Nov 2026), ports are 3-letter
  codes (MIA = Miami).
  - Search form filters are dropdown chips: "Dates" is a month grid
    (click a month button, e.g. "Nov" under 2026 — the first "Nov" in the
    list is the earliest year). Picking a month greys out (`disabled`)
    ports/ships with no sailings that month, so the "Depart From" list
    itself tells you which ports are live.
  - **No adult/child split anywhere in the search** — only a total guest
    count. The rate-code URL carries one blank `birthDates=` per guest, so
    child pricing probably only kicks in once birthdates are entered later
    in the booking flow (not yet verified). Quote a family as N guests and
    say the child's fare may come in slightly lower.
- **Results extraction notes:** Search form (Dates/Sail To/Ports of
  Call/Ship/Depart From/Duration) sets amountOfGuests=2 by default — after
  submitting, use the "Guests" +/- stepper in the results page sidebar to
  set the real party size, then click "Apply Filters" (it stays disabled
  until you touch a filter) or prices won't reflect the true per-person
  rate for that guest count. The "Guests" panel starts *expanded* —
  clicking its header collapses it. After applying, the page URL still says
  `amountOfGuests=2`, but the prices do update (verify by watching a price
  change) and "Select Sailing" carries the right count forward.
  Results show 10 sailings at a time — click "Load More" until it
  disappears to see the full count in the "N Available Sailings" heading.
  "From" prices on the results list are per-person for the guest count
  applied, taxes & fees included, and are the **lowest across all offer
  codes** for that sailing. Clicking "Select Sailing" opens a rate-code
  page (`/app/bookingengine/rateCode?...&shipCode=CB&itineraryCode=WSI&sailingDate=2026-11-29`)
  showing every active promo/offer code with per-category pricing — a
  category showing "N/A" there is genuinely sold out for that guest count
  (checked across all offer codes, not just one).
  - **Always read each offer's terms before quoting.** The cheapest "From"
    price is often a restrictive fare, e.g. "Pack & Go" (PUG): Carnival
    picks the cabin, full non-refundable payment at booking. Each price
    tile has a small icon button beside the category name; clicking it
    opens the offer terms (perks such as onboard credit, deposit, final
    payment, expiry) in a popup that `browser_snapshot` doesn't show — read
    it with `browser_evaluate`:
    `() => document.getElementById('popup-root')?.innerText`.
    Press Escape to close it — while it's open it blocks clicks on the
    price buttons underneath.
  - "Free 3rd & 4th Guest" offers aren't automatically cheapest for a
    family — their base fare can be higher than another sale's. Compare
    the per-category totals across offers, net of onboard credit.
- **Ship/stateroom photos:** on each ship's own page (`/Ships/<Ship-Name>`,
  e.g. `/Ships/Carnival-Celebration`). Both are CSS `background-image`s, so
  use the usual `backgroundImage` scan. Both download with plain `curl`, no
  login needed.
  - Ship: the hero banner at the top (`goccl.com/-/media/project/goccl/...
    /<ship>-banner*.jpg`). Drop the `?h=&w=&hash=` query string when
    downloading. ~1400×380, a wide side profile — crops well as a header.
  - Staterooms: one carousel per category, served from carnival.com at a
    predictable URL:
    `https://www.carnival.com/-/media/Images/Ships/<SHIP>/StateroomCodes/<SHIP><CAT>_1200x350.jpg`
    — `<SHIP>` is the 2-letter ship code (CB = Celebration, also in the
    rate-code URL's `shipCode=`), `<CAT>` is the category code (`IS…`
    interior, `OS…` ocean view, `OB…` balcony, `SU…` suite; e.g. `OBSTB` =
    standard balcony, `OBSPB` = Cloud 9 Spa balcony). **Skip `_fl.jpg`
    files — those are floor plans, not photos.** Some suites also have
    `_1200x350_V2.jpg` / `_ext.jpg` alternates. Anchors like `#OS`/`#OB`
    on the page sometimes jump to a sub-variant (e.g. "Cloud 9 Spa
    Balcony") rather than the plain category — pick the standard code for
    a plain Balcony quote.
- **Promotions:** Home page surfaces current promos (e.g. "More Time, More
  Perks Sale", "Early Saver" sales) with onboard-credit/deposit-reduction
  details — worth checking here for anything applicable to a client's dates
  before finalizing a quote.
  Each offer on the rate-code page also has its own expiry — sale fares
  often end within days, so note the expiry date in the quote.
- **Known issues:** none yet beyond the cookie-dialog quirk above. Piloted
  2026-09-26 — see `clients/_carnival-pilot/research.md`.

---

### MSC Book (MSC Cruises travel agent trade portal)
- **URL:** https://www.mscbook.com — country picker first (`/changecountry` →
  region → "United States English" → lands on `/us/welcome` logged out or
  `/us/home` once logged in).
- **Login:** manual, by the user, in the Playwright browser window. A cookie
  consent banner appears first — click "Agree and close" before doing
  anything else. Click "LOG IN" in the header to open a login modal
  (Username/Password fields).
- **Best for:** MSC Cruises sailings, individual staterooms and groups.
- **Known issue — overlay interception is constant on this site.** Two
  separate overlays repeatedly intercept normal clicks and make
  `browser_click` time out:
  1. A sticky header (`<header class="fixed ...">`) that overlaps controls
     positioned near the top of the viewport.
  2. An **Appcues** onboarding/survey widget (`div.appcues`, plus a
     `"How would you rate..."` feedback dialog after searching) that
     reinjects itself on every page load.
  Fix: before interacting, run
  `() => document.querySelectorAll('.appcues').forEach(el => el.remove())`
  via `browser_evaluate`, and dismiss any feedback dialog
  ("Close dialog" button) that appears after a search. Real (trusted)
  `browser_click`/`browser_type` still work once the overlay is gone —
  **don't** dispatch synthetic `.click()`/`MouseEvent`s via `browser_evaluate`
  as a workaround, the date-picker and toggle buttons on this site are built
  on a library that ignores untrusted events and silently no-ops.
- **Search flow notes:** Home page search widget: Departure dates (calendar,
  see below), Passengers/Stateroom stepper (Adult 18+, Child 12-17, Junior
  Child 2-11, Infant), then filter chips — Embarkation Port, Destination,
  Ship, Cabin Type, "All filters" (expands all of the above plus Duration,
  Promotions, Price Range into one panel). Selecting an embarkation port
  changes which ships/destinations are selectable (e.g. Port Canaveral vs.
  Miami surface different ship quick-picks).
  - **Departure dates widget:** has "Months" and "Days" tabs. Clicking a
    month in "Months" only *navigates* the "Days" calendar to that month —
    it does not select a date range by itself. You must switch to "Days"
    and click actual day cells (real trusted clicks) to populate the field;
    clicking one day sets start=end, click a second day to set the actual
    end of the range. For "sometime in a given month" requests, just click
    the 1st and the last day of that month to pull every sailing in it.
  - **Ship quick-pick chips are incomplete/curated** — they do NOT list
    every ship serving a port (e.g. newer ships like MSC World America
    didn't appear in the Miami quick-picks even though it demonstrably
    sails from there). Don't conclude a ship isn't available just because
    it's missing from these chips — leave the Ship filter blank and check
    the actual results list instead, which reflects real inventory.
  - Once fields are set, "Search Cruises" stays disabled until dates AND
    at least one other filter register — check via
    `document.querySelectorAll('button')` `.disabled` if unsure, since the
    button's visual (orange) styling doesn't change when disabled.
- **Results extraction notes:** Results list groups by itinerary; each
  itinerary card lists ship, night count, port list, and price tiers by
  departure day-of-month. **Prices are labeled "PC" = per cabin (the whole
  stateroom's total for the occupancy you searched), not per person** —
  don't double-count by multiplying by guest count. Click "Explore Deals"
  on a specific itinerary card to see the true per-category breakdown
  (Interior/Ocean View/Balcony/Suite, each "From $X P.C.") for the exact
  occupancy searched, plus the full day-by-day itinerary with arrival/
  departure times. Careful when multiple cards share a departure date (e.g.
  a 7-night sailing and a 14-night back-to-back both starting the same day)
  — they can look similar in a locator's matched index; confirm you opened
  the right one by checking the night count and `cruise-id` in the
  resulting URL before reading its price.
- **Ship/stateroom photos:** not exposed as `background-image` on this site
  (unlike GoCCL) — plain `<img>` tags, but the search/results pages don't
  have good ones. Instead go to **Fleet Overview** (top nav) → pick the
  ship → this opens a wrapper page whose real content is in an `<iframe>`;
  find its `src` (pattern `https://www.mscbook.com/pages/sdl/us_us/<ship-
  slug>.html`) and navigate to that URL directly rather than reading images
  out of the iframe in place. That page has a full set of clean marketing
  photos: a wide ship-silhouette shot and one photo per stateroom category
  (Balcony, Ocean View, Interior, Suite variants) at consistent
  `B2B_TA_<code>__<Category-Name>_<size>.jpg` URLs — download those directly
  rather than screenshotting.
- **Promotions:** Home page "All Promotions" section surfaces current
  offers (onboard credit, kids-sail-free, flash sales) — same pattern as
  GoCCL, worth checking before finalizing a quote.

---

### CruisingPower (Royal Caribbean Group trade portal — RC, Celebrity, Silversea)
- **URL:** https://www.cruisingpower.com (redirects to
  `secure.cruisingpower.com/login`).
- **Login:** manual, by the user. One login covers all three brands
  (Royal Caribbean International, Celebrity Cruises, Silversea) — a
  "Brand" dropdown on the search widget switches between them.
  **Known issue:** on first load the Username/Email and Password fields
  can come up `disabled` with the sign-in button stuck on "Loading..." for
  several seconds — console shows a CSP violation blocking a worker script
  (`Creating a worker from 'blob:...' violates ... Content Security
  Policy`), which looks like a bot-mitigation/device-check script failing
  in this automated context. **Don't try to work around the CSP block or
  force the disabled fields** — it resolved on a plain reload/re-navigate
  in testing (fields became enabled after a few seconds), so try that
  first; if it stays stuck, stop and hand back to the user rather than
  digging further, per the read-only/no-evasion rule.
- **Best for:** Royal Caribbean International, Celebrity Cruises, and
  Silversea sailings — one search widget, brand-filtered.
- **Search flow notes:** Home page has an "Espresso" search widget:
  Cruise Type, Brand (All/Royal Caribbean/Celebrity/Silversea), Ship,
  Destination, Port of Call, Departure (date), Adults, Children, Currency,
  plus a "Promotion Qualifiers" section (Residency, Loyalty Number, Promo
  Code). (Navigation/results notes to fill in as we search.)
- **Guest limits:** Both the home page "Espresso" widget and the "Create an
  eQuote" tool are **per-stateroom** searches capped at 3–4 guests — neither
  supports searching a large party (e.g. 8+ guests) as one booking. There is
  no self-service "group rate" search: "Group Travel Center" (under
  `Booking` in the hamburger nav) only manages *existing* group bookings
  (tasks/reports) — real group rates/perks require a formal request through
  that desk, not a live self-service quote. For a large party, search at
  max per-stateroom occupancy (e.g. 3 or 4) to get real baseline pricing,
  then multiply by however many staterooms the party actually needs — and
  say plainly to the client/user that this is a *budgetary estimate* from
  standard rates, not an official group quote.
- **eQuote tool** (`Booking → Create an eQuote`, opens in a new tab — use
  `browser_tabs` to switch to it): a proper search tool with Cruise
  Type/Brand/Ship/Destination/Guests/Departure+Nights/Promotional
  Qualifiers, better suited to real per-sailing pricing than the home page
  widget. Selecting a Ship narrows the Destination dropdown to only that
  ship's actual current deployment (e.g. selecting "Wonder of the Seas"
  left only "Bahamas" selectable) — a real signal about what that ship is
  actually sailing, not a bug.
  - **Departure date field:** single date + Nights by default; click
    "Show Date Range" to reveal a Return/end-date field for a broader
    window. A single fixed date + nights combo can come back "No sailings
    available" even when sailings exist nearby — always widen to a date
    range (e.g. the whole month) before concluding nothing's available.
  - The date-range calendar's "next month" control has no accessible name
    (icon-only) — find it via
    `getByRole('button', { name: 'Move forward to switch to the next month.' })`
    rather than by name.
  - Results table columns (Interior/Oceanview/Balcony/Deluxe-Suite) show
    the **blended average per-person price across the whole stateroom**
    (e.g. 1st+2nd guest at full fare + 3rd guest at a reduced/near-taxes-only
    rate, all divided by guest count) — not a flat per-person rate. Click
    "preview" on a row to see the real per-guest breakdown (1st/2nd vs.
    3rd/4th pricing) and confirm the math before quoting; the average ×
    guest count in the room should equal the total from the preview.
  - "preview" also shows a full-bleed ship hero photo — pull it via
    `browser_evaluate` scanning for `backgroundImage` as usual, but note
    the image URL lives under authenticated `secure.cruisingpower.com/...`
    — `curl` alone won't work (returns an HTML login page instead of the
    image); navigate the Playwright browser itself to the image URL (reuses
    its session) and screenshot the `img` element instead.
  - No stateroom-category photos on CruisingPower itself, but Royal
    Caribbean's **public consumer site** has them: go to
    `royalcaribbean.com/cruise-ships/<ship-slug>/rooms` (note: `/rooms`,
    not `/staterooms` — that 404s). It has a clean 4-tile grid
    (Interior/Ocean View/Balcony/Virtual Balcony), each a real photo —
    grab via the same `<img>`/background-image scan as usual. These are
    generic fleet-wide category shots (filenames trace to whichever ship's
    photo shoot Royal Caribbean used), not necessarily the specific ship
    you searched — same caveat as MSC's generic category photos. Unlike
    CruisingPower's authenticated eQuote images, this consumer site is
    public — plain `curl` works fine, no need to route through the
    Playwright session.
- **Known issues:** the login-form-stuck-loading quirk above; per-stateroom
  guest caps described above.

  **Celebrity Cruises** (piloted 2026-09-26 — see
  `clients/_celebrity-pilot/research.md`): same login; use eQuote
  (home page → "Create an eQuote" link, opens in a new tab) with Brand =
  "Celebrity". Notes:
  - **No departure-port filter or column in eQuote.** Destination is a
    region ("Caribbean"), and the results table doesn't show the port —
    you only see it in each sailing's "preview". In the Nov 2026 pilot,
    Celebrity Caribbean sailings left from Tampa, Port Canaveral, Miami
    *and* San Juan (others may exist), so always open the preview before
    assuming a Florida port.
  - The "Return" date in "Show Date Range" acts as the **latest departure
    date**, not a return date. Results URL carries
    `startDate=/endDate=/region=CARIB/guestCount=/count=30` — results may
    be capped at 30 rows.
  - Each price cell shows its **promo code and deposit type** inline,
    e.g. "$976.94 · BOGO75 NRD · Non Refund. Deposit" or
    "$495.59 · EXCITINGDLS NPK · Refund. Deposit". eQuote doesn't explain
    the codes — look up their terms before quoting.
  - Table price = blended per-person average for the guest count. Preview
    shows the 1st/2nd vs. 3rd guest split, but **only for the cheapest
    available category**, not the one you're quoting.
  - 3-guest occupancy sells out fast (lots of "Sold Out") — few Celebrity
    cabins sleep 3.
  - Row ids (`tr[id^="package-"]`) carry the package code + date, e.g.
    `package-AX07W682-2026-11-14-[3]` (AX = Apex, 07 nights, W = western
    itinerary) — handy for clicking a specific row's "preview":
    `[id="package-<code>-<date>-[3]"] >> role=button[name="preview"]`.
  - Preview is a modal (`.equote__package-detail-modal`) that
    `browser_snapshot` doesn't show — read it with `browser_evaluate`.
    Its itinerary loads a second or two after it opens; re-read if you
    only get the first few days. Close with its "DISMISS" text.
  - **Photos:** the preview has both a ship hero and per-category
    stateroom photos (Celebrity calls balconies "Veranda"), served from
    `secure.cruisingpower.com/trade/...` — these download with plain
    `curl` (no login), unlike the Royal Caribbean eQuote images.

---

### Norwegian Central (Norwegian Cruise Line trade portal)
- **URL:** https://norwegiancentral.ncl.com — redirects to an NCL sign-in
  page on `sso.ncl.com/secureauth1/?SAMLRequest=...`. Always start from
  `norwegiancentral.ncl.com`, **not** a saved `sso.ncl.com` link — the
  `SAMLRequest` in it is a one-time sign-in request (unique ID +
  timestamp) generated per visit, and a stale one can fail after login.
  (The old `nclcentral.com` address
  has an expired security certificate — don't use it, and don't click
  through the certificate warning.)
- **Login:** manual, by the user. Username + Password ("Seaweb"
  credentials). "I Forgot My Credentials" and "New agency" links on the
  same page. Registration help: per NCL, email their Norwegian Central
  support address (see the login page).
- **Best for:** Norwegian Cruise Line sailings.
- **What's inside (seen 2026-09-26):** after login, the home page shows
  the logged-in agent's username and agency name, and tiles for NCLU
  (training), Marketing Headquarters, **Quest Booking System**, and
  Partners First Facebook; top nav has Quest, Marketing HQ, Itineraries,
  Deck Plans, Ships. A cookie "Privacy" banner appears first — "Close" it.
- **Booking engine is Quest** (`https://quest.nclh.io/travel-partners/northamerica`),
  not Seaweb as NCL's older announcement said. **Quest needs its own
  second sign-in** ("Please sign in using your Book NCL username &
  password", on `sso.ncl.com/SecureAuth36/`) — the Norwegian Central login
  doesn't carry over. Hand this to the user.
- **Search flow** (piloted 2026-09-26 — see
  `clients/_ncl-pilot/research.md`): Quest is a **booking workflow**, not
  a separate search tool — every search runs inside a "New Reservation"
  (URL carries a `bid=` booking id) with steps CRUISE VACATION → STATEROOM
  TYPES → STATEROOMS → PRICING & OFFERS → GUEST DETAILS → … → PAYMENT.
  The header may show a pending item from the agent's earlier activity
  (e.g. another sailing left in a draft) — leave it alone.
  - Filters are Angular Material fields: Staterooms & Guests (Adults
    **21+**, Children **2–20**, Infants under 2 — click the field, use the
    +/− buttons, Escape to close), Residency, Destinations, Ships, Dates
    (Month/Day tabs → click a month → "Apply"), Durations (free text,
    e.g. `5-7`), Ports of Departure, Ports of Call. For the port fields,
    the floating label intercepts clicks and typing doesn't search — click
    the little ▾ arrow icon to open the checklist, click the port, then
    "Apply".
  - Results: 10 per page ("Next page" button), columns per category
    (Inside/Oceanview/Balcony/Club Balcony Suite/Suite/Haven). **Prices
    are lead prices per person for guests 1–2 only** (footnote says so) —
    the 3rd guest is priced separately.
- **Results extraction:** click a row's itinerary name to open it: shows
  the **full day-by-day itinerary with times** right away ("port order may
  vary by date of departure"). "Choose Cruise Vacation" → STATEROOM TYPES:
  lists only categories that fit the party (for 3 guests, the "Family"
  categories — the cheaper 2-person lead prices from the list disappear),
  with "Price 1-2" (each) and "Price 3+" columns, sale price next to a
  struck-through regular price. Page scrolls inside a panel — use
  `scrollIntoView` before screenshotting. Taxes/fees not stated here.
  - **Stop at STATEROOM TYPES.** "Assign" / "Continue to stateroom
    selection" picks an actual cabin (may place a hold); PRICING & OFFERS
    — where NCL's perks packages (drinks, dining, Wi-Fi, excursions) are
    chosen — only comes after that. Get the user's OK before going further.
- **Photos:** each category's "View" link opens a gallery of public
  `www.ncl.com/sites/default/files/...` images (plain `curl` works).
  Includes a "Schematic" floor-plan image — skip that one.
- **Known issues:** none yet.

---

### Disney Travel Agents (Disney trade portal — cruises, parks, resorts)
- **URL:** https://www.disneytravelagents.com — **this is the portal to
  use** (confirmed by the user 2026-09-26). Home page once logged in has a
  "Quick Quote Search" widget (Destination dropdown — defaults to "Walt
  Disney World Resort", Dates, Adults (18+)/Children steppers, Hotel),
  plus My Reservations, Offers ("View All Offers"), Disney Travel News,
  and My Quick Links panels.
- **Login:** manual, by the user.
- **Not this one:** `disneycruise.disney.go.com/travel-agent-login/` is a
  different, consumer-site login that asks for agency phone number +
  "affiliation" (field labels render as raw placeholder text). Don't
  send the user there.
- **Best for:** Disney Cruise Line sailings; also Walt Disney World /
  Disneyland packages.
- **Search flow** (piloted 2026-09-26 — see
  `clients/_disney-pilot/research.md`): Quick Quote → Destination
  "Disney Cruise Line" (the date/hotel fields disappear) → Adults (18+)
  and Children steppers — **Disney takes real child ages** ("Age of
  Children at Time of Travel", 6mo+ to 17). "Continue" hands off to
  `disneycruise.disney.go.com/cruises-destinations/list?...gatewayId=Agent&tokens=...`
  — Disney's consumer site in agent mode (agency banner at the top). So
  that site *is* where searching happens, but only when entered from
  disneytravelagents.com.
  - The page is **slow** — can load blank for several seconds. Wait ~5 s
    before reading it; a blank first load came good on retry.
  - Filters: "Leaving" (month checkboxes), "Departing from" (ports with
    nothing that month are disabled), "3 Guests", "More Filters". Then
    click **"View Dates"** to apply — filters don't apply until you do.
    A "How many nights" row (1–3, 4, 5–6, 7, 8–13, 14+) then appears.
    Filters are also reflected in the URL hash, e.g.
    `#november-2026,fort-lauderdale-florida,port-canaveral-florida,7`.
  - Disney's Florida ports are **Port Canaveral and Fort Lauderdale** (no
    Miami in Nov 2026).
- **Results extraction:** **prices aren't in the page text or the
  accessibility snapshot** — they only render visually. Take a
  `browser_take_screenshot` (fullPage) and crop the price column with
  `sips -c <h> <w> --cropOffset <y> <x>` to read them. "Price from" is the
  **total for the whole party**, taxes/fees/port expenses included (not
  per person), with a "You Save" amount. "Show N Dates" expands each card
  into per-date rows with Inside / Oceanview / Verandah (balcony) /
  Concierge totals and crossed-out regular prices.
  - Most November fares were **"Guaranteed with Restrictions"** — Disney
    assigns the stateroom. No offer code is shown on these screens; terms
    are behind "Learn More" / "Rate & Room Details".
  - The per-date "Continue" button (next step toward booking, where the
    day-by-day itinerary likely lives) didn't respond to automated clicks
    — a hidden popup has its own "Continue" button that matches first.
    Day-by-day itinerary times not yet captured for Disney.
- **Photos:** Disney images come from the public CDN
  `cdn1.parksmedia.wdprapps.disney.com/...` (ship images under
  `.../disney-cruise-line/ships/<ship>/...`); the results page only has
  generic ones — check each ship's page for proper ship/stateroom shots.
- **Known issues:** none yet.

<!-- Next pilot portal goes here -->
