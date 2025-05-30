# Field Operations Support Agent - Architecture Diagram

## System Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        FIELD OPERATIONS SUPPORT AGENT                       │
│                          (MuleSoft Application)                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────┐    ┌─────────────────────────────────────────────────────┐
│   Field Tech    │    │                 AI Assistant                       │
│   🧑‍🔧 Mobile    │◄──►│              (Claude via MCP)                      │
│   Device        │    │                                                     │
└─────────────────┘    └─────────────────────────────────────────────────────┘
                                               │
                                               │ MCP Protocol
                                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            MULE RUNTIME                                     │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────────┐  │
│  │  global-config  │  │ error-handlers  │  │     main flows              │  │
│  │     .xml        │  │     .xml        │  │                             │  │
│  │                 │  │                 │  │  ┌─────────────────────────┐ │  │
│  │ • HTTP Config   │  │ • DB Errors     │  │  │    MCP Tool Listeners   │ │  │
│  │ • DB Configs    │  │ • HTTP Errors   │  │  │                         │ │  │
│  │ • Claude AI     │  │ • Validation    │  │  │ 1. SOP Manual Access    │ │  │
│  │ • MCP Server    │  │ • Timeouts      │  │  │ 2. Asset History        │ │  │
│  │                 │  │ • Auth Errors   │  │  │ 3. Parts Inventory      │ │  │
│  └─────────────────┘  │ • General       │  │  │ 4. Work Order Info      │ │  │
│                       └─────────────────┘  │  └─────────────────────────┘ │  │
│                                            └─────────────────────────────┘ │  │
└─────────────────────────────────────────────────────────────────────────────┘
                                               │
                                               │ Database Queries
                                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        BACKEND DATABASES                                    │
│                                                                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────────┐  │
│  │ Asset Management│  │ Inventory Mgmt  │  │   Work Order Management     │  │
│  │    Database     │  │    Database     │  │        Database             │  │
│  │                 │  │                 │  │                             │  │
│  │ • Assets        │  │ • Parts         │  │ • Work Orders               │  │
│  │ • Maintenance   │  │ • Inventory     │  │ • Customer Info             │  │
│  │   History       │  │ • Locations     │  │ • Scheduling                │  │
│  │ • Known Issues  │  │ • Alternatives  │  │ • Requirements              │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Data Flow Architecture

