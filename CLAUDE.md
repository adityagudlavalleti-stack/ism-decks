# ISM VIP Deck Questionnaire — Project Context

## What This Is
A password-gated web tool for collecting event-specific briefs across ISM's VIP deck portfolio (~30 events). Built and maintained by Aditya Gudlavalleti. Aditya is both the operator AND the deck builder — he fills briefs, submits them, and then uses Claude to build the actual decks.

## Live URLs
- **Tool:** https://adityagudlavalleti-stack.github.io/ism-decks/
- **Password:** `ism2026`
- **GitHub repo:** https://github.com/adityagudlavalleti-stack/ism-decks
- **Local file:** `C:\Users\adity\OneDrive\Desktop\ism-decks\docs\index.html`

## Google Sheet
- **Sheet ID:** `1pQj5DIDTcRqIQJyBHe-M9RnTk936AXsBMn_D3fJUiBQ`
- **Sheet URL:** https://docs.google.com/spreadsheets/d/1pQj5DIDTcRqIQJyBHe-M9RnTk936AXsBMn_D3fJUiBQ/edit
- **Tab 1 — Live Data:** The sync database. Every event auto-saves here every 5 seconds. 7 columns (ID, Name, Status, Created, Last Edit, Data JSON, Notes JSON). Don't edit manually.
- **Tab 2 — Submitted Briefs:** Human-readable format. One row per submitted event with all 57 fields expanded. This is what Aditya reads to build decks.

## Apps Script
- **Final URL:** `https://script.google.com/macros/s/AKfycbztbivNpsxTEeUoeF2XKGpVcjs8EP8GHkvNB8P-CUEUWdPvbAWVj0eUw1wHNM3xNXVl8A/exec`
- **GET:** Returns all events from Live Data tab as JSON
- **GET ?tab=submitted:** Returns all rows from Submitted Briefs tab
- **POST action=save:** Upserts event in Live Data
- **POST action=delete:** Removes event from Live Data
- **POST action=submit:** Writes clean readable row to Submitted Briefs + updates Live Data status

## How to Pull a Brief Into Claude
When Aditya says "pull the [event name] brief", run:
```powershell
$url = "https://script.google.com/macros/s/AKfycbztbivNpsxTEeUoeF2XKGpVcjs8EP8GHkvNB8P-CUEUWdPvbAWVj0eUw1wHNM3xNXVl8A/exec?tab=submitted"
$r = Invoke-WebRequest -Uri $url -Method GET -MaximumRedirection 10 -UseBasicParsing
$data = $r.Content | ConvertFrom-Json
# Fuzzy match — finds closest name regardless of exact wording
$keyword = "KEYWORD" # replace with word from event name e.g. "Daytona"
$brief = $data.rows | Where-Object { $_.'Event Name' -like "*$keyword*" } | Select-Object -First 1
if(!$brief){ Write-Host "No match found. Available events:"; $data.rows | ForEach-Object { Write-Host " - $($_.'Event Name')" } }
else { $brief.PSObject.Properties | ForEach-Object { if($_.Value) { Write-Host "$($_.Name): $($_.Value)" } } }
```
Then format the output as a structured deck brief and start building with Aditya.

## Architecture
```
Form (GitHub Pages)
  → Auto-saves to Sheet Tab 1 (Live Data) every 5 seconds via Apps Script POST
  → On Submit → writes clean row to Sheet Tab 2 (Submitted Briefs)
  → Aditya says "pull [event] brief" → Claude fetches Tab 2 via PowerShell → builds deck
```

## Form Structure — 11 Sections
1. Event Overview (name, location, date, sport, program, budget, relationship, decision date, priority, assigned to, prev deck ref)
2. Target Audience (decision maker, attendees, guest count, experience level, deck for, sensitivities)
3. Business Goals (pill checkboxes + primary goal, competitors, belief, CTA)
4. Core Message (story, key points, phrases, avoid)
5. VIP Experience (wow factor, elements pills, journey, moments, ISM managed)
6. Event-Specific Access (sport tabs + specific access textarea)
7. Visual Direction (style pills, colors, feel, refs, brand assets, generated imagery, logos avoid)
8. Content & Proof Points (ISM points pills, testimonials, ROI)
9. Slide Structure (slide pills, deck length, format)
10. Logistics & Contact (contact name, company, email, client email, phone, website, deadline, final contact)
11. Final Notes + Internal Team Notes thread

## ISM Defaults (pre-fill on every new event)
- **Goals:** Accelerate Deal Close, Client Retention, Customer Hospitality, Executive Networking
- **ISM Points:** All 8 (29 years, Premium hospitality, Executive entertainment, Relationship-driven access, Turnkey management, Private aviation, Multi-city programs, Custom corporate)
- **Slides:** Cover → Who ISM Is → Why This Event → Guest Journey → Access Moments → Business Outcomes → ROI / Relationship Value → Next Steps
- **Style:** Left open per event

## Status Flow
Not Started → In Progress → Submitted → Deck Delivered

## Dashboard Features
- Filter by status, search by name, sort by deadline/priority/status/last edited
- Deadline urgency badge (red) when ≤ 7 days out
- Priority badges (Urgent/High/Normal/Low)
- Duplicate event button
- Export all as CSV
- Last edited timestamp

## Deployment
Push changes from `C:\Users\adity\OneDrive\Desktop\ism-decks\` — GitHub Pages auto-deploys in ~30s.
```
git add docs/index.html
git commit -m "..."
git push
```

## ISM Company Info
- **Website:** https://ism-usa.com/
- **Logo (white PNG):** https://ism-usa.com/wp-content/uploads/2026/03/ism2026logo-wht.png
- Aditya is working with ISM on a temporary/consulting basis
- Key contacts: Christian Griffith (surftrip@gmail.com), Jason Parker (jparker@ism-usa.com), Tulaib Faizy (tulaib@theyourid.com), ISM Marketing (marketing@ism-usa.com)
