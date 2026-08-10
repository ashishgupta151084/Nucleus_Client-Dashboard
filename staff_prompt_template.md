# NCLS — Report to JSON Prompt

## Use Claude — not ChatGPT

This process is built and tested for **[claude.ai](https://claude.ai)**. Claude reads PDFs and
slide decks directly, including scanned pages and images, and handles these reports reliably.
Please don't use ChatGPT for this — it has failed on these reports before.

## How to use this

1. Open **[claude.ai](https://claude.ai)** — sign in.
2. Click **+** (or the paperclip icon) and **attach the audit report** (PDF or PPTX) FIRST.
3. Copy everything below the line, then **type or paste it directly into the message box** —
   the same box you'd normally type a chat message into — and send it.
   **Important: don't save this text as its own file and attach that file.** Typing/pasting it
   as your message is what makes Claude treat it as your own request. Attaching it as a second
   file alongside the report can confuse things and cause Claude to hesitate or ask
   clarifying questions instead of just producing the JSON.
4. Claude will reply with JSON. Ask it: **"Save that as a downloadable JSON file"** and it will
   create the file for you — no copy-pasting into TextEdit needed.
5. Download that `.json` file, then upload it in the Staff Portal along with the report PDF.

### If Claude asks a clarifying question instead of giving you the JSON
This can happen if the prompt text ended up attached as a file rather than typed into the
message box — reply with: **"Yes please, go ahead and use that prompt on the attached report."**
That's usually enough. If it's still stuck, close the chat and start a fresh one, making sure
to paste the prompt directly into the message box this time.

### If Claude says it cannot read the file
Reply: **"You can read the file. Extract whatever text is available and proceed."**
That resolves it in almost every case. If the report is very long, use the **chunked method**
at the bottom of this document instead.

---

Hi Claude — I work at NCLS & Associates, a Chartered Accountancy firm, and I've attached one of
our internal audit reports (it may be a PDF or a slide deck export, sometimes scanned or
image-heavy). Could you help me convert it into a JSON file? I need this for a dashboard system
our firm uses, so the structure below has to be followed closely.

A few things that would really help me:

- **Please reply with just the JSON object** — no text before or after it, and no markdown code
  fences. I'll be copying your reply straight into a `.json` file, so anything extra means I have
  to go back and edit it out by hand.
- **Please don't spend time visually analysing or describing the photos** in the report — things
  like site photos, stock boards, or screenshots of physical documents. The written observation
  text next to each photo already explains what it shows, so the photo itself isn't needed for
  the JSON. Skipping over these should also make this faster. The one exception is when a table's
  actual figures only exist as an image with no separate text version — in that case a quick read
  of just that table is fine, but everything else (illustrative photos, site pictures, signage,
  stacked boxes, etc.) can be skipped entirely.
- **If parts of the report are hard to read, please don't skip them or stop** — just do your best
  with what you can make out, and note anything you're unsure about in a `clarifications_needed`
  list at the end. A partial JSON with a few open questions is far more useful to me than being
  told the file can't be processed — these reports are always genuine, readable audit reports,
  just sometimes as image-heavy slide decks.
- **Please don't invent or guess at values.** Everything should come from the report. If a field
  genuinely isn't in the report, `""` for text or `0` for numbers is fine.
- **Please don't shorten or summarise the observation text or action plans** — copy them close to
  as written, keeping the specific numbers, sample sizes, percentages and names intact.
- **Please make sure every observation is captured.** If the executive summary says there are 25
  observations, I'd like to see all 25 in the JSON — it's worth double-checking the count before
  you finish.

A few structural things to keep consistent:

- `risk_level` should be exactly one of: `Critical`, `High`, `Medium`, `Low`.
- For every area, `critical + high + medium + low` should add up to that area's `total`, and
  `total` should match the number of observations listed for that area.
- Dates in `YYYY-MM-DD` where possible. If the report just says "Immediate" or "Not Provided",
  `""` is fine — a note in `clarifications_needed` is helpful there.
- `area_name` under `areas` should match `area` under `area_wise_summary` exactly, using the
  area names from the executive summary table.
- Cover slides, section dividers, tables of contents and "Thank You" slides can be skipped. For
  data tables or images referenced inside an observation, the observation's own narrative text
  is what matters — the raw table rows can be left out.

## Required structure

Here's the JSON shape I'm looking for:
{
  "report_metadata": {
    "client_name": "Full legal name of the client",
    "unit_name": "Unit / location / plant covered",
    "fy": "2025-26",
    "audit_period_from": "2025-10-01",
    "audit_period_to": "2025-12-31",
    "auditors": ["Name 1", "Name 2"],
    "udin": "UDIN as printed, else empty string",
    "overall_rating": "Overall rating/conclusion as stated, else empty string",
    "report_date": "YYYY-MM-DD or empty string"
  },
  "audit_summary": {
    "observation_count_by_risk": { "critical": 0, "high": 0, "medium": 0, "low": 0 },
    "area_wise_summary": [
      { "area": "Inventory Management", "total": 13, "critical": 2, "high": 3, "medium": 5, "low": 3 }
    ],
    "executive_summary": "Conclusion / executive summary text, else empty string"
  },
  "limitations": [
    {
      "area": "Sales",
      "items": ["Part Wise Daily Dispatch Report", "Set Wise Sales Rate", "RMC Report"]
    }
  ],
  "areas": [
    {
      "area_name": "Inventory Management",
      "observations": [
        {
          "title": "Observation heading as given in the report",
          "risk_level": "Critical",
          "observation": "Full observation text.",
          "action_plan": "Management action plan as written. Include any Auditor's Remarks.",
          "responsible": "Responsible person as named, else empty string",
          "target_date": "YYYY-MM-DD or empty string",
          "annexure_ref": "Table/Image/Annexure reference if cited, else empty string",
          "financial_impact": "Amount as stated e.g. 'INR 1.13 lakhs', else empty string",
          "is_repeated": false,
          "pending_since_days": 0,
          "root_cause": "Process Design Deficiency"
        }
      ]
    }
  ],
  "clarifications_needed": [
    {
      "field": "report_metadata.overall_rating",
      "question": "State the conflict or uncertainty as a question for the auditor.",
      "extracted_value": "The value you used"
    }
  ]
}

## Limitations section — how to fill it

Reports usually have a "Limitation of Scope" slide with a table of Area → data not made
available. Capture this exactly:
- One entry per area listed in that table.
- `items` is the list of documents/data not provided, as stated.
- If the report has no such section, use an empty array `[]` — do not invent limitations.

## Root cause — how to fill `root_cause`

Each observation's detail panel in the report states a root cause (often shown as a label
like "Root Cause: Process Design Deficiency"). Copy it as stated. It is almost always one of:
- "Process Design Deficiency" — the process/policy itself was never properly designed
- "Operating Ineffectiveness" — the process exists but wasn't followed correctly
- "People Ineffectiveness" — a skills/awareness/staffing gap

If the report uses different wording, copy it as written rather than forcing it into one of
these three. If no root cause is stated for an observation, use an empty string `""`.

## Repeated observations — how to fill `is_repeated` and `pending_since_days`

Reports mark carried-over findings in one or both of these ways — check for either:
1. The observation heading itself is labelled, e.g. "Observations – Sales Process -
   **Repeated Observation**".
2. A separate "Previous Audit Non-Compliance" section lists the same finding with a
   "Pending Since (days)" figure.

A couple of specifics that help:
- Set `is_repeated: true` only when the report itself marks the observation as repeated —
  never infer this from wording alone.
- If a pending-days figure is given for that observation (in the Previous Audit
  Non-Compliance table, or elsewhere), copy the number into `pending_since_days`. Otherwise
  use `0`.
- If unsure whether an observation is the same as a prior one, leave `is_repeated: false`
  and add a note to `clarifications_needed` instead of guessing.

## One last check before you send it back

If you could just glance over these before replying, that'd be great:

- The reply starts with `{` and ends with `}` — nothing else around it.
- The observation count per area matches that area's `total`.
- The risk numbers within each area add up to that area's `total`.
- `observation_count_by_risk` matches the sum across all areas.
- Every `risk_level` is Critical, High, Medium, or Low.
- Every area in the Limitation of Scope table has a matching entry in `limitations`.
- Every observation explicitly labelled "Repeated Observation" has `is_repeated: true`.
- Every observation has a `root_cause` where the report states one.
- Anything you're unsure about is noted in `clarifications_needed`.

Thanks so much for your help with this!

---

## Chunked method — for long reports (40+ pages or 30+ observations)

If the AI truncates or stops midway, do it in stages instead:

**Message 1:** "From the attached report, give me ONLY the `report_metadata` and `audit_summary`
sections as JSON, using the structure above."

**Message 2 onward:** "Now give me ONLY the `areas` entry for **[area name]** — every observation
in that area, full text, using the structure above."

Repeat for each area, then paste all the pieces together into one file in this order:
`report_metadata`, `audit_summary`, `areas` (all areas inside one array), `clarifications_needed`.