```
TECHNICIAN REQUEST FLOW:
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  1. "What's the SOP for transformer replacement?"                           │
│     ┌─────────────┐    Natural Language    ┌─────────────────────────────┐  │
│     │Field Tech   │─────────────────────────►│     AI Assistant (Claude)   │  │
│     │Mobile App   │                         │                             │  │
│     └─────────────┘                         └─────────────────────────────┘  │
│                                                           │                 │
│  2. AI converts to structured MCP tool call               │                 │
│     ┌─────────────────────────────────────────────────────┼─────────────────┤
│     │ {                                                   ▼                 │
│     │   "tool": "getSopManual",               ┌─────────────────────────────┐ │
│     │   "parameters": {                       │     MCP Tool Listener       │ │
│     │     "equipmentType": "transformer",     │   (sop-manual-access-flow)  │ │
│     │     "procedureType": "replacement"      │                             │ │
│     │   }                                     └─────────────────────────────┘ │
│     │ }                                                   │                 │
│     └─────────────────────────────────────────────────────┼─────────────────┤
│                                                           │                 │
│  3. Route to appropriate subflow                          ▼                 │
│     ┌─────────────────────────────────────────────────────────────────────┐ │
│     │               CHOICE ROUTER                                         │ │
│     │                                                                     │ │
│     │  if (equipmentType contains 'transformer')                         │ │
│     │    ├── get-transformer-sop-subflow                                 │ │
│     │  elif (equipmentType contains 'fuse')                              │ │
│     │    ├── get-fuse-sop-subflow                                        │ │
│     │  elif (equipmentType contains 'meter')                             │ │
│     │    ├── get-meter-sop-subflow                                       │ │
│     │  else                                                               │ │
│     │    └── get-general-sop-subflow                                     │ │
│     └─────────────────────────────────────────────────────────────────────┘ │
│                                           │                                 │
│  4. Generate contextual SOP               ▼                                 │
│     ┌─────────────────────────────────────────────────────────────────────┐ │
│     │              DATAWEAVE TRANSFORMATION                               │ │
│     │                                                                     │ │
│     │  %dw 2.0                                                            │ │
│     │  output application/java                                            │ │
│     │  ---                                                                │ │
│     │  {                                                                  │ │
│     │    title: "Transformer Replacement Procedure",                     │ │
│     │    safetyPrecautions: [...],                                       │ │
│     │    steps: [...],                                                    │ │
│     │    estimatedTime: "4-6 hours"                                       │ │
│     │  }                                                                  │ │
│     └─────────────────────────────────────────────────────────────────────┘ │
│                                           │                                 │
│  5. Format for human consumption          ▼                                 │
│     ┌─────────────────────────────────────────────────────────────────────┐ │
│     │              RESPONSE FORMATTING                                    │ │
│     │                                                                     │ │
│     │  "**Transformer Replacement Procedure**                            │ │
│     │                                                                     │ │
│     │   **Safety Precautions:**                                           │ │
│     │   • Verify power isolation and lockout/tagout                      │ │
│     │   • Test for voltage using approved equipment                      │ │
│     │                                                                     │ │
│     │   **Steps:**                                                        │ │
│     │   1. Coordinate with system operations                              │ │
│     │   2. Verify transformer is de-energized                            │ │
│     │   ..."                                                              │ │
│     └─────────────────────────────────────────────────────────────────────┘ │
│                                           │                                 │
│  6. Return to technician                  ▼                                 │
│     ┌─────────────┐      Formatted Text   ┌─────────────────────────────┐  │
│     │Field Tech   │◄─────────────────────│     AI Assistant (Claude)   │  │
│     │Mobile App   │                       │                             │  │
│     └─────────────┘                       └─────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Tool Architecture Detail

```
MCP TOOL ECOSYSTEM:
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  ┌─────────────────────┐    ┌─────────────────────┐                        │
│  │   Tool 1: SOP       │    │   Tool 2: Asset    │                        │
│  │   Manual Access     │    │   History Lookup   │                        │
│  │                     │    │                     │                        │
│  │ Input:              │    │ Input:              │                        │
│  │ • equipmentType     │    │ • assetId           │                        │
│  │ • procedureType     │    │ • location          │                        │
│  │ • modelNumber       │    │ • equipmentType     │                        │
│  │                     │    │                     │                        │
│  │ Output:             │    │ Output:             │                        │
│  │ • Safety steps      │    │ • Maintenance hist  │                        │
│  │ • Procedures        │    │ • Known issues      │                        │
│  │ • Required tools    │    │ • Next maintenance  │                        │
│  │ • Time estimates    │    │ • Asset details     │                        │
│  └─────────────────────┘    └─────────────────────┘                        │
│                                                                             │
│  ┌─────────────────────┐    ┌─────────────────────┐                        │
│  │   Tool 3: Parts     │    │   Tool 4: Work      │                        │
│  │   Inventory Check   │    │   Order Info        │                        │
│  │                     │    │                     │                        │
│  │ Input:              │    │ Input:              │                        │
│  │ • partNumber        │    │ • workOrderNumber   │                        │
│  │ • partDescription   │    │ • technicianId      │                        │
│  │ • location          │    │ • status            │                        │
│  │ • findAlternatives  │    │                     │                        │
│  │                     │    │ Output:             │                        │
│  │ Output:             │    │ • Customer info     │                        │
│  │ • Availability      │    │ • Job details       │                        │
│  │ • Locations         │    │ • Safety notes      │                        │
│  │ • Alternatives      │    │ • Equipment info    │                        │
│  │ • Recommendations   │    │ • Scheduling        │                        │
│  └─────────────────────┘    └─────────────────────┘                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Configuration & Environment Management

