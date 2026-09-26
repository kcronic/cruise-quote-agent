# Branding

Copy this file to `branding.md` (which is git-ignored) and fill it in.
The quote templates' `{{AGENCY_*}}` / `{{HOST_AGENCY_*}}` placeholders and
brand colors are filled from here. Put logo files in `assets/` (also
git-ignored).

- **Agency name** (`AGENCY_NAME`): Your Agency Name
- **Host agency name** (`HOST_AGENCY_NAME`): Your Host Agency (if you're an
  affiliate — shown small on quotes with an "affiliate of" line; delete the
  affiliate line from the templates if you don't have one)
- **Agent name / title** (`AGENT_NAME`): optional — leave blank to sign as
  the agency
- **Contact email** (`AGENCY_EMAIL`): you@example.com
- **Contact phone** (`AGENCY_PHONE`): 555-555-5555
- **Website** (`AGENCY_WEBSITE` as displayed / `AGENCY_WEBSITE_URL` full
  link): example.com/youragency / https://example.com/youragency
- **Logo files** (in `assets/`):
  - `AGENCY_LOGO_FILE`: your-agency-logo.png (primary)
  - `HOST_AGENCY_LOGO_FILE`: host-agency-logo.png (affiliate mark)
- **Brand colors** (set in the `:root` block of `templates/quote-branded.html`):
  - Primary: `#1B2A4A`
  - Accent: `#C9A227`
  - Host agency accent (affiliate mark only): `#3D7C8C`
- **Required disclaimer / accreditation text** (`REQUIRED_DISCLAIMER_TEXT`):
  any E&O/accreditation or cancellation-policy language your host agency
  requires on client-facing quotes.
