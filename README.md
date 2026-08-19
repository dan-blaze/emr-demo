# emr-demo

A small, self-contained mock EMR/EHR page used for [Text Blaze](https://blaze.today) demos.

It looks and behaves enough like a real clinical charting screen — patient banner, left nav,
tabbed note document, form fields, a scored PHQ-9, right-hand patient summary rail — to record
demos or test snippets against, without touching any real system or patient data.

> **This is a mock.** Synthetic data only. Not a real EHR, not affiliated with any EHR vendor,
> and not for clinical use.

## Files

| File | Description |
| --- | --- |
| `index.html` | Main demo page — a psychiatric follow-up note for a depression visit. |
| `alternative_index.html` | Same page with the patient summary rail swapped to a Type 2 diabetes context (recent labs instead of medications). |
| `form_filling.html` | A separate product entirely — "Northstar Charting", a blank new-patient encounter form. Different brand, layout, palette and interaction model. |

Every file is standalone: one HTML file each, with inline CSS, inline JavaScript, and inline SVG
icons. No build step, no dependencies, no network requests, no data persistence — reloading the
page resets everything.

## Running it

Open the file directly in a browser:

```bash
open index.html          # macOS
xdg-open index.html      # Linux
```

Or serve the folder if you want it on a URL (handy for demos where a `file://` path looks wrong
in the address bar):

```bash
python3 -m http.server 8000
# then visit http://localhost:8000/
```

Since `index.html` sits at the repo root, the repo can also be published as-is via GitHub Pages.

## What's on the page

**Top bar** — product logo ("GenericCare"), patient context (`Williams, Katie (2104730)`), alert
icons, and a staff menu. Most icons are decorative; clickable ones raise a toast.

**Left sidebar** — Client Dashboard, Client Information (active by default), Patient Intake
Information, Medication Management / Rx, Client Orders, Client MAR, SmartLinks, and — below its own
separator at the bottom — Patient Form Filling, which is a link out to `form_filling.html`, a
different application altogether. Within this page, two items switch what the main area shows:

| Nav item | Shows |
| --- | --- |
| Client Information | The tabbed Psychiatric Note — SOAP, Service, Note, PHQ-9, Billing Diagnosis, Add-On Codes, Warnings |
| Patient Intake Information | The completed intake form — filled in with the open chart's record |

The intake section replaces the note entirely: the tab strip and note toolbar are hidden and the
document title changes. Returning to Client Information restores the note tab that was open before,
along with the tab strip, toolbar, and title. Every other nav item toasts.

**Document header** — "Psychiatric Note" with Effective date, Status, Author dropdown, and a
**Sign** button. In either sidebar form section the title changes to that section's name and this
toolbar is hidden — those forms carry their own action rows.

**Main tabs**

| Tab | State |
| --- | --- |
| SOAP | Built out — the default tab. Empty S/O/A/P + Treatment Goals textareas. |
| Service | Placeholder |
| Note | Built out — has its own sub-tabs (below) |
| PHQ-9 | Built out — fully interactive, scored |
| Billing Diagnosis | Placeholder |
| Add-On Codes | Placeholder |
| Warnings | Placeholder |

**Patient Intake Information** — a sidebar section, not a tab: it shows only when you click
**Patient Intake Information** in the left nav, and the note's tabs are untouched. The intake
record shown the way a completed form looks in the chart: a
patient identity bar, then 14 numbered sections (demographics, contact, emergency contacts,
employment/guarantor, insurance, reason for visit, allergies, medications, medical & surgical
history, family history, behavioral health history & safety, social history, intake screening &
vitals, consents & e-signature).

Every value sits in a real, pre-filled control — text inputs, dropdowns with the recorded option
selected, checked checkboxes, and a textarea for the presenting concern — so the text can be
clicked into, selected, copied, and edited like any other field on the page. Repeating data
(emergency contacts, allergies, medications, family history) uses labelled row groups with an
**Add** link that toasts. The action row above the form has Save and History (toast) and **Print**
(real `window.print()`, with print styles that drop the app chrome).

**Note sub-tabs** — General and Medical Decision Making are built out; Exam, AIMS, Diagnosis, and
Psychotherapy are placeholders. General is pre-filled with a sample depression follow-up
(chief complaint, HPI, medications, MSE, assessment, plan). Medical Decision Making has a problem
row, a Complexity of Problem dropdown, an associated signs/symptoms field, and Data Reviewed
checkboxes.

**Right rail** — patient summary: name, MRN, age, recent diagnosis, vitals grid, and (in
`index.html`) current medications.

## Interactive behavior

Everything real on the page is driven by the inline script at the bottom of the file:

- **Section switching** — `showSection('intake' | 'client')`, driven by the two live sidebar items
  and the `SECTIONS` map. The intake section hides the tab strip and note toolbar and retitles the
  document header; going back calls `showMain(currentMain)` to restore the tab that was open.
- **Tab switching** — main tabs and Note sub-tabs; unbuilt tabs fall back to a shared placeholder
  panel. Switching a main tab scrolls the content area back to the top.
- **PHQ-9 scoring** — click a response per item; the total, `x of 9 answered` counter, and severity
  band update live. Bands: 0–4 minimal, 5–9 mild, 10–14 moderate, 15–19 moderately severe, 20–27
  severe. Answering item 9 (self-harm thoughts) above `Not at all` reveals a safety alert. **Reset**
  clears all responses.
- **Textarea auto-grow** — note fields expand as you type.
- **Toasts** — every non-functional control (nav items, Sign, Add Problem, etc.) shows a brief
  "(demo)" toast instead of doing nothing.

## Element IDs for automation

Stable IDs are on the fields a demo snippet is most likely to target:

- **Patient Intake Information** — the identity bar (`intake-name`, `intake-mrn`, `intake-dob`,
  `intake-age`, `intake-sex`, `intake-acct`, `intake-visit`) is plain text; every other ID below is
  a form control, so read and write `.value` (or `.checked`) rather than `.textContent`:
  `intake-preferred`, `intake-pronouns`,
  `intake-marital`, `intake-language`, `intake-address`, `intake-phone`, `intake-email`,
  `intake-occupation`, `intake-payer`, `intake-memberid`, `intake-referral`, `intake-pcp`,
  `intake-pharmacy`, `intake-reason`, `intake-si`, `intake-phq2`, `intake-gad7`, `intake-auditc`,
  `intake-bp`, `intake-hr`, `intake-rr`, `intake-temp`, `intake-ht`, `intake-wt`, `intake-bmi`;
  row-group and checkbox-group wrappers `intake-emergency`, `intake-allergies`, `intake-meds`,
  `intake-family`, `intake-conditions`, `intake-consents`
- **SOAP tab** — `soap-hpi`, `soap-ros`, `soap-pe`, `soap-ap`, `soap-goals`
- **Note → General** — `cc`, `hpi`, `meds`, `mse`, `assess`, `plan`
- **Note → Medical Decision Making** — `complexity`, `assoc`
- **PHQ-9** — response buttons follow `phq9-q{1-9}-{0-3}` (e.g. `phq9-q4-2`); readouts are
  `phqTotal`, `phqSev`, `phqAnswered`, `phqAlert`, and `phqReset`
- **Sidebar / header** — `nav-client`, `nav-intake`, `nav-form`, `docTitle`
- **Tabs** — `tab-soap`, `tab-service`, `tab-note`, `tab-phq9`, `tab-billing`, `tab-addon`,
  `tab-warnings`; sub-tabs `subtab-general`, `subtab-exam`, `subtab-mdm`, `subtab-aims`,
  `subtab-diagnosis`, `subtab-psychotherapy`
- **Right rail** — `rail-dx`, `rail-age`, `rail-bp`, `rail-hr`, `rail-rr`, `rail-temp`, `rail-wt`,
  `rail-bmi`, `rail-med`

## The form filling page (`form_filling.html`)

A deliberately different product: **Northstar Charting**, a blank new-patient encounter form. It
shares no styling, markup or script with the EMR page — indigo on off-white instead of clinical
blue, rounded cards instead of dense panels, a sticky step rail instead of tabs, pill buttons,
chip toggles and a segmented control instead of checkboxes and radios. Reached from the Patient
Form Filling item at the bottom of the EMR's sidebar; "Back to chart" in its header returns to
`index.html`.

**Layout** — a sticky top bar (brand, breadcrumb, unsaved-draft indicator, back link, submit), a
left step rail listing the eight sections, the form itself as one card per section, and a fixed
bottom dock with progress and actions.

**Sections** — 1 Patient (identity, contact, coverage, emergency contact) · 2 Encounter · 3 Vitals ·
4 History (complaint, HPI, allergy rows, medication rows) · 5 Review of systems · 6 Exam &
assessment (with diagnosis rows) · 7 Plan (prescriptions, orders, follow-up) · 8 Billing &
signature. Everything starts empty; 12 fields are required and marked with a red asterisk.

**Behavior**

- **Progress** — the meter and dock show the percentage of required fields complete, the count of
  filled fields overall, and how many required fields are left; all live.
- **Step rail** — click to jump to a section; an `IntersectionObserver` highlights the section you
  are reading, and any section with data entered gets a green marker.
- **BMI** — calculated from height and weight as you type.
- **Submit** — validates the required fields. Missing ones get a red banner naming each one plus
  red highlights on the fields themselves, cleared as you fill them; a complete form shows a green
  confirmation. Nothing is actually saved.
- **Clear form**, **Print**, and **Save draft** behave as labelled (Save draft toasts).

**Element IDs for automation** — the same `form-*` naming as before, all controls empty on load.

- **Patient** (entered per form, never inherited from a chart) — `form-last-name`,
  `form-first-name`, `form-middle-name`, `form-preferred-name`, `form-dob`, `form-age`,
  `form-sex` (segmented radio group), `form-gender`, `form-pronouns`, `form-marital`,
  `form-language`, `form-mrn`, `form-account`, `form-need-interpreter`, `form-phone`,
  `form-email`, `form-address`, `form-city`, `form-state`, `form-zip`, `form-payer`,
  `form-member-id`, `form-emergency-name`, `form-emergency-phone`
- **Encounter** — `form-dos`, `form-time-in`, `form-time-out`, `form-visit-type`, `form-location`,
  `form-provider`, `form-supervising`, `form-referral`, `form-interpreter`, `form-reason`
- **Vitals** — `form-bp-sys`, `form-bp-dia`, `form-bp-pos`, `form-hr`, `form-rr`, `form-spo2`,
  `form-temp`, `form-temp-route`, `form-pain`, `form-ht-ft`, `form-ht-in`, `form-wt`,
  `form-bmi` (read-only, auto-calculated), `form-vitals-by`
- **History** — `form-cc`, `form-onset`, `form-duration`, `form-severity`, `form-context`,
  `form-hpi`, `form-pmh`, `form-nkda`, `form-meds-reviewed`
- **Review of systems** — `form-ros-negative`, `form-ros-notes`
- **Exam & assessment** — `form-exam-general`, `form-exam-orientation`, `form-exam-type`,
  `form-exam`, `form-mse`, `form-assessment`
- **Plan** — `form-plan`, `form-referrals`, `form-order-priority`, `form-instructions`,
  `form-followup`, `form-followup-date`, `form-appt-scheduled`, `form-precautions`
- **Billing & signature** — `form-cpt`, `form-modifiers`, `form-units`, `form-time-spent`,
  `form-coding-basis`, `form-pos`, `form-addon-codes`, `form-prior-auth`, `form-attest`,
  `form-signer`, `form-credentials`, `form-sign-date`
- **Groups and chrome** — row groups `form-allergies`, `form-medications`, `form-diagnoses`,
  `form-rx`; chip groups `form-ros`, `form-orders`; readouts `pct`, `bar`, `filled`, `total`,
  `reqLeft`, `reqCount`, `reqTotal`, `dockPct`, `dockReq`; banners `errBanner`, `errText`,
  `okBanner`; button `clearBtn`

## Editing

Each page is a single file laid out in the same order: `:root` CSS variables (colors, borders,
severity palette) → component styles → SVG icon sprite → markup → script. To restyle, change the
variables at the top; to add a tab, add a `.tab1` with a `data-main` value plus a matching
`.main-panel[data-main="…"]`, and register its title in the `showMain` fallback map. To add another
standalone sidebar section, add a `.nav-item` calling `showSection('…')`, a matching
`.main-panel[data-main="…"]`, and an entry in the `SECTIONS` map.

`form_filling.html` shares nothing with the other two — its own palette, type scale, components and
script — so edit it on its own terms. Its layout is a sticky step rail plus one `.card` per section;
to add a section, add a `.card` with an `id`, a matching `.step` button with `data-target="…"`, and
the scroll-spy and progress meter pick it up automatically.

Keep the two files in sync when changing shared structure — they currently differ only in the
`<title>`, the right-rail diagnosis, and the rail's medications-vs-labs section. The Patient Intake
Information section is identical in both files, so edits to it belong in both. `form_filling.html`
is independent of both.

## Disclaimer

Mock interface for demonstration only — synthetic data, not affiliated with any EHR vendor, not a
real EHR or patient, and not for clinical use. PHQ-9 item wording © Pfizer Inc. (public-domain
instrument), shown here illustratively.
