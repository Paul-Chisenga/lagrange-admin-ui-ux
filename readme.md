# Admin Dashboard UI/UX Design Guide & Specifications

> **Target Audience:** UI/UX Designers, Product Designers, and Frontend Developers.  
> **Platform Scope:** Web Admin Dashboard (Desktop Optimized, Responsive Tablet Support).

---

# PART 1: SYSTEM OVERVIEW & CORE UX FLOWS

## 1. Core Platform Lifecycle

The Admin Dashboard is designed around the complete operational lifecycle—from tree planter onboarding and identity verification to tree planting, continuous care monitoring, carbon calculation, and marketplace credit transactions.

``` mermaid
flowchart LR
    A["1. Planter Registration & Profile"] --> B["2. Identity & Location Verification"]
    B -->|Admin Verified| C["3. Join Project"]
    C --> D["4. Tree Assignment"]
    D --> E["5. Tree Planting"]
    E --> F["6. Tree Care Logging<br/>(Activities & Growth)"]
    F --> G["7. Carbon Calculation"]
    G --> H["8. Carbon Credit Issuance"]
    H --> I["9. Marketplace Listing"]
    I --> J["10. Buyer Purchase & Retirement"]
```

---

## 2. Core UX Flows

### 2.1 Flow 1: Tree Planter Verification

Tree planters register via mobile and submit KYC documents and location details. Administrators review and approve/reject each section independently.

``` mermaid
sequenceDiagram
    autonumber
    participant TP as Tree Planter (Mobile)
    participant APP as Mobile API / System
    participant AD as Admin Dashboard (UI)
    participant ADM as Administrator

    TP->>APP: Register and complete profile
    TP->>APP: Submit GPS location and ID photos
    APP->>AD: Create verification queue item
    AD->>ADM: Surface in Pending Verification queue
    ADM->>AD: Open multi-section review screen
    ADM->>AD: Verify Personal Information
    ADM->>AD: Verify Administrative Location and Pin
    ADM->>AD: Verify ID and Passport Photos
    alt All Sections Verified
        ADM->>AD: Click Approve and Mark Verified
        AD->>APP: Set status to VERIFIED
        APP->>TP: Notify planter: Ready to join projects
    else Rejected or Incomplete
        ADM->>AD: Click Reject Section (with reason note)
        AD->>APP: Return rejection feedback
        APP->>TP: Prompt planter to resubmit
    end
```

### 2.2 Flow 2: Profile Update Request

Tree planters cannot directly modify critical personal or location details after verification; they must submit a change request with a justification reason.

``` mermaid
sequenceDiagram
    autonumber
    participant TP as Tree Planter (Mobile)
    participant APP as Mobile API
    participant AD as Admin Dashboard (UI)
    participant ADM as Administrator

    TP->>APP: Request profile change (with reason)
    APP->>AD: Push to Profile Update Requests
    AD->>ADM: Display Side-by-Side Diff UI
    ADM->>AD: Inspect current vs requested values
    alt Approved
        ADM->>AD: Confirm approval modal
        AD->>APP: Overwrite profile data and log audit
        APP->>TP: Notify planter of update
    else Rejected
        ADM->>AD: Reject with mandatory admin explanation
        AD->>APP: Record rejection and send feedback
        APP->>TP: Notify planter with reason
    end
```

### 2.3 Flow 3: Project, Tree & Carbon Lifecycle

``` mermaid
flowchart TD
    A["Admin Creates Project & Defines Geofence"] --> B["Admin Generates Tree IDs in Bulk"]
    B --> C["Unique Tree IDs Created in Available Pool"]
    C --> D["Admin Assigns Trees to Verified Planters"]
    D --> E["Planter Receives Tree & Plants Seedling"]
    E --> F["Planter Submits Tree Care Logs<br/>(Care Activities + Growth Measurements + Photos)"]
    F --> G["System Calculates Carbon Sequestration"]
    G --> H["Carbon Credits Minted / Issued"]
    H --> I["Admin Lists Credits on Marketplace"]
    I --> J["Carbon Buyer Purchases / Retires Credits"]
```

---

## 3. Important Entity Relationships

The data model relationships directly guide the UI architecture, navigational breadcrumbs, and detail screen tabs.

> **Note on Care & Growth:** Tree Care Logs and Tree Growth Records are unified into a single entity: `CARE_LOG`. Each log captures both care actions (e.g., watering, weeding, pruning, pest control) and measurable growth indicators (height, diameter, canopy health, photos).

``` mermaid
erDiagram
    ADMIN ||--o{ PROJECT : "creates"
    ADMIN ||--o{ TREE_PLANTER : "verifies"
    ADMIN ||--o{ PROFILE_UPDATE_REQUEST : "reviews"
    ADMIN ||--o{ AUDIT_LOG : "generates"
    
    PROJECT ||--o{ TREE : "contains"
    PROJECT ||--o{ TREE_PLANTER : "includes"
    
    TREE_PLANTER ||--o{ TREE : "receives"
    TREE_PLANTER ||--o{ PROFILE_UPDATE_REQUEST : "submits"
    
    TREE ||--o| TREE_PLANT : "becomes"
    TREE_PLANT ||--o{ CARE_LOG : "logs care and growth"
    
    TREE ||--o{ CARBON_CREDIT : "generates"
    CARBON_CREDIT ||--o{ MARKETPLACE_LISTING : "listed on"
    
    CARBON_BUYER ||--o{ PURCHASE : "executes"
    PURCHASE }o--|| MARKETPLACE_LISTING : "buys"
```

---

