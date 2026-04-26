# Renegade Insurance – M&A Interactive Agency Map

> **A real-time, no-code data pipeline that transforms raw Excel leads into a **live M&A (Mergers & Acquisitions) intelligence map** that shows insurance agency buyers and sellers across the United States — with real-time filtering, matching, and deal-stage tracking.


[![Live Demo](https://img.shields.io/badge/🌐_Live_Demo-Visit_Map-2ea44f?style=for-the-badge)]


**The problem it solves:** The BD team maintained a growing Excel spreadsheet of 100+ agency buyers and sellers. Finding geographic matches, tracking deal stages, and sharing insights across the team was slow and manual.

**The solution:** A fully automated pipeline that reads the spreadsheet, geocodes every location, renders a live interactive map with the buyer/seller presence (with other deal information pop up) and matches sellers with top buyers based on proximity.

---

## Architecture & Data Flow

```
┌─────────────────────┐
│  Microsoft Excel    │  ← BD team updates this spreadsheet
│  (SharePoint/OneDrive) │
└────────┬────────────┘
         │ n8n reads via Microsoft Excel node (OAuth2)
         ▼
┌─────────────────────┐
│   n8n Workflow      │
│  ┌───────────────┐  │
│  │ Webhook Trigger│  │  ← HTML map calls this on page load / refresh
│  └──────┬────────┘  │
│         │           │
│  ┌──────▼────────┐  │
│  │ Field Mapping │  │  ← Normalizes 19 raw columns → clean JSON schema
│  └──────┬────────┘  │
│         │           │
│  ┌──────▼────────┐  │
│  │ OpenCage API  │  │  ← Converts city/state text → lat/lng coordinates
│  └──────┬────────┘  │
│         │           │
│  ┌──────▼────────┐  │
│  │  Merge Node   │  │  ← Combines geocoords + all original fields
│  └──────┬────────┘  │
│         │           │
│  ┌──────▼────────┐  │
│  │ Format Output │  │  ← Structures final JSON payload for the frontend
│  └──────┬────────┘  │
└─────────┼───────────┘
          │ JSON response (CORS-enabled)
          ▼
┌─────────────────────┐
│  Interactive HTML   │  ← Leaflet.js map, hosted on GitHub Pages
│  Map (Leaflet.js)   │  ← No backend, no database, no server costs
└─────────────────────┘
```

---

## Features


- 📍 **Buyers** and **Sellers** plotted with color-coded, shape-coded markers
- **Filter by buyer type:** Highly Motivated Buyer · Franchisee · Platform
- **Filter by Seller Stage:** Pre-LOI · LOI Signed
- **Search bar** to find any buyer/seller by name or location instantly
- **Match tab** — surface compatible buyer↔seller pairs by geography and book criteria


### Rich Data Popups
Each pin opens a detailed card showing:
| Field | Example |
|-------|---------|
| Name & Location | William "Bill" Coates — Cocoa, FL |
| Stage | Pre LOI |
| Book Size | 7–8 Mil |
| Revenue | ~$2M revenue; >$1M profit |
| Asking Price | NA |
| Book Type | 100% Specialty – Marine & Aviation |
| Employees | Owner (age 76) + 2 part-time staff |
| Notes / Limitations | Specialized |

### Live Refresh
- One-click **Refresh** button re-fetches the latest spreadsheet data
- Status bar shows live counts: `Buyers: 96 · Sellers: 12 · LOI Signed: 4 · Pre-LOI: 8`
- No deployment needed when the Excel sheet changes

---

## 🔧 Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Data Source** | Microsoft Excel (SharePoint) | Source of truth — BD team edits this |
| **Automation** | [n8n](https://n8n.io/) | No-code workflow orchestration |
| **Geocoding** | [OpenCage Geocoding API](https://opencagedata.com/) | City/State → lat/lng conversion |
| **Frontend** | [Leaflet.js](https://leafletjs.com/) | Interactive map rendering |
| **Hosting** | GitHub Pages | Free, zero-maintenance hosting |
| **Transport** | Webhook (REST/JSON) | Real-time data bridge between n8n and HTML |

---

## ⚙️ n8n Workflow Deep Dive


The automation has **6 nodes** running in sequence:

### Node 1 — Webhook Trigger
Listens for `GET` requests from the HTML frontend. Kicks off the entire pipeline on demand.

### Node 2 — Microsoft Excel Reader
Reads rows `A1:S200` from `Final lead source list.xlsx` on SharePoint using OAuth2 authentication. Handles 19 columns across buyers and sellers.

### Node 3 — Field Normalizer (Code Node)
```javascript
// Maps messy Excel column names → clean camelCase JSON
// Handles: Name, Target region, Type, Ownership,
//          Book Size, Revenue, Asking Price, Book Type,
//          Office/Employee details, Limitations, Buyer Type...
// Also: cleans location strings, appends ", USA" for geocoding accuracy
// Filters out rows missing name, location, or type
```

### Node 4 — OpenCage Geocoding
Sends each `location` string to the OpenCage REST API → gets back `lat` / `lng` coordinates. Runs per-item (one API call per lead).

### Node 5 — Merge Node
Combines the **geocoordinates stream** (lat/lng) with the **full data stream** (all fields) by position — reuniting the location data with the complete record.

### Node 6 — Respond to Webhook
Returns a CORS-enabled JSON array of all processed records to the HTML frontend.

```json
[
  {
    "name": "William \"Bill\" Coates",
    "location": "Cocoa, FL, USA",
    "lat": 28.3861,
    "lng": -80.7425,
    "type": "Seller",
    "stage": "Pre LOI",
    "book_size": "7-8 Mil",
    "revenue": "~$2 Mil revenue; >$1M profit",
    "asking_price": "NA",
    "book_type": "100% Specialty – Marine & Aviation",
    "office_or_employee_details": "Owner (age 76) + 2 part-time staff...",
    "limitations": "Specialized"
  }
]
```

---

## 🚀 Setup & Replication

### Prerequisites
- n8n instance (cloud or self-hosted)
- Microsoft 365 account with Excel file on SharePoint/OneDrive
- [OpenCage API key](https://opencagedata.com/api) (free tier: 2,500 req/day)
- GitHub account (for Pages hosting)

### Steps

**1. Import the n8n Workflow**
```
Import `n8n-mapping_workflow.json` into your n8n instance
```

**2. Connect Credentials**
- Add your **Microsoft Excel OAuth2** credentials in n8n
- Replace the OpenCage API key in the HTTP Request node

**3. Update the Webhook URL**
Copy your n8n webhook URL and update the `fetch()` call in the HTML file:
```javascript
const res = await fetch('https://your-n8n-instance.com/webhook/YOUR-ID');
```

**4. Deploy HTML to GitHub Pages**
```bash
git init
git add M_A_map.html
git commit -m "Initial map deployment"
git branch -M main
git remote add origin https://github.com/yourusername/your-repo.git
git push -u origin main
# Enable GitHub Pages in repo Settings → Pages → Deploy from main branch
```

**5. Refresh and Done ✅**
Open your GitHub Pages URL — the map auto-fetches live data.

---

## 📊 Business Impact

- **Reduced lookup time** from 20+ minutes (manual Excel search) to under 5 seconds
- **Enabled geographic matching** — instantly see which sellers are in a buyer's target region
- **Zero infrastructure cost** — GitHub Pages (free) + n8n + existing Microsoft 365 subscription
- **Non-technical users** can update the map by just editing the Excel file
- **Scales to 200 records** in the current config; adjustable to any range

---

## 🗂️ Repository Structure

```
📁 your-repo/
├── 📄 M_A_map.html           # Complete frontend — Leaflet map + UI
├── 📄 n8n-mapping_workflow.json  # Importable n8n automation workflow
└── 📄 README.md              # This file
```

---

## 📸 Screenshots

| Map View | Seller Detail Popup |
|----------|-------------------|
| ![Map Overview](map-preview-1.png) | Popup shows Stage, Book Size, Revenue, Asking Price, Employees |

---

## 🤝 About This Project

Built for the **Business Development team at Renegade Insurance** to support M&A strategy across the US insurance agency market. The project connects a non-technical sales workflow (Excel) to a visual intelligence layer — demonstrating how automation and lightweight frontend tooling can replace expensive BI software.

---

## Author

**Rakshana Magesh**

---
