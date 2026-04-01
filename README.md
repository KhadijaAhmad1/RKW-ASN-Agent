# RKW ASN Agent

A client-side browser tool that parses Advanced Shipping Notice (ASN) XML files from ESP/K&N and produces structured Excel reports and shipment summaries — built to support RKW Limited's migration to Microsoft Dynamics 365 F&O.

## What it does

RKW receives ASN files from K&N (Kuehne+Nagel), their freight forwarder, containing inbound shipment data for goods arriving from suppliers in China and Hong Kong. Previously this data was manually re-keyed into purchase order records. This tool automates the reading, validation and reporting of that data.

**Upload an ASN XML file → instantly get:**

- Full shipment summary (vessel, voyage, ETS/ETA, route, carrier)
- Line-by-line PO breakdown with quantity match/mismatch flags
- Container summary showing which POs are loaded in which container
- Downloadable Excel workbook (two sheets: all lines + summary)
- Downloadable plain-text report for sharing with the procurement team

## Live demo

Open `index.html` in any browser. No installation, no server, no login required.

Drag and drop an ASN XML file or click to upload. The tool works entirely offline — no data is sent anywhere.

## Screenshots

![ASN Agent — vessel banner and PO lines table](docs/screenshot.png)

## Business context

RKW Limited is Europe's leading distributor of small domestic appliances and housewares, distributing 3,000+ product lines with £25m of stock across a 750,000 sqft distribution centre in Staffordshire. They supply major UK retailers including Asda, Tesco, Argos, Amazon and B&M.

This tool is part of a broader ESP→D365 integration project, mapping inbound shipment data from RKW's ESP procurement system into Microsoft Dynamics 365 Finance & Operations as part of a company-wide ERP migration.

**Problem it solves:** The warehouse and procurement teams received ASN files from K&N but had no quick way to validate quantities, check vessel details, or produce a summary report without manually opening the XML. Cathy (Procurement) was updating vessel details, dates, and quantities on each PO line by hand when booking confirmations arrived from ESP.

**Impact:** Instant validation of ordered vs advised quantities across all PO lines. Discrepancies are flagged automatically. Saves ~30 minutes of manual cross-referencing per ASN file received.

## Technical details

### Architecture decision — client-side only

This tool runs entirely in the browser with no backend server. This was a deliberate decision:

- ASN files contain commercially sensitive supplier and shipment data that should not be transmitted to external servers
- Zero deployment cost and zero infrastructure to maintain
- Works offline inside RKW's network without any IT setup

### Technologies used

| Technology | Purpose |
|---|---|
| `DOMParser` (browser-native) | Parse XML ASN files into a navigable document tree |
| `SheetJS / xlsx.js` | Generate `.xlsx` Excel files client-side with multiple sheets |
| Vanilla JavaScript (ES6) | Field extraction, data transformation, UI rendering, file download |
| HTML5 File API | Drag-and-drop and file input handling |
| CSS custom properties | Theming and responsive layout |

No frameworks. No build step. No dependencies beyond SheetJS loaded from CDN.

### XML structure handled

```
ASNEnvelope
├── ASNHeaderIdentifier      (sender, receiver, envelope ID, date/time)
└── ASNPurchaseOrderEvent[]  (one per PO line)
    ├── EventIdentifer       (event code, PO number, line, item)
    ├── LineDeliveryOrder    (status date/time, ordered qty, advised qty)
    ├── Transportation       (mode, ports, carrier, loading type)
    ├── VesselInformation    (ETS, ETA, vessel name, voyage no, ATS, ATA)
    ├── VolumeInformation    (CBM, carton count)
    ├── ContainerInformation (weight, volume, container no, size, seal, tracking)
    └── ClearanceInformation (clearance date, entry number)
```

### Discrepancy detection

The tool compares `OrderQty` vs `AdvisedQuantity` on every line. Mismatches are flagged in red with a `DIFF` badge in the table and a `MISMATCH` cell in the Excel output. The summary header shows an overall `⚠ mismatch` warning if any line has a discrepancy.

## Folder structure

```
rkw-asn-agent/
├── index.html          # The entire application
├── sample/
│   └── ASN_sample.xml  # Anonymised sample ASN file for testing
├── docs/
│   └── screenshot.png  # UI screenshot
└── README.md
```

## Planned next steps

- [ ] Power Automate integration — write parsed fields directly to D365 F&O Purchase Order records via the Dynamics 365 connector
- [ ] Multi-file batch processing — upload multiple ASN files and consolidate into one Excel workbook
- [ ] D365 field mapping layer — map ESP XML field names to D365 API field names with District 5 conditional overrides
- [ ] UTC → UK timezone conversion for ETS/ETA dates before D365 write-back

## Part of a larger project

This tool is one component of an ESP→D365 integration being built during RKW's ERP migration. Other components in development:

- **Field mapping layer** — conditional mapping of ESP data to D365 PO event fields including District 5 adjustments
- **Date synchronisation** — ETD/ETA propagation from PO header to PO lines and Sales Order requested receipt date for direct orders
- **Vessel update automation** — replacing Cathy's daily manual vessel/date updates with an automated flow

## Author

Built by [Your Name] as part of RKW Limited's D365 migration project.  
[Your LinkedIn] · [Your GitHub profile]

---

*This repository is part of a portfolio submitted in support of a UK Global Talent visa application (Exceptional Promise — Digital Technology).*
