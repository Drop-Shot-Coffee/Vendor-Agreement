# UX NOTE - Vendor agreement page

Agent U, 6 October 2026. For the principal's review. Built on the ux-designer skill, the dsc-house-style skill and the Moneyball confirmation page as precedent. The page is index.html beside this note.

## Summary

One person, the Designer, opens a personal link and has to do four things: read the agreement and its exhibits, enter his details and his delivery dates, consent, and sign by typing his name. The page shows the whole agreement on the left and the form on the right on a desktop; on a phone the agreement comes first and the form follows it. The blanks for his details and dates are marked in the text and fill in as he types, so he sees the exact wording he is accepting. Accept stays off until he has scrolled to the end of the agreement text and everything required is valid, and a short list under the button says what is still missing. On Accept the page hashes the exact agreement text it showed, sends it with his details, and replaces the form with a confirmation (reference, date and time, his name) and the executed copy, ready to print or save as PDF. Opening the link again shows the executed copy, not the form.

## Who, what, success (ux-designer step 1)

- **User:** one designer, an individual contractor, possibly signing for a business. Desktop is likely for reading a long agreement; phone must also work. Reached by a link from Drop Shot.
- **Job:** read the terms and exhibits, enter identity and dates, give consent, sign, and keep a copy.
- **Success:** one acceptance row recorded with correct values and a hash that matches the text he saw, and a printed or saved copy in his hands.
- **Assumption:** he reads on a desktop first. Not tested with him.

## Flow

