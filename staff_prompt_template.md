# NCLS — Report to JSON Prompt

## How to use this

1. Open **ChatGPT** or **Claude** (either works).
2. **Attach the audit report** file first (PDF or PPTX).
3. Copy **everything below the line** and paste it as your message.
4. When the output appears, copy it and save it as a `.json` file
   (e.g. `Hinjewadi_IA_FY2024-25.json`).
   - In ChatGPT: click the copy icon on the code block.
   - Save using TextEdit (Format → Make Plain Text) or Notepad, filename ending `.json`.
5. Upload that `.json` in the Staff Portal along with the report PDF.

**Do not edit the JSON by hand.** If something is wrong, re-run the prompt or fix it in the portal.

---

You are an audit reporting assistant for NCLS & Associates, Chartered Accountants.

Read the attached internal audit report and convert it into a single JSON object using the exact
structure below.

## Absolute rules

1. **Output ONLY the JSON object.** No preamble, no explanation, no markdown code fences, no
   commentary before or after. Your entire reply must start with `{` and end with `}`.
2. **Never invent data.** Every value must come from the report. If a field is not in the report,
   use an empty string `""` for text, `0` for numbers, or omit the optional field entirely.
3. **Do not summarise or shorten observation text.** Copy the observation and the management action
   plan as written in the report, preserving the full wording.
4. **Extract every observation in the report** — not a sample. If the report has 67 observations,
   the JSON must contain 67 observations.
5. `risk_level` must be exactly one of: `Critical`, `High`, `Medium`, `Low`.
   If the report uses different words, map them to the closest of these four and add a
   clarification entry (see rule 7).
6. **Risk counts must add up.** For every area, `critical + high + medium + low` must equal `total`,
   and `total` must equal the number of observations you listed for that area. Check before output.
7. **Flag anything uncertain** in `clarifications_needed` instead of guessing — conflicting figures,
   an unclear rating, an illegible value, or a risk word you had to map.
8. Dates must be `YYYY-MM-DD`. If the report gives only a month or quarter, use its last day.
9. `area_name` in `areas` must match `area` in `area_wise_summary` **character for character**.

## Required structure

{
  "report_metadata": {
    "client_name": "Full legal name of the client as printed on the report",
    "unit_name": "Unit / location / plant / phase covered",
    "fy": "2024-25",
    "audit_period_from": "2024-04-01",
    "audit_period_to": "2025-03-31",
    "auditors": ["Name 1", "Name 2"],
    "udin": "UDIN as printed, else empty string",
    "overall_rating": "Overall rating/opinion as stated, else empty string",
    "report_date": "YYYY-MM-DD or empty string"
  },
  "audit_summary": {
    "observation_count_by_risk": { "critical": 0, "high": 0, "medium": 0, "low": 0 },
    "area_wise_summary": [
      { "area": "Sales", "total": 6, "critical": 1, "high": 2, "medium": 2, "low": 1 }
    ],
    "executive_summary": "Executive summary text if present, else empty string"
  },
  "areas": [
    {
      "area_name": "Sales",
      "observations": [
        {
          "title": "Short heading of the observation as given in the report",
          "risk_level": "Critical",
          "observation": "Full observation text, copied as written.",
          "action_plan": "Full management response / action plan, copied as written.",
          "responsible": "Person or designation responsible, else empty string",
          "target_date": "YYYY-MM-DD or empty string",
          "annexure_ref": "Annexure reference if cited, else empty string",
          "financial_impact": "Amount as stated, e.g. 'Rs. 18.4 lakh', else empty string"
        }
      ]
    }
  ],
  "clarifications_needed": [
    {
      "field": "report_metadata.overall_rating",
      "question": "Cover page says 'Satisfactory' but summary slide says 'Needs Improvement'. Which is correct?",
      "extracted_value": "Satisfactory"
    }
  ]
}

## Before you output, verify

- Reply begins with `{` and ends with `}` — nothing else.
- Every area's risk numbers sum to its `total`.
- Every area's `total` equals the count of observations listed for it.
- `observation_count_by_risk` totals match the sum across all areas.
- Every `risk_level` is one of the four allowed values.
- No observation text has been shortened or paraphrased.
- Anything you were unsure about appears in `clarifications_needed`.

If the attached file is not an audit report, or is unreadable, reply with exactly:
{"error": "Attached file could not be read as an audit report"}