```
CONFIGURATION ARCHITECTURE:
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────┐ │
│  │                   ENVIRONMENT-SPECIFIC CONFIG                           │ │
│  │                                                                         │ │
│  │  Development (config-dev.properties):                                  │ │
│  │  ├── http.port=8082                                                     │ │
│  │  ├── asset.db.url=localhost:5432/asset_management_dev                  │ │
│  │  ├── logging.level=DEBUG                                               │ │
│  │  └── docs.api.url=https://docs-api-dev.utility.com                     │ │
│  │                                                                         │ │
│  │  Production (config-prod.properties):                                  │ │
│  │  ├── http.port=8081                                                     │ │
│  │  ├── asset.db.url=prod-db.utility.com:5432/asset_management            │ │
│  │  ├── logging.level=INFO                                                │ │
│  │  └── docs.api.url=https://docs-api.utility.com                         │ │
│  └─────────────────────────────────────────────────────────────────────────┘ │
│                                           │                                 │
│                              Property Resolution                            │
│                                           ▼                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐ │
│  │                    GLOBAL CONFIGURATION                                 │ │
│  │                                                                         │ │
│  │  <configuration-properties file="config-${mule.env}.properties"/>      │ │
│  │                                                                         │ │
│  │  HTTP Listener: host="${http.host}" port="${http.port}"                │ │
│  │  Database: url="${asset.db.url}" user="${secure::asset.db.user}"       │ │
│  │  Claude AI: apiKey="${secure::claude.api.key}"                         │ │
│  │  MCP Server: serverName="field-operations-support-agent"               │ │
│  └─────────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Error Handling Flow

```
ERROR HANDLING ARCHITECTURE:
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────┐ │
│  │                        GLOBAL ERROR HANDLER                             │ │
│  │                                                                         │ │
│  │  Database Errors (DB:CONNECTIVITY)                                     │ │
│  │  ├── Log error details                                                  │ │
│  │  ├── Return HTTP 503 Service Unavailable                               │ │
│  │  └── Structured JSON response with correlation ID                      │ │
│  │                                                                         │ │
│  │  HTTP Errors (HTTP:CONNECTIVITY, HTTP:TIMEOUT)                         │ │
│  │  ├── Log connection/timeout details                                     │ │
│  │  ├── Return HTTP 502 Bad Gateway / 408 Timeout                         │ │
│  │  └── User-friendly error message                                        │ │
│  │                                                                         │ │
│  │  Validation Errors (APIKIT:BAD_REQUEST)                                 │ │
│  │  ├── Log validation failure                                             │ │
│  │  ├── Return HTTP 400 Bad Request                                        │ │
│  │  └── Include validation details                                         │ │
│  │                                                                         │ │
│  │  Authentication Errors (HTTP:UNAUTHORIZED)                              │ │
│  │  ├── Log auth failure                                                   │ │
│  │  ├── Return HTTP 401 Unauthorized                                       │ │
│  │  └── Request credentials                                                │ │
│  │                                                                         │ │
│  │  General Errors (ANY)                                                   │ │
│  │  ├── Log unexpected error                                               │ │
│  │  ├── Return HTTP 500 Internal Server Error                             │ │
│  │  └── Generic error message                                              │ │
│  └─────────────────────────────────────────────────────────────────────────┘ │
│                                           │                                 │
│                               Applied to All Flows                         │
│                                           ▼                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │SOP Manual   │  │Asset History│  │Parts Inv    │  │Work Order Info      │ │
│  │Access Flow  │  │Lookup Flow  │  │Check Flow   │  │Flow                 │ │
│  │             │  │             │  │             │  │                     │ │
│  │<error-      │  │<error-      │  │<error-      │  │<error-handler       │ │
│  │handler ref= │  │handler ref= │  │handler ref= │  │ref="global-error-   │ │
│  │"global-..."/>│  │"global-..."/>│  │"global-..."/>│  │handler"/>           │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────────────┘ │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Testing Strategy

```
MUNIT TESTING ARCHITECTURE:
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────┐ │
│  │                       UNIT TESTS                                        │ │
│  │                                                                         │ │
│  │  SOP Subflow Tests:                                                     │ │
│  │  ├── test-sop-transformer-replacement                                   │ │
│  │  ├── test-sop-fuse-replacement                                          │ │
│  │  ├── test-sop-meter-installation                                        │ │
│  │  └── test-sop-general-equipment                                         │ │
│  │                                                                         │ │
│  │  Work Order Tests:                                                      │ │
│  │  ├── test-specific-work-order-retrieval                                 │ │
│  │  └── test-technician-work-orders-retrieval                              │ │
│  │                                                                         │ │
│  │  Error Handling Tests:                                                  │ │
│  │  ├── test-error-handling-database-connectivity                          │ │
│  │  └── test-error-handling-http-connectivity                              │ │
│  └─────────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  Test Pattern:                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐ │
│  │                                                                         │ │
│  │  <munit:test name="test-name">                                          │ │
│  │    <munit:behavior>                                                     │ │
│  │      <!-- Set up test data and variables -->                           │ │
│  │    </munit:behavior>                                                    │ │
│  │    <munit:execution>                                                    │ │
│  │      <!-- Execute the flow/subflow being tested -->                    │ │
│  │    </munit:execution>                                                   │ │
│  │    <munit:validation>                                                   │ │
│  │      <!-- Assert expected outcomes -->                                  │ │
│  │    </munit:validation>                                                  │ │
│  │  </munit:test>                                                          │ │
│  └─────────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```