1. Open link. The page reads `?t=` and calls `get_agreement`.
2. Ready: the exhibit list (rendered from the server's list, sorted by precedence, never hard-coded; four today, A to D), the agreement text, the form.
3. He reads, opens exhibits as needed (each click asks the server for a fresh five-minute link and opens it in a new tab; nothing is stored).
4. He fills the form. The marked blanks in the text fill in as he types.
5. Accept. On success, the confirmation and the executed copy. Print or save as PDF.
6. Later visits with the same link: the confirmation and executed copy.

## Layout

- **Desktop (over 960 px):** two columns in the house 1120 px wrap. Left: the exhibits card and the agreement. Right, 380 px: the form, sticky, with its own scroll, so the live fill can be watched while typing.
- **Phone (960 px and under):** one column. Exhibits, agreement, then the form. A "Skip to the form" link appears on keyboard focus.
- **House look:** the template's tokens, header with the court mark, eyebrow, cards, crema button and footer, carried from the Moneyball page. **The agreement itself is set as a plain legal instrument**, black serif on white, no color and no branding, on screen and in print (dsc-house-style standing warning). Marked blanks are the only color inside the text, and they print plain.
- **Fonts:** no web fonts are loaded, by the build prompt's rule. The house font names stay first in each stack and fall back to the system sans and serif. The Moneyball page loads Google Fonts; this page does not.

## States, with their copy

| State | When | What shows |
|---|---|---|
| Loading | while `get_agreement` runs | "Loading your agreement." |
| No token | no `?t=` | "This page needs the link you were sent." / "Open the agreement from the link you received. The link carries your personal access code." |
| Invalid or expired | malformed token (not 43 URL-safe characters, checked before any call), or the server says `invalid_or_expired` | "This link does not work." / "It may be incomplete, or it may have expired. Open the link exactly as you received it, or ask Drop Shot Coffee for a new one." |
| Not open | status is not `sent` and there is no acceptance, or the server says `not_open` | "This agreement is not open for signing." / "It has not been released for acceptance, or it has been withdrawn. Nothing has been recorded. Ask Drop Shot Coffee if you expected to sign it." |
| Network error | the call fails | "We could not reach the server." / "Check your connection and try again. Nothing has been recorded." with Try again |
| Offline at load | the browser reports offline | "You are offline." / "Connect to the internet and open this link again." with Try again |
| Ready | `sent`, no acceptance | the exhibits, the text, the form |
| Submitting | after Accept | the button reads "Recording your acceptance" and is held |
| Done | success, or reload after acceptance | "Thank you. Your acceptance is recorded." Reference, Accepted (Pacific time), Accepted by, Text fingerprint (the executed text's SHA-256), Print or save as PDF, the executed copy |
| TEST | `is_test` true, in every state that has an agreement | a yellow bar: "TEST - This is a test agreement for review." It also prints |

No state offers the form unless the server has returned an open agreement.

## The form

- Full legal name; Mailing address (multi-line); Email; "I am signing for a business" which reveals Business name, Type of business (suggestions: limited liability company, corporation, limited partnership, general partnership), State where it is registered. Entity fields are optional, but once the box is ticked all three are required, so the party description is never half-written. Unticked, they are sent as empty strings.
- **Your delivery dates** (shown only when the agreement has schedule limits): First directions presented; Refinement complete; Final files, Line A; Final files, Line D. Native date pickers, with `min` set to the server's `today_pacific` and `max` to the right limit from `schedule_limits`, and the latest date written under each field.
- "I consent to sign electronically"; "I have read and agree". The labels are the build prompt's words, unchanged. Any further consent disclosure is counsel's wording and lives in the agreement text.
- Typed signature, with the hint "Type your full legal name exactly as you entered it above." and a live line: "Matches your full legal name." or "Does not match your full legal name yet." Case and spacing are ignored, as on the server.

### Date rules on the page (the same rules the server applies)

Each date required and real; on or after `today_pacific`; first directions no later than `directions_by`; refinement and both final-files dates no later than `final_by`; first directions before refinement complete; refinement complete on or before both final-files dates. Messages: "Enter the date for first directions." / "Enter a real date." / "This date is before today, Tuesday, October 6, 2026." / "First directions must be presented by Monday, October 12, 2026." / "Refinement must be complete by Friday, October 23, 2026." / "Final files must be delivered by Friday, October 23, 2026." / "Refinement complete must come after first directions." / "Final files for Line A cannot come before refinement is complete." The dates in these messages come from the server's values, not from the page.

### Server codes, in plain words

| Code | Message |
|---|---|
| `invalid_or_expired` | goes to the invalid state |
| `already_accepted` | "This agreement has already been accepted. Showing the executed copy." then reloads into Done |
| `not_open` | goes to the not-open state |
| `missing_<field>` | "Please enter your full legal name." (and the same for each field, in words) |
| `invalid_<field>` | "Please check the date for first directions; it is not in a form we can accept." (and the same for each field) |
| `invalid_date` | "One of your delivery dates is not a real date." |
| `<field>_before_today` | "The date for first directions is before today. Choose today or a later date." |
| `date_directions_after_limit` | "First directions must be presented by Monday, October 12, 2026." |
| `date_refinement_after_limit` | "Refinement must be complete by Friday, October 23, 2026." |
| `date_final_a_after_limit`, `date_final_d_after_limit` | "Final files for Line A (or D) must be delivered by Friday, October 23, 2026." |
| `date_order_directions_refinement` | "Refinement complete must come after first directions." |
| `date_order_refinement_final` | "Both final-files dates must be on or after the date refinement is complete." |
| `consent_required` | "Tick both boxes: consent to sign electronically, and that you have read and agree." |
| `signature_mismatch` | "Your typed signature must match your full legal name." |
| `text_changed` | "The agreement text changed after this page loaded. Nothing was recorded. Reload the page and read the current version." |
| anything else | "We could not record your acceptance. Nothing was recorded. Ask Drop Shot Coffee for help and quote this code: <code>." |
| network failure on Accept | "We could not reach the server, so nothing was recorded. Check your connection and press Accept again." |

## Decisions and readings (each reversible)

1. **"Until the text has been shown"** is built as: Accept stays off until the "End of the agreement text." line under the agreement has been on screen. It proves the end was reached, not that it was read.
2. **Accept is `aria-disabled`, not `disabled`.** It stays focusable and announced, and a press while incomplete records nothing and shows the field errors and the list of what is missing. A truly disabled button gives a keyboard or screen reader user no reason.
3. **The hash** is SHA-256, by `crypto.subtle`, over the exact `agreement_md` string the server returned and the page displayed, before any live fill. The live fill changes only the marked blanks on screen; the server writes the final values into the executed text, which carries its own hash.
4. **The party blank** previews as his name, or "Business, acting by Name" when the entity box is ticked. The exact legal wording of the party is set by the server from counsel's pattern at acceptance, and the executed copy shows it. Reference and date-and-time blanks stay grey and read "Set when you accept".
5. **Markdown** is rendered by a small built-in renderer that creates DOM nodes and text only (headings, paragraphs, bold, italic, lists, tables, block quotes, rules, inline code, https links). No HTML in the text is ever interpreted, and nothing untrusted goes through innerHTML. HTML entities from the server's escaping are shown as their characters, as text.
6. **Exhibits** open in a new tab that the page opens at the click and then points at the fresh signed link, so pop-up blockers allow it. If a browser still blocks it, a one-time link appears beside the exhibit and is removed before it expires.
7. **A test agreement** that has already been accepted shows a "Show the form again (test only)" button, so the principal can review the form after a test acceptance. It never appears on a real agreement.
8. **Preview mode,** `?preview=1`, shows a short sample text marked as a sample, makes no network call, and stubs Accept. It is for layout review and self-test only. A real token always takes the live path.
9. **Print** shows the executed copy only, US Letter with 1 inch margins, black serif, no header, footer or color; the TEST bar prints on a test copy.

## Accessibility

- Every input has a visible label tied by `for`; hints and errors are tied by `aria-describedby`; invalid fields carry `aria-invalid`; the form-level error has `role="alert"`; the "still needed" list and the signature line are polite live regions.
- Keyboard: natural tab order, the form heading takes focus from the skip link, the Done view moves focus to the confirmation, and every control shows the template's crema focus ring.
- Targets: inputs 48 px tall, checkboxes 22 px with full-width labels, buttons 44 to 52 px.
- Contrast, computed by the WCAG formula and not measured on a device: body text #15322B on paper #F6F2EA 12.35:1; muted #5E6B63 on paper 5.00:1; error #9B2C1F on white 7.57:1; agreement text #111 on white 18.88:1; blank text #6b4a10 on #FFF6DC 7.45:1; ink on the crema button 7.99:1; the TEST bar 11.66:1.
- **One house value fails AA and is flagged, not changed:** the template's eyebrow, crema-deep #C2872F on paper, is 2.77:1 at 11.5 px (AA needs 4.5:1). The page uses it for three small labels that carry no information the heading beside them does not ("Agreement for your review and acceptance", "Accepted", "Agreement"). Changing a house token is the Creative Director's and the Brand Guardian's call, not this page's. The Moneyball page carries the same value.
- Nothing moves. Errors carry words, never color alone.

## Self-test (6 October 2026, headless Chrome on localhost, own temporary profile)

- No token, malformed token, and a well-formed unknown token (refused by the live `get_agreement`), each at 1440 x 1000 and 390 x 844: the right notice shows, the form, the agreement text and the TEST bar are not rendered. 30 checks, all pass. Offline at load: the offline notice, no form. Pass.
- Preview, driven over the DevTools protocol: exhibits from data in precedence order; blanks marked; live fill of name, party, entity party, email, signature and all four dates in long form; email check; entity rules; every date rule; signature match ignoring case and spacing; the read-to-end gate on a short phone screen; Accept enabled only when all is met; SHA-256 correct against a known digest; the confirmation in Pacific time; the executed copy; the form gone; print media showing only the executed copy and the TEST bar; typed HTML shown as text. 49 checks, all pass.
- The renderer run over counsel's template draft of this date (scratch copy, not shipped): 1 title, 19 sections, 91 paragraphs, 36 list items, 80 bold runs, no stray Markdown characters, no raw braces.
- The file holds the publishable key only, and no em or en dash.

## What is not tested yet

- Anything against a real agreement: `get_agreement` with a valid token, the four exhibit links opening real PDFs, `accept_agreement` recording, the server hash agreeing with the page's, the server's date checks and error codes as actually returned, the executed copy as the server writes it, and the reload into Done. All wait for the test token, in the next turn.
- Real phones and real browsers (Safari on iPhone above all: date picker, pop-up handling, print to PDF). Headless Chrome only so far.
- Screen readers. Labels and roles are in place, but no screen reader has been run.
- Whether the Designer understands the marked blanks and the "still needed" list without help. Only a real user answers that.
- The page served from GitHub Pages. Not deployed.
