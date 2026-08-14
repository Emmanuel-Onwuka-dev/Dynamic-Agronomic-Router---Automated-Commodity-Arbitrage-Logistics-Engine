
# Dynamic Agronomic Router - Automated Commodity Arbitrage & Logistics Engine

An autonomous n8n system that scans agricultural commodity markets every 8 hours, identifies profitable arbitrage routes after real logistics costs, and pushes approved shipments straight to a Slack alert channel and a Notion ledger, with no manual price-checking or spreadsheet work required.

---

## The Problem

### Core Business Challenges

- **Rapid Price Volatility:** Agricultural commodity prices swing sharply due to seasonal shocks, regional insecurity, and localized supply-demand imbalances.
- **Manual Decision-Making:** Relying on phone calls and manual spreadsheets causes severe delays, leading to missed arbitrage opportunities and food spoilage.
- **Fluctuating & Opaque Logistics Costs:** Commercial freight pricing is rarely transparent, making it difficult to calculate true net margins quickly.
- **Fragmented Market Information & Unit Inconsistencies:** Disparate market sources publish data using inconsistent schemas, key names, and varying local volume units (such as derica measurements instead of standard kilograms or tons).

### Impact on Food Security in Nigeria

Agricultural produce often rots at farm-gate markets in northern producing hubs (e.g., Kano) while southern urban centers (e.g., Lagos) suffer from artificial scarcity and high food inflation. By delivering automated market intelligence every 8 hours, this workflow bridges regional supply gaps, accelerates distribution, and ensures commodities flow to high-demand areas efficiently.

---

## Screenshots

**Main workflow: Dynamic Agronomic Router**

<img width="1360" height="557" alt="REAL TIME DATA WORKFLOW" src="https://github.com/user-attachments/assets/2607f33e-e98b-41e2-857a-512305f3e3a6" />


**Error handler sub-workflow**

<img width="1360" height="644" alt="ERROR HANDLER WORKFLOW" src="https://github.com/user-attachments/assets/1e16889a-a1b7-4bc4-9e56-c74f7128050f" />


**Slack output: Profitable route alert**

<img width="1391" height="828" alt="Logistics alert" src="https://github.com/user-attachments/assets/66c88457-1e67-43a6-aad4-98737fe09c2f" />


**Slack output: No profitable routes found**

<img width="1370" height="798" alt="Admin alert-no profit" src="https://github.com/user-attachments/assets/f8882fc3-b410-4d5c-8fb6-4907f72db249" />


**Slack output: System error alert**

<img width="822" height="263" alt="admin alert-error" src="https://github.com/user-attachments/assets/5bb73e04-07b0-4dfa-a65f-accf47678477" />


**Notion output: Master Ledger record**

<img width="1902" height="910" alt="Master Ledger" src="https://github.com/user-attachments/assets/7f6abf61-7f7a-474d-9a62-47b1911dbb5c" />


---

## What it does

The workflow runs on a schedule (every 8 hours) and moves through five stages:

1. **Data Collection** - pulls live commodity prices from multiple market data sources and APIs, plus a separate logistics/freight data feed, in parallel.
2. **Data Processing** - merges the market data from every source, cleans and standardizes inconsistent schemas and field names, normalizes local units (e.g. derica to kilograms), and prepares a clean commodity list and a separate freight/logistics list for analysis.
3. **Decision Engine** - calculates the arbitrage spread (sell price minus buy price) for each route, deducts real logistics costs to get true net profit and margin, and routes each opportunity as profitable or non-profitable against a minimum margin threshold (15%).
4. **AI Automation** (profitable routes only) - uses an AI model to generate a shipment manifest, summarize the route details, draft the Slack alert copy, and prepare a logistics report.
5. **Execution & Reporting** - loops through approved routes, records each one to a Notion Master Ledger (full financial breakdown, confidence ratings, and market summary), and notifies the logistics and admin teams on Slack. Non-profitable scans still notify admins with a scan summary, so the absence of opportunities is as visible as the presence of one.

A separate **Error Handler** sub-workflow catches any failure in the main router (e.g. an expired credential or failed node), extracts the error details, and immediately alerts an admin on Slack with the workflow name, failed node, error message, and timestamp.

---

## Workflow structure

\```
Dynamic Agronomic Router (main, runs every 8 hours)
├─ DATA COLLECTION
│  ├─ Price feed source 1 (API)
│  ├─ Price feed source 2 (API)
│  ├─ Price feed source 3 (API)
│  ├─ Price feed source 4 (API)
│  └─ Freight/logistics data source
├─ Merge (all price sources)
├─ Sanitize data (clean & standardize schema/units)
├─ DATA PROCESSING
│  ├─ Append Commodity List
│  └─ Append Route/Logistics data
├─ Merge (commodities + logistics)
├─ DECISION ENGINE
│  ├─ Arbitrage Calculator (spread, logistics cost, net profit, margin)
│  └─ Router: Profitable vs Non-Profitable
├─ Non-profitable → Notify Admin (Slack: scan summary, no action needed)
└─ Profitable →
   ├─ AI AUTOMATION (OpenAI Chat Model)
   │  └─ Generate manifest, route summary, Slack copy, logistics report
   └─ EXECUTION & REPORTING
      ├─ Loop Over Items (each approved route)
      ├─ Update Master Ledger (Notion)
      └─ Notify Logistics (Slack: #logistics-alerts)

Dynamic Agronomic Router - Error Handler (sub-workflow)
├─ Catch Workflow Errors
├─ Extract Error Details
└─ Alert Admin For Errors (Slack: #admin-alerts)
\```

---

## Setup

1. Import the main `dynamic-agronomic-router.json` and `dynamic-agronomic-router-error-handler.json` workflows into your n8n instance.
2. Connect your commodity price API credentials for each data source, plus your logistics/freight data source.
3. Connect Google Sheets (or your chosen data store) for the commodity and route lists.
4. Connect your OpenAI (or equivalent) credentials for the AI Automation stage.
5. Connect Notion (Master Ledger database) and Slack (#logistics-alerts and #admin-alerts channels).
6. Set the main workflow's error workflow to the Error Handler sub-workflow in workflow settings, so failures route there automatically.
7. Adjust the schedule trigger (default: every 8 hours) and the minimum profit margin threshold (default: 15%) to fit your markets.

## Notes / gotchas

- Unit normalization matters more than it looks. Local volume units (e.g. derica) must be converted to a standard unit (kg/tons) before any price comparison, or margins will be silently wrong.
- The 15% minimum margin threshold is intentional headroom above the true net profit, not the raw spread, since logistics costs can vary significantly by route and season.
- Confidence ratings (e.g. "Low" buy confidence in the Master Ledger) matter as much as the profit numbers. A profitable route with low source confidence should be verified on the ground before committing, the ledger entry says as much explicitly.
- Non-profitable scan cycles still post to Slack. This is deliberate, silence could otherwise be mistaken for the workflow not running at all.

## License

MIT - free to use and adapt for your own commodity or logistics arbitrage tracking setup.