## 4. Cross-Entity Contextual Navigation (Detail-to-Detail)

Administrators must never need to perform manual searches to find connected records. Every detail view must provide bidirectional hyperlinks across linked entities.

``` mermaid
flowchart LR
    P["Project Detail"] --> T["Tree Detail"]
    T --> P
    P --> TP["Tree Planter Detail"]
    TP --> P
    T --> TP
    TP --> T
    T --> PR["Planting Record"]
    PR --> T
    PR --> CL["Tree Care Logs<br/>(Care & Growth)"]
    CL --> PR
    T --> CC["Carbon Credit Detail"]
    CC --> T
    CC --> ML["Marketplace Listing"]
    ML --> CC
    ML --> PU["Purchase Order"]
    PU --> ML
    PU --> CB["Carbon Buyer Detail"]
    CB --> PU
```

---

# PART 2: APP SHELL & INFORMATION ARCHITECTURE

## 5. Recommended Dashboard Layout (App Shell)

``` text
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ [LAGRANGE LOGO]   [Global Search (Cmd+K)...                           ]   [Notifications: 8]  [Admin Profile ▾] │
├──────────────────────┬─────────────────────────────────────────────────────────────────────────────────┤
│ SIDEBAR NAVIGATION   │ BREADCRUMBS: Projects / Mumbwa Forest / Trees / TREE-MUM-000001                 │
│                      ├─────────────────────────────────────────────────────────────────────────────────┤
│ Dashboard            │ PAGE HEADER                                                                     │
|                      |                                                                                 |
│ Tree Planters        │ TREE-MUM-000001                          [Badge: Planted] [Badge: Active]       │
│   • All Planters     │ Mango Seedling • Mumbwa Community Forest         [+ Log Care] [Edit] [Actions ▾]│
│   • Verification (34)├─────────────────────────────────────────────────────────────────────────────────┤
│   • Update Reqs (12) │ TABS: [ Overview ] [ Planting Record ] [ Care Logs ] [ Carbon ] [ History ]      │
│                      ├─────────────────────────────────────────────────────────────────────────────────┤
│ Projects             │ PAGE CONTENT AREA                                                               │
|                      |                                                                                 |
│ Trees                │                                                                                 │
│   • All Trees        │                                                                                 │
│   • Generate Trees   │                                                                                 │
│   • Assignments      │                                                                                 │
|                      |                                                                                 |
│ Tree Planting        │                                                                                 │
│   • Planting Records │                                                                                 │
│   • Care Logs        │                                                                                 │
|                      |                                                                                 |
│ Carbon Credits       │                                                                                 │
│ Marketplace          │                                                                                 │
│   • Carbon Buyers    │                                                                                 │
│   • Listings         │                                                                                 │
│   • Purchases        │                                                                                 │
|                      |                                                                                 |
|                      |                                                                                 |
|                      |                                                                                 |
│ Admin Users          │                                                                                 │
│ Audit Logs           │                                                                                 │
│ Settings             │                                                                                 │
└──────────────────────┴─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Information Architecture

``` mermaid
flowchart TD
    A["Admin Dashboard"] --> B["1.0 Dashboard Home"]
    A --> C["2.0 Tree Planters"]
    A --> D["3.0 Projects"]
    A --> E["4.0 Trees & Seedlings"]
    A --> F["5.0 Tree Planting & Care"]
    A --> G["6.0 Carbon Credits"]
    A --> H["7.0 Marketplace"]
    A --> I["8.0 Admin Users & Roles"]
    A --> J["9.0 Reports & Exports"]
    A --> K["10.0 Audit Logs"]
    A --> L["11.0 Settings"]

    C --> C1["2.1 All Tree Planters Directory"]
    C --> C2["2.2 Verification Queue (Badge Count)"]
    C --> C3["2.3 Profile Update Requests"]

    D --> D1["3.1 Project Directory"]
    D --> D2["3.2 Create Project Wizard"]
    D --> D3["3.3 Project Details & Geofence Map"]

    E --> E1["4.1 All Trees Master Table"]
    E --> E2["4.2 Tree Generation Wizard"]
    E --> E3["4.3 Tree Assignment Manager"]

    F --> F1["5.1 Planting Baseline Records"]
    F --> F2["5.2 Tree Care Logs (Activities + Growth Measurements)"]
    F --> F3["5.3 Care Compliance & Exception Tracker"]

    G --> G1["6.1 Carbon Inventory & Issuance Overview"]
    G --> G2["6.2 Credit Ledger & Calculations"]

    H --> H1["7.1 Carbon Buyers Directory"]
    H --> H2["7.2 Marketplace Listings"]
    H --> H3["7.3 Orders & Purchases"]

    I --> I1["8.1 Admin User Management"]
    I --> I2["8.2 Roles & Permissions"]
