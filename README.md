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

Both files are standalone: one HTML file each, with inline CSS, inline JavaScript, and inline SVG
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

**Left sidebar** — Client Dashboard, Client Information (active), Medication Management / Rx,
Client Orders, Client MAR, and SmartLinks.

**Document header** — "Psychiatric Note" with Effective date, Status, Author dropdown, and a
**Sign** button.

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

**Note sub-tabs** — General and Medical Decision Making are built out; Exam, AIMS, Diagnosis, and
Psychotherapy are placeholders. General is pre-filled with a sample depression follow-up
(chief complaint, HPI, medications, MSE, assessment, plan). Medical Decision Making has a problem
row, a Complexity of Problem dropdown, an associated signs/symptoms field, and Data Reviewed
checkboxes.

**Right rail** — patient summary: name, MRN, age, recent diagnosis, vitals grid, and (in
`index.html`) current medications.

## Interactive behavior

Everything real on the page is driven by the inline script at the bottom of the file:

- **Tab switching** — main tabs and Note sub-tabs; unbuilt tabs fall back to a shared placeholder panel.
- **PHQ-9 scoring** — click a response per item; the total, `x of 9 answered` counter, and severity
  band update live. Bands: 0–4 minimal, 5–9 mild, 10–14 moderate, 15–19 moderately severe, 20–27
  severe. Answering item 9 (self-harm thoughts) above `Not at all` reveals a safety alert. **Reset**
  clears all responses.
- **Textarea auto-grow** — note fields expand as you type.
- **Toasts** — every non-functional control (nav items, Sign, Add Problem, etc.) shows a brief
  "(demo)" toast instead of doing nothing.

## Element IDs for automation

Stable IDs are on the fields a demo snippet is most likely to target:

- **SOAP tab** — `soap-hpi`, `soap-ros`, `soap-pe`, `soap-ap`, `soap-goals`
- **Note → General** — `cc`, `hpi`, `meds`, `mse`, `assess`, `plan`
- **Note → Medical Decision Making** — `complexity`, `assoc`
- **PHQ-9** — response buttons follow `phq9-q{1-9}-{0-3}` (e.g. `phq9-q4-2`); readouts are
  `phqTotal`, `phqSev`, `phqAnswered`, `phqAlert`, and `phqReset`
- **Tabs** — `tab-soap`, `tab-service`, `tab-note`, `tab-phq9`, `tab-billing`, `tab-addon`,
  `tab-warnings`; sub-tabs `subtab-general`, `subtab-exam`, `subtab-mdm`, `subtab-aims`,
  `subtab-diagnosis`, `subtab-psychotherapy`
- **Right rail** — `rail-dx`, `rail-age`, `rail-bp`, `rail-hr`, `rail-rr`, `rail-temp`, `rail-wt`,
  `rail-bmi`, `rail-med`

## Editing

Each page is a single file laid out in the same order: `:root` CSS variables (colors, borders,
severity palette) → component styles → SVG icon sprite → markup → script. To restyle, change the
variables at the top; to add a tab, add a `.tab1` with a `data-main` value plus a matching
`.main-panel[data-main="…"]`, and register its title in the `showMain` fallback map.

Keep the two files in sync when changing shared structure — they currently differ only in the
`<title>`, the right-rail diagnosis, and the rail's medications-vs-labs section.

## Disclaimer

Mock interface for demonstration only — synthetic data, not affiliated with any EHR vendor, not a
real EHR or patient, and not for clinical use. PHQ-9 item wording © Pfizer Inc. (public-domain
instrument), shown here illustratively.
