# AgriLogiX — System Architecture

> Based on [About_project.md](./About_project.md)

> **Note:** All diagrams below use [Mermaid](https://github.blog/2022-02-14-include-diagrams-markdown-files-mermaid/). They render automatically when you view this file on GitHub — no extra setup needed.

AgriLogiX is a **farm-to-fork supply chain platform**. Two roles log in today: **Farmer** and **User (Buyer)**. After auth, each role gets a different dashboard and permissions. The core journey is: **list/source produce → store (optional) → transport → pay → trace**. Transport is GPS-driven with live tracking; payments are transparent settlements; traceability (QR/blockchain) is planned for full chain visibility.

---

## 1. High-Level System Architecture

```mermaid
flowchart TB
    subgraph Clients["Client Layer"]
        WEB["React Web App"]
        MOB["React Native Mobile"]
    end

    subgraph Auth["Auth and Identity"]
        LOGIN["Login / Register"]
        RBAC["Role Router<br/>Farmer | User Buyer"]
        JWT["JWT / Session + RBAC"]
    end

    subgraph Gateway["API Gateway"]
        GW["REST API + WebSocket Hub"]
    end

    subgraph CoreModules["Core Business Modules"]
        USER_MOD["User Module<br/>Buyer"]
        FARM_MOD["Farmer Module"]
        TRANS_MOD["Transport Module<br/>GPS"]
        PAY_MOD["Payment Module"]
        STORE_MOD["Storage Module"]
        TRACE_MOD["Traceability Module<br/>Future"]
    end

    subgraph Services["Shared Services"]
        NOTIF["Notifications / Push"]
        MAPS["Maps API<br/>Google / Mapbox"]
        PRICING["Dynamic Pricing Engine"]
        AI_ASSIGN["AI Vehicle Assignment"]
        MARKET["Market Price Feed"]
    end

    subgraph Data["Data Layer"]
        PG["PostgreSQL<br/>Users, Orders, Payments"]
        MONGO["MongoDB<br/>GPS logs, Tracking events"]
        REDIS["Redis<br/>Live location cache"]
    end

    subgraph External["External Integrations"]
        RAZOR["Razorpay / Stripe"]
        BLOCK["Blockchain<br/>Hyperledger / Ethereum<br/>Future"]
        SMS["SMS / OTP Provider"]
    end

    WEB --> LOGIN
    MOB --> LOGIN
    LOGIN --> RBAC --> JWT --> GW

    GW --> USER_MOD
    GW --> FARM_MOD
    GW --> TRANS_MOD
    GW --> PAY_MOD
    GW --> STORE_MOD
    GW --> TRACE_MOD

    USER_MOD --> PG
    FARM_MOD --> PG
    TRANS_MOD --> MONGO
    TRANS_MOD --> REDIS
    PAY_MOD --> PG
    STORE_MOD --> PG
    TRACE_MOD --> BLOCK

    TRANS_MOD --> MAPS
    TRANS_MOD --> AI_ASSIGN
    TRANS_MOD --> PRICING
    FARM_MOD --> MARKET
    PAY_MOD --> RAZOR
    GW --> NOTIF
    JWT --> SMS
```

---

## 2. Role-Based Login and Routing

```mermaid
flowchart LR
    START(["User opens app"]) --> AUTH{"Login / Register"}

    AUTH -->|Farmer| F_AUTH["Farmer Auth"]
    AUTH -->|"User / Buyer"| U_AUTH["Buyer Auth"]

    F_AUTH --> F_DASH["Farmer Dashboard"]
    U_AUTH --> U_DASH["Buyer Dashboard"]

    subgraph FarmerAccess["Farmer - allowed actions"]
        F1["List produce and harvest dates"]
        F2["View market prices"]
        F3["Book storage"]
        F4["Book transport on GPS map"]
        F5["Track shipment live"]
        F6["View payment settlements"]
    end

    subgraph BuyerAccess["User Buyer - allowed actions"]
        U1["Browse verified produce"]
        U2["Filter by location / quality / qty"]
        U3["Place bulk orders"]
        U4["Track order and transport"]
        U5["Pay and view invoice breakdown"]
        U6["Scan QR traceability - Future"]
    end

    F_DASH --> F1
    F_DASH --> F2
    F_DASH --> F3
    F_DASH --> F4
    F_DASH --> F5
    F_DASH --> F6

    U_DASH --> U1
    U_DASH --> U2
    U_DASH --> U3
    U_DASH --> U4
    U_DASH --> U5
    U_DASH --> U6
```

### Extra Login Logic (Recommended)

| Logic | Purpose |
|--------|---------|
| Role at signup | Farmer vs User selected once; KYC differs per role |
| OTP / phone verify | Common in agri platforms for trust |
| Farmer KYC | Land docs, crop certs, quality certifications |
| Buyer KYC | Business GST, export license (for exporters) |
| JWT + role claims | API enforces `farmer:*` vs `buyer:*` scopes |
| Route guards | Farmer cannot access buyer checkout; buyer cannot list produce |

---

## 3. End-to-End Business Flow (Start to End, incl. Future)

```mermaid
sequenceDiagram
    autonumber
    participant F as Farmer
    participant B as "User (Buyer)"
    participant APP as AgriLogiX Platform
    participant ST as Storage Module
    participant TR as "Transport Module (GPS)"
    participant PAY as Payment Module
    participant TRACE as "Traceability (Future)"

    Note over F,B: Phase 1 - Onboarding
    F->>APP: Register as Farmer + KYC
    B->>APP: Register as Buyer + KYC

    Note over F,B: Phase 2 - Supply and Demand
    F->>APP: List produce (crop, qty, cert, harvest date)
    APP->>B: Notify matching buyers
    B->>APP: Search, filter, place bulk order

    Note over F,B: Phase 3 - Storage (optional)
    F->>ST: Check nearby cold storage capacity
    ST->>APP: Real-time availability
    F->>ST: Book storage slot

    Note over F,B: Phase 4 - Transport (GPS)
    F->>TR: Open map, filter vehicles (reefer, tonnage)
    TR->>APP: Proximity search + dynamic fare
    APP->>TR: AI assigns best vehicle
    TR->>F: Driver notified, live status (En Route to Delivered)
    B->>APP: Live track on map (WebSocket)

    Note over F,B: Phase 5 - Payment
    B->>PAY: Pay order (Razorpay/Stripe)
    PAY->>APP: Escrow / split settlement
    PAY->>F: Farmer payout (minus transport + storage fees)
    PAY->>TR: Transporter payout

    Note over F,B: Phase 6 - Traceability (Future)
    APP->>TRACE: Generate QR at farm pickup
    TRACE->>B: Scan full chain history (farm to buyer)
    TRACE->>TRACE: Optional blockchain anchor
```

---

## 4. Module Breakdown

| Module | Owner role | Key responsibilities | Status |
|--------|------------|----------------------|--------|
| **User (Buyer)** | User | Sourcing, filters, bulk orders, order tracking | Core |
| **Farmer** | Farmer | Produce listing, price discovery, booking storage/transport | Core |
| **Transport** | Farmer (books), system (assigns) | GPS map, proximity search, booking, ETA, ratings | Core |
| **Payment** | Both | Invoices, escrow, split payouts, fee breakdown | Core |
| **Storage** | Farmer | Capacity view, dynamic booking | Core (doc) |
| **Traceability** | Both | QR scan, chain-of-custody, blockchain | Future |

---

## 5. Transport Module (Logical Sub-Flow)

```mermaid
flowchart TD
    A["Farmer opens Transport tab"] --> B["GPS centers on farm"]
    B --> C["Filter: vehicle type, radius 10-50 km"]
    C --> D["Map shows live vehicle markers"]
    D --> E["Tap marker: driver, rating, fare estimate"]
    E --> F{"Book Now?"}
    F -->|Yes| G["Create booking request"]
    G --> H["AI picks best vehicle if multiple"]
    H --> I["Driver gets push notification"]
    I --> J["WebSocket: En Route, At Farm, Loaded, Delivered"]
    J --> K["Payment module: transport fee settled"]
```

---

## 6. Backend Service Map (Suggested)

```mermaid
flowchart TB
    WEB["agrilogix-ui<br/>React Web"]
    MOBILE["agrilogix-mobile<br/>Future"]
    GW["API Gateway"]

    AUTH_SVC["auth-service<br/>roles, JWT"]
    USER_SVC["user-service<br/>buyer profile"]
    FARMER_SVC["farmer-service<br/>listings, KYC"]

    TRANS_SVC["transport-service<br/>GPS, booking"]
    PAY_SVC["payment-service<br/>Razorpay"]
    STORE_SVC["storage-service<br/>capacity"]

    TRACE_SVC["traceability-service<br/>Future"]

    DATA["PostgreSQL + MongoDB + Redis<br/>Maps API + WebSockets"]

    WEB --> GW
    MOBILE --> GW

    GW --> AUTH_SVC
    GW --> USER_SVC
    GW --> FARMER_SVC
    GW --> TRANS_SVC
    GW --> PAY_SVC
    GW --> STORE_SVC

    AUTH_SVC --> DATA
    USER_SVC --> DATA
    FARMER_SVC --> DATA
    TRANS_SVC --> DATA
    PAY_SVC --> DATA
    STORE_SVC --> DATA

    TRACE_SVC --> DATA
    GW --> TRACE_SVC
```

---

## 7. Data Flow Summary

1. **Login** → role → scoped token → correct dashboard
2. **Farmer lists** → catalog indexed → buyers discover
3. **Buyer orders** → order created → notifications to farmer
4. **Storage booked** (if needed) → capacity reserved
5. **Transport booked** → GPS match → AI assign → live track
6. **Payment** → buyer pays → platform splits to farmer + transporter (+ storage)
7. **Future trace** → QR at each hop → buyer scans full history

---

## 8. Tech Stack Alignment

| Layer | Choice |
|-------|--------|
| Frontend | React (web) / React Native (mobile) |
| Backend | Node.js or Python (Django) |
| DB | PostgreSQL + MongoDB |
| Real-time | WebSockets / Socket.io |
| Maps | Google Maps / Mapbox |
| Payments | Razorpay / Stripe |
| Traceability | Hyperledger / Ethereum (optional, future) |