```

---

## 7. Top Navigation & Global Search

### 7.1 Global Search Modal (`Cmd+K` / `Ctrl+K`)
Supports instantaneous search across all system entities with categorized group headers:

``` text
┌────────────────────────────────────────────────────────────────────────┐
│ Search by Planter name, Tree ID, Project, Carbon Credit, Buyer...     │
├────────────────────────────────────────────────────────────────────────┤
│ TREE PLANTERS                                                          │
│   John Banda — Mumbwa District (Verified)                              │
│   Mary Phiri — Lusaka District (Pending Verification)                  │
│                                                                        │
│ TREES                                                                  │
│   TREE-MUM-000001 — Mango (Planted • Mumbwa Community Forest)          │
│   TREE-MUM-000002 — Acacia (Available • Mumbwa Community Forest)        │
│                                                                        │
│ PROJECTS                                                               │
│   Mumbwa Community Forest — Central Province (Active)                  │
│                                                                        │
│ CARBON CREDITS & PURCHASES                                             │
│   CC-2026-MUM-0042 — 500 tCO₂e (Available)                             │
│   PUR-000123 — ABC Corporation ($12,500 • Completed)                   │
└────────────────────────────────────────────────────────────────────────┘
```

### 7.2 Actionable Notifications Drawer
Contains direct deep-links to urgent queues:
- New tree planter registered (Direct link to verification)
- Profile change request submitted (Direct link to diff review)
- Care frequency non-compliance alert (Direct link to Tree Care Log)
- Carbon purchase completed / settlement required

---

## 8. Dashboard Home (Operational Overview)

The Dashboard Home is an **operational command center** designed to immediately highlight items needing human intervention alongside top-level platform metrics.

### 8.1 Metric Summary Cards

``` text
┌───────────────────────────┐  ┌───────────────────────────┐  ┌───────────────────────────┐  ┌───────────────────────────┐
│ TOTAL TREE PLANTERS       │  │ PENDING VERIFICATION      │  │ ACTIVE PROJECTS           │  │ TOTAL TREES PLANTED       │
│ 1,248                     │  │ 34                        │  │ 18                        │  │ 12,458                    │
│ +24 this month            │  │ Needs Attention           │  │ 2 starting soon           │  │ 94.2% survival rate       │
└───────────────────────────┘  └───────────────────────────┘  └───────────────────────────┘  └───────────────────────────┘
┌───────────────────────────┐  ┌───────────────────────────┐  ┌───────────────────────────┐  ┌───────────────────────────┐
│ ACTIVE GROWING TREES      │  │ TOTAL CARBON GENERATED    │  │ AVAILABLE CREDITS         │  │ MARKETPLACE REVENUE       │
│ 11,923                    │  │ 8,430 tCO₂e               │  │ 2,340 tCO₂e               │  │ $142,500                  │
│ In regular care logging   │  │ Accumulated to date       │  │ Ready for listing         │  │ Net volume processed      │
└───────────────────────────┘  └───────────────────────────┘  └───────────────────────────┘  └───────────────────────────┘
```

### 8.2 "Requires Attention" Action Queue
A prioritized interactive list of pending administrative tasks:

``` text
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ REQUIRES ATTENTION                                                                                     │
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ [34] Tree Planters awaiting identity & location verification                [ Review Queue ➔ ]        │
│ [12] Profile update requests awaiting admin review                          [ Review Requests ➔ ]     │
│ [ 8] Care logs flagged for missed weekly frequency (< 4 logs/wk)             [ View Compliance ➔ ]     │
│ [23] Unassigned generated trees awaiting planter allocation                 [ Assign Trees ➔ ]        │
│ [ 5] Newly calculated carbon credits ready for issuance review              [ Review Credits ➔ ]      │
└────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 8.3 Operational Analytics Widgets
- **Tree Lifecycle Breakdown:** Trees Generated $\rightarrow$ Trees Assigned $\rightarrow$ Trees Planted $\rightarrow$ Active Trees $\rightarrow$ Inactive/Lost Trees.
- **Tree Care & Growth Progress:** Log submission volume vs. expected frequency over time (7D, 30D, 3M, 1Y, Custom range).
- **Carbon Sequestration Curve:** Biomass accumulation and calculated $tCO_2e$ credits generated.

---

# PART 3: MODULE SPECIFICATIONS & SCREENS

## 9. Tree Planters

### 9.1 Tree Planter Directory

``` text
TREE PLANTERS DIRECTORY
Manage registered tree planters, review verification status, and monitor planting activity.

[Search by name, email, phone...  ] [Filter: Location ▾] [Status: All ▾] [Verification: All ▾]  [Export CSV]

☐ Planter Name      Location           Status      Verification   Assigned Trees   Planted   Joined Date   Actions
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
☐ John Banda        Mumbwa, Central    Active      Verified       24               22        12 Mar 2026   [View Details]
☐ Mary Phiri        Lusaka, Lusaka     Pending     Pending        10               0         19 Sep 2026   [Review KYC]
☐ Peter Zulu        Kabwe, Central     Active      Verified       31               31        04 Feb 2026   [View Details]
☐ Grace Mwansa      Chibombo, Central  Suspended   Rejected       0                0         10 Jan 2026   [View Details]

Showing 1–25 of 1,248 tree planters                                              [ < Prev ] [ 1 ] 2  3 [ Next > ]
```

### 9.2 Tree Planter Detail Screen

Header displays high-level metadata, quick status badges, and immediate action buttons (`[Edit Profile]`, `[Suspend Account]`, `[Reset Password]`).

**Tabs:**
1. **Overview:** Full personal info, contact info, administrative location hierarchy, registration timestamps.
2. **Verification:** Checklist of verified components (Personal Info, Location, National ID/Passport).
3. **Documents:** High-resolution zoomable ID document and passport photos.
4. **Projects:** List of participating projects with joined date and assigned quotas.
5. **Trees:** Table of all assigned trees with Tree IDs, species, planting status, and individual carbon yield.
6. **Care Logs:** Unified feed of all care activities and growth measurement logs submitted by this planter.
7. **Profile Update History:** Log of past change requests, admin review timestamps, and approval/rejection notes.
8. **Activity Log:** Audit trail of planter actions and admin interactions.

---

### 9.3 Tree Planter Verification Queue (List View)

