# DEPLOY - Vendor agreement page (index.html)

Prepared by Agent U, 6 October 2026, on the house brief for token pages (counsel/AGENT_FORM_BRIEF.md). **Nothing here has been done. No repository exists and nothing is public.** This session has no gh CLI and no git credential, and Agent U does not publish.

## The one step that puts it live

1. Create a **public** repository under the Drop-Shot-Coffee GitHub organization named **Vendor-Agreement**.
2. Upload **index.html only**, at the root. Do not upload DEPLOY.md or UX_NOTE.md: a Pages repository is public source and these notes are internal.
3. In Settings, Pages, choose Deploy from a branch, branch main, folder root.

The page then serves at:

    https://drop-shot-coffee.github.io/Vendor-Agreement/?t=<token>

The test link has this shape, with the test token the loader printed:

    https://drop-shot-coffee.github.io/Vendor-Agreement/?t=<test token>

This is the house route used for the existing Pages applications (PreWire-Readiness-Tracker, Signing, Ledger, Branding, Project-Plan, Project-Control, Pre-Wire-Diligence, CL33-Cafe-Proposal-Discovery), and the same token-in-URL shape as the Moneyball confirmation page.

## The go the principal gives

Either the principal performs the three steps himself, or he says, in chat, **"go: publish the vendor agreement page"**, and an agent with a GitHub credential pushes the one file. Publishing the page does not send anything to anyone. The Designer's real link exists only after `vendor_agreement_admin.py send --ref <ref>`, which is run on the principal's word alone and is a separate go.

## What must be true first, in order

1. Counsel Comm's template and instance JSON loaded with `vendor_agreement_admin.py load ... --test`, and the TEST token printed.
2. Agent U's end-to-end check against the live database with that test token, in headless Chrome, served from localhost: the agreement loads, all four exhibits open, a test acceptance records, the executed copy shows his dates, and the reload shows the executed copy. (The edge function's CORS allows localhost and drop-shot-coffee.github.io, so the check can run before deploy.)
3. Counsel Comm reads the agreement as rendered on the page.
4. The Brand Guardian's check of the page against the house style, for the principal's acceptance. Agent U does not certify its own page.
5. The principal approves the agreement text. Nothing reaches the Designer before that.

## What the file carries

- The project URL `https://iwufnpzlwxzzedwcrjrk.supabase.co` and the **publishable key only**, exactly as the existing Pages applications do. The key can execute `get_agreement` and `accept_agreement` and call the exhibit function, and nothing else of this portal. No secret key, no service-role key, no other credential.
- A Content-Security-Policy that allows network calls to that one project and nothing else, inline script and style only, no external fonts, scripts or images, and `no-referrer`, so the token in the URL is not passed on when an exhibit opens.
- No cookies, no local storage, no analytics.

**If the project URL ever changes,** both the `SUPABASE_URL` constant and the `connect-src` entry in the Content-Security-Policy meta tag change with it, or every call fails.

## Changing things after deploy

- The agreement text, the exhibits and the schedule limits live in the database. Changing them needs no redeploy. A text changed after the Designer opened the page is refused at Accept with `text_changed`, and the page tells him to reload.
- A change to the page itself is a new index.html uploaded over the old one. Hard-refresh past the GitHub Pages cache afterward.

## After deploy

Open the test link on a phone and on a desktop. Open each exhibit. Do not send the Designer's link until the principal says so. Test agreements and their acceptances are kept and never deleted.
