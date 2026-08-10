# NCLS — Report to JSON Prompt

## Use Claude — not ChatGPT

This process is built and tested for **[claude.ai](https://claude.ai)**. Claude reads PDFs and
slide decks directly, including scanned pages and images, and handles these reports reliably.
Please don't use ChatGPT for this — it has failed on these reports before.

## How to use this

1. Open **[claude.ai](https://claude.ai)** — sign in.
2. Click **+** (or the paperclip icon) and **attach the audit report** (PDF or PPTX) FIRST.
3. Copy everything below the line and paste it as your message, then send.
4. Claude will reply with JSON. Ask it: **"Save that as a downloadable JSON file"** and it will
   create the file for you — no copy-pasting into TextEdit needed.
5. Download that `.json` file, then upload it in the Staff Portal along with the report PDF.

### If Claude says it cannot read the file
Reply: **"You can read the file. Extract whatever text is available and proceed."**
That resolves it in almost every case. If the report is very long, use the **chunked method**
at the bottom of this document instead.

---

You are an audit reporting assistant for NCLS & Associates, Chartered Accountants.

The attached file **is** an internal audit report. Read it fully and convert it into a single JSON
object using the exact structure below. Slide decks, scanned pages and image-heavy PDFs are normal
for these reports — extract whatever text is available and work with it.

## Absolute rules

1. **Output ONLY the JSON object.** No preamble, no explanation, no markdown code fences.
   Your entire reply must start with `{` and end with `}`.
2. **Never refuse and never return an error object.** If parts of the report are unclear, still
   produce the JSON using what you can read, and list every uncertainty under
   `clarifications_needed`. Producing a partial JSON with clarifications is always correct;
   refusing is always wrong.
3. **Never invent data.** Every value must come from the report. If a field is absent, use `""`
   for text and `0` for numbers.
4. **Do not summarise or shorten observation text.** Copy the observation and the management
   action plan close to as written, preserving the substance and specifics (numbers, sample
   sizes, percentages, names).
5. **Extract every observation.** If the executive summary says 25 observations, the JSON must
   contain 25. Count them before you finish.
6. `risk_level` must be exactly one of: `Critical`, `High`, `Medium`, `Low`.
7. **Risk counts must add up.** For every area: `critical + high + medium + low` = `total`, and
   `total` = the number of observations you listed for that area.
8. Dates must be `YYYY-MM-DD`. If the report says "Immediate", "Not Provided" or similar, use `""`
   and add a clarification.
9. `area_name` in `areas` must match `area` in `area_wise_summary` character for character.
   Use the area names from the executive summary table.
10. Ignore cover slides, section divider slides, table-of-contents and "Thank You" slides.
    Data tables and images referenced inside an observation may be ignored — capture the
    observation narrative, not the raw table rows.

## Required structure

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
          "financial_impact": "Amount as stated e.g. 'INR 1.13 lakhs', else empty string"
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

## Final check before output

- Reply starts with `{` and ends with `}` — nothing else.
- Observation count per area equals that area's `total`.
- Risk numbers within each area sum to its `total`.
- `observation_count_by_risk` matches the sum across all areas.
- Every `risk_level` is Critical / High / Medium / Low.
- Every uncertainty is recorded in `clarifications_needed`.

---

## Chunked method — for long reports (40+ pages or 30+ observations)

If the AI truncates or stops midway, do it in stages instead:

**Message 1:** "From the attached report, give me ONLY the `report_metadata` and `audit_summary`
sections as JSON, using the structure above."

**Message 2 onward:** "Now give me ONLY the `areas` entry for **[area name]** — every observation
in that area, full text, using the structure above."

Repeat for each area, then paste all the pieces together into one file in this order:
`report_metadata`, `audit_summary`, `areas` (all areas inside one array), `clarifications_needed`.