A dedicated operational queue table showing all tree planters awaiting identity and location verification, with per-section status indicators.

``` text
TREE PLANTER VERIFICATION QUEUE
Review, validate, or reject submitted KYC documents and geographic locations.

[Tabs: All Pending (34) | Requires Resubmission (8) | Recently Approved | Rejected ]
[Search by name, NRC ID, phone... ] [Filter: Location ▾] [Sort: Oldest First ▾]

☐ Planter Name      Location           Personal   Location   ID Docs    Submitted      Status        Action
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────
☐ John Banda        Mumbwa, Central    Valid      Valid      Pending    20 Sep 10:14   Pending       [Review KYC ➔]
☐ Mary Phiri        Lusaka, Lusaka     Valid      Pending    Pending    19 Sep 16:30   Pending       [Review KYC ➔]
☐ Peter Zulu        Kabwe, Central     Valid      Flagged    Valid      18 Sep 11:20   Incomplete    [Review KYC ➔]
☐ Esther Mwale      Chongwe, Lusaka    Pending    Pending    Pending    18 Sep 09:05   Pending       [Review KYC ➔]

Showing 1–4 of 34 pending verifications                                          [ < Prev ] [ 1 ] 2 [ Next > ]
```

---

### 9.4 Tree Planter Verification Review Screen (Multi-Step Detail View)

A dedicated split-view validation interface where administrators independently inspect, verify, or reject each section of a planter's KYC submission.

``` text
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ BREADCRUMBS: Tree Planters / Verification Queue / John Banda                                           │
│ VERIFICATION REVIEW: John Banda                                              Submission Date: 20 Sep 2026│
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ VERIFICATION PROGRESS: [■■■■■■■■■■■■■■■■■□□□□□□□□] 2 of 3 Sections Approved                            │
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                        │
│ 1. PERSONAL INFORMATION                                                          STATUS: [VERIFIED]     │
│    Full Name:       John Banda                   Date of Birth: 15 March 1990                          │
│    Email:           john@example.com             Gender:        Male                                   │
│    Phone:           +260 97 123 4567             NRC/ID No:     348291/10/1                            │
│                                                                           [ Reject ] [ Edit ] [ Verify ]│
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                        │
│ 2. ADMINISTRATIVE LOCATION & GPS PIN                                             STATUS: [VERIFIED]     │
│    Province:   Central           District:  Mumbwa                                                     │
│    Chiefdom:   Moono             Village:   Kabulwebulwe                                               │
│    GPS Coordinates: -15.02341, 27.91234                                                                │
│    ┌────────────────────────────────────────────────────────────────┐                                  │
│    │  [MAP VIEW: Showing submitted planter pin vs. Project boundary]│                                  │
│    │                           [Planter Location Pin]               │                                  │
│    └────────────────────────────────────────────────────────────────┘     [ Reject ] [ Edit ] [ Verify ]│
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                        │
│ 3. IDENTITY DOCUMENTS                                                            STATUS: [PENDING]      │
│    Document Type: National Registration Card (NRC)                                                     │
│    ┌───────────────────────────┐    ┌───────────────────────────┐                                      │
│    │                           │    │                           │                                      │
│    │     [NRC FRONT/BACK]      │    │     [PASSPORT PHOTO]      │                                      │
│    │                           │    │                           │                                      │
│    └───────────────────────────┘    └───────────────────────────┘                                      │
│    [ Zoom Document Full-Size ]      [ Zoom Photo Full-Size ]                                           │
│                                                                                                        │
│    Document Verification Actions:                                           [ Reject ID ] [ Verify ID ]│
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ FINAL DECISION:                                                    [ Reject All with Reason ] [ Approve KYC ]│
└────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

### 9.5 Profile Update Requests Queue (List View)

A centralized queue of all profile modification requests submitted by verified planters, requiring administrative validation.

``` text
PROFILE UPDATE REQUESTS QUEUE
Review, inspect field diffs, and approve or reject requested profile modifications.

[Tabs: Pending Requests (12) | Approved History | Rejected History ]
[Search by Planter Name, ID, Request #... ] [Filter: Request Type ▾] [Sort: Newest First ▾]

☐ Request ID  Planter Name      Changed Fields            Submitted Date    Stated Reason Excerpt          Action
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
☐ REQ-0042    John Banda        District, Chiefdom, Vill  18 Sep 2026 14:20 "Relocated farming block..."   [Review Diff ➔]
☐ REQ-0041    Mary Phiri        Phone Number              18 Sep 2026 11:05 "Updated primary mobile..."    [Review Diff ➔]
☐ REQ-0040    Peter Zulu        NRC Document Re-upload    17 Sep 2026 16:45 "Clearer photo attached..."    [Review Diff ➔]
☐ REQ-0039    Grace Mwansa      Email Address             16 Sep 2026 09:30 "Changed company email..."     [Review Diff ➔]

Showing 1–4 of 12 pending requests                                               [ < Prev ] [ 1 ] 2 [ Next > ]
```

---

### 9.6 Profile Update Request Review Screen (Side-by-Side Diff)

When an administrator opens a request from the queue, a side-by-side comparison highlights the changes and requires an explanation note upon rejection.

``` text
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ BREADCRUMBS: Tree Planters / Profile Update Requests / #REQ-0042                                       │
│ PROFILE CHANGE REQUEST #REQ-0042                                            Submitted: 18 Sep 2026     │
│ Planter: John Banda (ID: TP-00912)                                                                     │
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ PLANTER'S STATED REASON:                                                                               │
│ "I have relocated from Chongwe farming block to Mumbwa Central chiefdom."                             │
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ FIELD                    CURRENT VALUE                     REQUESTED NEW VALUE                         │
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ District                 Chongwe                           Mumbwa                      [CHANGED]       │
│ Chiefdom                 Nkomeshya                         Moono                       [CHANGED]       │
│ Village                  Shantumbu                         Kabulwebulwe                [CHANGED]       │
│ Phone Number             +260 97 123 4567                  +260 97 123 4567            (Unchanged)     │
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ REJECTION NOTE (Required if rejected):                                                                 │
│ [Enter feedback reason sent to planter's mobile app...                                               ] │
│                                                                                                        │
│                                                     [ Reject Request ]     [ Approve Profile Update ]  │
└────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 10. Projects

### 10.1 Project Directory

Lists reforestation initiatives with progress tracking and quick metrics.

``` text
PROJECTS
[Search projects...              ] [Filter: Province ▾] [Status: Active ▾]                 [ + Create Project ]

Project Name               Location           Status    Planters   Trees Gen   Planted   Carbon (tCO₂e)  Action
───────────────────────────────────────────────────────────────────────────────────────────────────────────────
Mumbwa Community Forest    Mumbwa, Central    Active    248        5,000       4,320     1,240           [Manage]
Chongwe Agroforestry       Chongwe, Lusaka    Active    120        2,500       2,100       680           [Manage]
Kafue Basin Reforestation  Kafue, Lusaka      Planning  0          10,000      0             0           [Manage]
```

### 10.2 Create Project Wizard

1. **Basic Information:** Project title, initiative scope, primary sponsor/partner, banner imagery, target tree capacity.
2. **Administrative Location Hierarchy:** Dependent dropdowns (Province $\rightarrow$ District $\rightarrow$ Constituency $\rightarrow$ Ward $\rightarrow$ Chiefdom).
3. **Planting Site Coordinates & Boundary Map:**
   - Search address/landmark or drop latitude/longitude pins.
   - Interactive map UI to adjust marker or draw a polygon boundary.

``` text
┌───────────────────────────────────────────────────────────────────────┐
│ SEARCH LOCATION: [ Mumbwa Central Forest Area...                    ] │
├───────────────────────────────────────────────────────────────────────┤
│                                                                       │
│                 INTERACTIVE MAP (Satellite / Streets)                 │
│                                                                       │
│                         [Planting Site Pin]                           │
│                         (-15.02341, 27.91234)                         │
│                                                                       │
└───────────────────────────────────────────────────────────────────────┘
  Latitude:  [-15.023410          ]   Longitude: [27.912340           ]
  Site Radius / Area: [250 Hectares]  [ Confirm Geolocation Coordinates ]
```

### 10.3 Project Detail & Summary Statistics

Provides high-level counters and entity tabs:
- **Overview:** Description, boundary map, location metadata, creation date.
- **Tree Planters:** All verified planters assigned to this project.
- **Trees:** Inventory of trees generated, assigned, and planted within this project.
- **Tree Care Logs:** Aggregate care and growth logs submitted across this project.
- **Carbon:** Total verified carbon yield and credit issuance history.

---

## 11. Trees & Assignment

### 11.1 Tree Directory

``` text
TREES DIRECTORY
Master inventory of generated physical trees, species, assigned planters, and current status.

[Search by Tree ID (TREE-XXX)... ] [Project: All ▾] [Species: All ▾] [Status: All ▾]   [ + Generate Trees ]

Tree ID           Species     Project                    Assigned Planter   Status      Planted Date   Action
───────────────────────────────────────────────────────────────────────────────────────────────────────────────
TREE-MUM-000001   Mango       Mumbwa Community Forest    John Banda         Planted     15 Sep 2026    [View]
TREE-MUM-000002   Acacia      Mumbwa Community Forest    John Banda         Planted     15 Sep 2026    [View]
TREE-MUM-000003   Mahogany    Mumbwa Community Forest    Unassigned         Available   —              [Assign]
TREE-MUM-000004   Faidherbia  Mumbwa Community Forest    Unassigned         Available   —              [Assign]
```

### 11.2 Tree Generation Wizard

Administrators generate serialized batches of unique tree identifiers for a specific project.

``` text
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ BATCH TREE GENERATION                                                                                  │
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Select Target Project:        [ Mumbwa Community Forest (Central Province) ▾ ]                         │
│ Tree Species:                 [ Mango (Mangifera indica) ▾ ]                                           │
│ Quantity to Generate:         [ 5000 ] (Max 50,000 per batch)                                          │
│ Identifier Prefix:            [ TREE-MUM- ] ➔ Preview: TREE-MUM-000001 to TREE-MUM-005000              │
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ CONFIRMATION NOTICE:                                                                                   │
│ You are about to permanently generate 5,000 unique tree records. These IDs will be locked and          │
│ formatted for QR-tag assignment and mobile NFC/QR scanning.                                           │
│                                                                                                        │
│                                                [ Cancel ]  [ Generate 5,000 Tree Records ]             │
└────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 11.3 Tree Detail Screen

- **Header:** `TREE-MUM-000001` | Species: **Mango** | Status: **Planted / Healthy** | Carbon: **0.12 tCO₂e**
- **Tabs:**
  - **Overview:** Tree ID, batch number, species details, project name, assignment date, planter profile link.
  - **Planting Record:** Initial baseline record submitted via mobile upon planting (initial GPS coordinates, initial height/stem diameter, initial photo).
  - **Care Logs:** Chronological timeline and charts of all periodic care activities and growth updates.
  - **Carbon Yield:** Calculated biomass and verified carbon sequestration contribution.
  - **Audit History:** Full log of assignment, transfers, status changes.

---

### 11.4 Tree Assignment Manager

Assigns single or bulk unassigned trees to verified tree planters.

``` text
ASSIGN TREES TO PLANTER

Project: [ Mumbwa Community Forest ▾ ]
Available Unassigned Trees in Project: 680 trees

Select Trees:
☑ TREE-MUM-000003 (Mahogany)
☑ TREE-MUM-000004 (Faidherbia)
☑ TREE-MUM-000005 (Mango)
☐ TREE-MUM-000006 (Mango)

Select Tree Planter:
[ Search verified planter by name, ID or phone... ]
Selected Planter: John Banda (Verified • Eligible • 24 active trees)

Selected: 3 Trees to assign to John Banda

[ Cancel ]  [ Confirm Tree Assignment (3 Trees) ]
```

**System Validation Rules:**
- Planter must have `VERIFIED` KYC status.
- Planter must be a member of the selected Project.
- Tree must have status `AVAILABLE` (cannot reassign already planted/assigned trees without formal transfer).

---

## 12. Tree Planting & Tree Care Logs (Unified)

### 12.1 Planting Records (Initial Baseline)

When a tree planter plants an assigned seedling in the field, they log a **Tree Plant Record** via the mobile app. This creates the foundational record for the physical tree.

**Planting Record Fields:**
- Tree ID & Species
- Tree Planter Name & ID
- Project Name & Geolocation (GPS Lat/Lng of planting hole)
- Planting Timestamp
- Baseline Height (cm) & Baseline Stem Diameter (cm)
- Initial Baseline Photo and Video

---

### 12.2 Tree Care Logs (Care Activities + Growth Measurements)

> **Core Concept:** Tree Care Logs and Growth Logs are merged into a unified log entry. Every care submission by a planter captures both **care actions taken** and **periodic physical growth metrics**.

**Unified Care Log Entity Data:**
- **Timestamp & Planter ID**
- **Tree ID & Project Reference**
- **Care Activities Executed (Multi-Select):**
  - Watering
  - Weeding & Clearing
  - Pruning & Staking
  - Pest / Disease Treatment
  - Mulching & Fertilizing
  - Fence / Animal Protection Maintenance
- **Growth & Health Measurements:**
  - Tree Height (in centimeters, e.g., $48\text{ cm}$)
  - Stem Diameter (in millimeters/centimeters, e.g., $1.4\text{ cm}$)
  - Canopy / Leaf Health Rating (Healthy, Average, Stunted, Pest Infested, Wilting)
- **Visual Evidence:** Photo upload (with EXIF timestamp and GPS validation)
- **Planter Field Notes:** Optional text notes

``` text
TREE CARE LOGS
Aggregated feed of field care submissions and growth measurement logs.

[Search by Tree ID, Planter...   ] [Filter: Project ▾] [Activity: All ▾] [Health: All ▾] [Date Range ▾]

Date & Time      Tree ID           Planter        Care Activities Done     Height   Diameter  Health   Photo        Action
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
20 Sep 2026 09:15 TREE-MUM-000001  John Banda     Watered, Weeded          48 cm    1.4 cm    Good     [View Photo] [Details]
18 Sep 2026 14:30 TREE-MUM-000002  John Banda     Watered, Pest Ctrl       36 cm    1.1 cm    Mild     [View Photo] [Details]
17 Sep 2026 11:00 TREE-CHG-000412  Mary Phiri     Watered, Mulched         62 cm    1.9 cm    Good     [View Photo] [Details]
```

---

### 12.3 Growth History Visualization

Inside any Tree Detail view or Project Summary, administrators visualize growth trends over time extracted from the unified Care Logs:

``` text
TREE GROWTH HISTORY & CARE TIMELINE (TREE-MUM-000001)

HEIGHT OVER TIME (cm)                                  STEM DIAMETER OVER TIME (cm)
60 |                                                  2.0 |
50 |                     *                             1.5 |                    *
40 |           *   *                                   1.0 |           *   *
30 |     *                                             0.5 |     *
 0 └─────────────────────────                          0.0 └─────────────────────────
    15 Aug   01 Sep   15 Sep                              15 Aug   01 Sep   15 Sep

LOGGED CARE & MEASUREMENT HISTORY:
• 15 Sep 2026: Height 47 cm (+4 cm) | Diameter 1.4 cm | Care: Watered, Pruned  | Condition: Excellent [View Log]
• 08 Sep 2026: Height 43 cm (+3 cm) | Diameter 1.3 cm | Care: Watered, Weeded  | Condition: Good      [View Log]
• 01 Sep 2026: Height 40 cm (Baseline)| Diameter 1.2 cm | Care: Planted, Mulched| Condition: Good      [View Log]
```

---

### 12.4 Care Frequency & Compliance Monitoring

Planters are required to perform and log care routines at least **4 times per week**. The dashboard automatically surfaces compliance exceptions to prevent seedling mortality.

``` text
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ CARE COMPLIANCE & EXCEPTION MONITORING                                                                 │
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Planter Name: John Banda   | Project: Mumbwa Forest | Active Trees: 24 | Overall Compliance: 92% (Good) │
│                                                                                                        │
│ CURRENT WEEK ACTIVITY MATRIX:                                                                          │
│ Tree ID          Mon         Tue         Wed         Thu         Fri         Sat         Weekly Score  │
│ ───────────────────────────────────────────────────────────────────────────────────────────────────────│
│ TREE-MUM-000001  Watered     Weeded      Missed      Watered     Pruned      —           4 / 4 (100%)  │
│ TREE-MUM-000002  Watered     Missed      Missed      Watered     Missed      —           2 / 4 (50% !) │
│                                                                                                        │
│ EXCEPTION ALERT:                                                                                       │
│ 4 trees under John Banda have received less than 3 care logs this week.                                │
│ [ Send App Reminder Notification ]  [ Assign Field Extension Officer ]                                 │
└────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 13. Carbon Credits & Marketplace

### 13.1 Tree-to-Carbon Lifecycle

``` mermaid
flowchart TD
    A["Tree Planted & Initial Baseline Logged"] --> B["Continuous Tree Care Logs Submitted"]
    B --> C["Biomass & Growth Rate Calculated<br/>(Height + Diameter + Species Model)"]
    C --> D["Carbon Sequestration Calculation Formula Run"]
    D --> E["Carbon Credit Issued (tCO₂e) with Unique Certificate ID"]
    E --> F["Credit Listed on Public Marketplace"]
    F --> G["Carbon Buyer Purchases Credit"]
    G --> H["Credit Retired or Transferred to Buyer Portfolio"]
```

### 13.2 Carbon Inventory Overview

``` text
CARBON CREDITS INVENTORY

┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────┐
│ TOTAL GENERATED     │  │ AVAILABLE FOR SALE  │  │ RESERVED (IN CART)  │  │ SOLD & RETIRED      │
│ 8,430 tCO₂e         │  │ 2,340 tCO₂e         │  │ 500 tCO₂e           │  │ 5,590 tCO₂e         │
│ Verified credits    │  │ Unlisted or active  │  │ Checkout in prog.   │  │ Permanently settled │
└─────────────────────┘  └─────────────────────┘  └─────────────────────┘  └─────────────────────┘

[Search by Credit Batch ID...    ] [Filter: Project ▾] [Vintage: 2026 ▾] [Status: Available ▾]  [ + Issue Batch ]

Batch ID        Project                   Vintage   Total Volume   Available   Price/tCO₂e   Status     Action
───────────────────────────────────────────────────────────────────────────────────────────────────────────────
CC-2026-MUM-01  Mumbwa Community Forest   2026      1,240 tCO₂e    450 tCO₂e   $25.00        Listed     [View Details]
CC-2026-CHG-02  Chongwe Agroforestry      2026        680 tCO₂e    680 tCO₂e   $22.50        Available  [List for Sale]
```

### 13.3 Carbon Buyers Directory
Displays corporate and individual buyers purchasing carbon offsets:
- Buyer Name / Organization
- Primary Contact Email & Billing Country
- KYC / Tax Verification Status
- Total Purchases & Lifetime Carbon Volume Retired ($tCO_2e$)

### 13.4 Marketplace Listings & Purchases
- **Listings Management:** Create, pause, or update unit pricing for carbon credit batches.
- **Purchase Order Table:** Order ID, Buyer, Project Source, Volume ($tCO_2e$), Gross Amount ($USD$), Transaction Date, Payment Status, Certificate Link.

### 13.5 Purchase Detail Screen (Provenance Tracing)

``` text
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ PURCHASE ORDER: PUR-000123                                                   Status: [COMPLETED]       │
│ Buyer: ABC Sustainable Tech Corp               Transaction Date: 20 September 2026                     │
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ ORDER SUMMARY:                                                                                         │
│ Total Carbon Purchased:  500 tCO₂e                 Unit Price:       $25.00 / tCO₂e                    │
│ Gross Settlement:        $12,500.00 USD            Payment Method:   Wire Transfer (Ref: #WT-9821)     │
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ PROVENANCE & CARBON ORIGIN:                                                                            │
│ Sourcing Project:        Mumbwa Community Forest (Central Province)                                    │
│ Credit Certificate ID:   CERT-2026-MUM-000123-RETIRED                                                  │
│ Underlying Trees:        1,250 Trees contributing (TREE-MUM-000001 through TREE-MUM-001250)             │
│ Growth Logs Verified:    5,400 verified Care Logs submitted by 42 local tree planters                  │
│                                                                                                        │
│ [ Download Certificate PDF ]  [ View Audit Trail ]  [ View Contributing Trees (1,250) ]                │
└────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 14. Administration & Governance

### 14.1 Admin Users & Role-Based Access Control (RBAC)

No public admin registration. System seeds root super-admin; super-admins invite new staff.

**Role Permissions Matrix:**
| Module / Capability | Super Admin | Verification Officer | Project Manager | Carbon Auditor |
| :--- | :---: | :---: | :---: | :---: |
| Admin User Management | Full | None | None | None |
| Planter Verification / Rejection | Full | Full | Read Only | Read Only |
| Project Creation & Geofencing | Full | Read Only | Full | Read Only |
| Tree Batch Generation | Full | None | Full | Read Only |
| Care Log & Compliance Monitoring | Full | Full | Full | Full |
| Carbon Credit Calculation & Issuance | Full | None | Read Only | Full |
| Marketplace Pricing & Settings | Full | None | None | Read Only |
| Audit Logs Inspection | Full | Read Only | Read Only | Full |

### 14.2 Audit Logs

Every critical change (verifications, profile approvals, tree generation, credit issuance, configuration updates) must be immutably recorded.

``` text
AUDIT TRAIL
[Search by Admin, Action, Entity ID...] [Filter: Date Range ▾] [Module: All ▾] [Action Type: All ▾]

Timestamp         Admin User      Action Performed             Affected Entity        Before / After Diff
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────
20 Sep 10:32:15   Alice Admin     Approved Planter KYC         Planter: John Banda    Status: Pending ➔ Verified
20 Sep 09:14:02   Bob Admin       Generated 5,000 Tree IDs     Project: Mumbwa Forest Batch: TREE-MUM-01
19 Sep 16:45:20   Alice Admin     Approved Profile Change      Planter: John Banda    District: Chongwe ➔ Mumbwa
19 Sep 11:20:11   Dave Auditor    Issued Carbon Credit Batch   Credit: CC-2026-MUM-01 1,240 tCO₂e Minted
```

### 14.3 System Settings & Carbon Parameters
Protected configuration requiring dual confirmation:
- **Tree Species Master Data:** Allometric equations, default carbon absorption rates per species.
- **Care Compliance Rules:** Minimum required logs per week (default: 4), warning thresholds.
- **Identifier Prefix Templates:** Serial generation formatting rules.

---

# PART 4: GLOBAL UI PATTERNS & DESIGN STATES

## 15. Data Tables (Search, Filters, Bulk Actions, Pagination)

All tabular views in the dashboard must follow consistent design rules:

``` text
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ [Search table contents...                    ] [Filter 1 ▾] [Filter 2 ▾] [Columns ▾]   [Bulk Action ▾] │
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ 3 items selected                                               [ Assign Selected ] [ Export Selected ]│
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ ☐   IDENTIFIER ↕       NAME / TITLE ↕       LOCATION ↕       STATUS ↕       METRIC ↕       ACTIONS   │
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ ☑   TREE-MUM-000001    Mango Seedling       Mumbwa           Planted        0.12 tCO₂e     [View ➔]  │
│ ☑   TREE-MUM-000002    Acacia Seedling      Mumbwa           Planted        0.09 tCO₂e     [View ➔]  │
│ ☑   TREE-MUM-000003    Mahogany             Mumbwa           Available      0.00 tCO₂e     [Assign ➔]│
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Showing 1–25 of 1,248 items                                    Page [ 1 ] of 50  [ < Prev ] [ Next > ] │
└────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

**Required Table Features:**
- Persistent search and filter bar with clear pill indicators for active filters.
- Tri-state checkbox selection header (None, Some, All).
- Sticky header when scrolling vertically.
- Responsive column truncation with tooltips for overflowing text.

---

## 16. UI States (Loading, Empty, Error, Confirmation)

Every major page and component must explicitly define these four fundamental states:

### 16.1 Skeleton Loading State
Use shimmering placeholder blocks mimicking exact layout geometry (avoid blocking full-page spinners).

``` text
┌───────────────────────────────────────────────────────────────────────┐
│ ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒             ▒▒▒▒▒▒▒▒▒▒▒▒▒                            │
├───────────────────────────────────────────────────────────────────────┤
│ ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒ │
│ ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒ │
│ ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒ │
└───────────────────────────────────────────────────────────────────────┘
```

### 16.2 Empty State
Never display a bare empty table. Provide an informative illustration, clear message, and primary call-to-action button.

``` text
┌───────────────────────────────────────────────────────────────────────┐
│                                                                       │
│                           (Illustration)                              │
│                          NO PROJECTS FOUND                            │
│                                                                       │
│       There are no active reforestation projects matching your        │
│       search criteria. Create a new project to get started.           │
│                                                                       │
│                       [ + Create New Project ]                        │
│                                                                       │
└───────────────────────────────────────────────────────────────────────┘
```

### 16.3 Error & Recovery State
Explain clearly what happened and provide an immediate actionable recovery button.

``` text
┌───────────────────────────────────────────────────────────────────────┐
│                     UNABLE TO LOAD CARE LOGS                          │
│      A connection timeout occurred while communicating with the       │
│      carbon calculation engine. Your session remains authenticated.    │
│                                                                       │
│                         [ Retry Request ]                             │
└───────────────────────────────────────────────────────────────────────┘
```

### 16.4 Confirmation & Destructive Action Modals

``` text
┌───────────────────────────────────────────────────────────────────────┐
│ SUSPEND TREE PLANTER ACCOUNT?                                         │
├───────────────────────────────────────────────────────────────────────┤
│ Are you sure you want to suspend John Banda (TP-00912)?               │
│                                                                       │
│ Suspending this account will:                                         │
│ • Block the planter from submitting new Tree Care Logs via mobile.    │
│ • Halt automated carbon credit calculations for 24 assigned trees.    │
│                                                                       │
│ Reason for Suspension (Required for audit log):                       │
│ [Enter detailed justification note...                               ] │
│                                                                       │
│                              [ Cancel ]  [ Confirm Account Suspension]│
└───────────────────────────────────────────────────────────────────────┘
```

---

## 17. Detail Page Standards & Breadcrumb Conventions

### 17.1 Breadcrumb Formatting
Always display the complete hierarchical path from root directory to entity:
- `Projects / Mumbwa Community Forest / Trees / TREE-MUM-000001`
- `Tree Planters / Verification Queue / John Banda`
- `Carbon Credits / Batches / CC-2026-MUM-01`

### 17.2 Status Badge Standardization
Use consistent semantic color styling across all badges:
- **Success / Verified / Active / Planted / Completed** (Green pill)
- **Pending / Under Review / In Progress / Reserved** (Amber / Yellow pill)
- **Rejected / Suspended / Failed / Dead / Inactive** (Red pill)
- **Available / Unassigned / Listed** (Blue / Cyan pill)
- **Archived / Retired / Draft** (Neutral Slate pill)

