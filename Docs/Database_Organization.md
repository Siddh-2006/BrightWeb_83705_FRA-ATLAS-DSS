# 🗃️ FRA Management System - Database Organization

## 📋 Table of Contents
- [Overview](#overview)
- [Design Goals](#design-goals)
- [Data Management Approach](#data-management-approach)
- [High-Level Architecture](#high-level-architecture)
- [Core Entities](#core-entities)
- [Entity Relationship Model](#entity-relationship-model)
- [Data Flow Lifecycle](#data-flow-lifecycle)
- [Access & Storage Strategy](#access--storage-strategy)
- [Scalability and Maintenance](#scalability-and-maintenance)
- [Performance Optimization](#performance-optimization)
- [Summary](#summary)

---

## 🌐 Overview

The **Database Organization** defines how all critical information—ranging from FRA documents and geospatial data to AI-processed results—is structured, interlinked, and accessed across the FRA Management System.  

We've designed a **logical, flexible, and scalable data model** that supports document processing, spatial analysis, AI integration, and decision support while maintaining data integrity and security.

> ⚡ **Implementation Ready:** This architecture is designed for immediate development with clear migration paths for future enhancements.

---

## 🎯 Design Goals

| Goal | Description | Implementation Approach |
|------|-------------|------------------------|
| **Modularity** | Each data domain is self-contained but interoperable | Separate databases for documents, spatial data, users |
| **Scalability** | Handle millions of documents and spatial features | Cloud-native design with horizontal scaling |
| **Flexibility** | Support new entity types (IoT, new AI layers) | Schema-less document storage + structured relational data |
| **Traceability** | Every action linked to verified user and timestamp | Comprehensive audit logging and version control |
| **Performance** | Fast spatial, text, and relational queries | Strategic indexing and caching layers |

---

## 🧠 Data Management Approach

### Multi-Database Strategy

```mermaid
graph TB
    A[Application Layer] --> B[PostgreSQL + PostGIS]
    A --> C[MongoDB]
    A --> D[Redis Cache]
    A --> E[File Storage]
    
    B --> F[Structured Data<br/>Users, Claims, Spatial]
    C --> G[Document Data<br/>OCR Results, AI Outputs]
    D --> H[Cache Layer<br/>Maps, Sessions, Hot Data]
    E --> I[File Storage<br/>Documents, Images, Exports]
```

| Layer | Purpose | Storage Technology | Data Examples |
|-------|---------|-------------------|---------------|
| **Structured Data** | Users, roles, spatial relationships, claims | PostgreSQL + PostGIS | User profiles, land parcels, village boundaries |
| **Document Data** | OCR text, AI/NER outputs, processing metadata | MongoDB | Extracted text, model confidence scores, version history |
| **Spatial Data** | Geographic features, maps, asset locations | PostGIS | FRA parcels, water bodies, forest cover polygons |
| **Cache Layer** | Frequently accessed data, session management | Redis | Map tiles, user sessions, hot queries |
| **File Storage** | Raw documents, images, export files | Cloud Storage | Scanned FRA documents, satellite imagery, reports |

---

## 🏗️ High-Level Architecture

```mermaid
flowchart TD
    A[Frontend: React + Leaflet] --> B[Node.js API Gateway]
    
    B --> C[Authentication Service]
    B --> D[Document Processing Service]
    B --> E[GIS & Mapping Service]
    B --> F[Decision Support Service]
    B --> G[AI Integration Service]
    
    C --> H[(PostgreSQL: Users & Roles)]
    D --> I[(MongoDB: Documents & OCR)]
    E --> J[(PostGIS: Spatial Data)]
    F --> K[(Analytics DB)]
    G --> L[(AI Models & Results)]
    
    J --> M[Bhuvan WMS Integration]
    L --> N[Satellite Data Feeds]
    
    H --> O[Redis Cache]
    I --> O
    J --> O
    
    style H fill:#e1f5fe
    style I fill:#f3e5f5
    style J fill:#e8f5e8
```

> 🎯 **Architecture Benefits:**
> - **Microservices Ready:** Each service can scale independently
> - **Data Isolation:** Sensitive user data separated from document processing
> - **AI Integration:** Clean interfaces for machine learning pipelines
> - **Real-time Capable:** WebSocket support for live updates

---

## 🔑 Core Entities

### Primary Data Entities

| Entity | Description | Key Fields | Storage |
|--------|-------------|------------|---------|
| **User** | System stakeholders and officials | `user_id, role, department, state, permissions` | PostgreSQL |
| **Document** | FRA files for OCR processing | `doc_id, file_path, status, upload_date, extracted_data` | MongoDB |
| **FRA Claim** | Individual/Community land claims | `claim_id, holder_name, village, status, geometry` | PostGIS |
| **Village** | Geographic boundary and metadata | `village_id, name, district, population, geometry` | PostGIS |
| **Asset** | AI-detected features (water, forests) | `asset_id, type, coordinates, confidence, source` | PostGIS + MongoDB |
| **AI Result** | OCR, NER, classification outputs | `result_id, model_name, confidence, input_data, output_data` | MongoDB |
| **Scheme** | Government scheme eligibility | `scheme_id, name, criteria, priority, benefits` | PostgreSQL |

### User Roles & Data Access Matrix

```mermaid
graph LR
    A[MOTA Admin] --> B[Full System Access]
    C[State Tribal Dept] --> D[State-level Data]
    E[District Officer] --> F[District Operations]
    G[Forest/Revenue] --> H[Spatial Data Management]
    I[NGO Worker] --> J[Limited View Access]
    
    style B fill:#4caf50
    style D fill:#8bc34a
    style F fill:#cddc39
    style H fill:#ffeb3b
    style J fill:#ff9800
```

| Role | Document Upload | Map Layers | Decision Support | Admin Features |
|------|----------------|------------|-----------------|---------------|
| **MOTA Admin** | ✅ Full Access | ✅ All Layers | ✅ Full Analytics | ✅ User Management |
| **State Tribal Dept** | ✅ State Level | ✅ State Layers | ✅ State Analytics | ⚠️ Limited |
| **District Officer** | ✅ District Level | ✅ District Layers | ✅ District Insights | ❌ None |
| **Forest/Revenue Dept** | ⚠️ Limited | ✅ Spatial Data | ✅ Environmental Data | ❌ None |
| **NGO Worker** | ❌ None | ✅ Basic View | ❌ None | ❌ None |

---

## 🧩 Entity Relationship Model

```mermaid
erDiagram
    USERS ||--o{ DOCUMENTS : uploads
    USERS ||--o{ CLAIM_VERIFICATIONS : verifies
    DOCUMENTS ||--o{ AI_EXTRACTIONS : processes_to
    VILLAGES ||--o{ FRA_CLAIMS : contains
    FRA_CLAIMS ||--o{ LAND_ASSETS : has
    FRA_CLAIMS ||--o{ SCHEME_ELIGIBILITY : qualifies_for
    AI_EXTRACTIONS ||--|| EXTRACTED_ENTITIES : produces
    LAND_ASSETS }o--|| SATELLITE_IMAGERY : detected_from

    USERS {
        string user_id PK
        string email UK
        string department
        string state
        string role
        json permissions
        timestamp created_at
    }
    
    DOCUMENTS {
        string doc_id PK
        string user_id FK
        string file_path
        string file_type
        string status
        json extracted_data
        timestamp upload_date
    }
    
    FRA_CLAIMS {
        string claim_id PK
        string village_id FK
        string holder_name
        string claim_type
        string status
        geometry polygon
        decimal area_hectares
    }
    
    VILLAGES {
        string village_id PK
        string village_name
        string district
        string state
        geometry boundary
        json census_data
    }
    
    LAND_ASSETS {
        string asset_id PK
        string claim_id FK
        string asset_type
        geometry location
        decimal confidence
        string detected_by
    }
    
    SCHEME_ELIGIBILITY {
        string eligibility_id PK
        string claim_id FK
        string scheme_name
        decimal score
        json criteria_met
        string priority
    }
```

> 🔗 **Relationship Highlights:**
> - **User-Centric:** All actions traceable to specific users
> - **Document Flow:** Clear pipeline from upload → OCR → verification → spatial mapping
> - **Spatial Integration:** Geographic relationships maintained in PostGIS
> - **AI Ready:** Structured storage for model inputs and outputs

---

## 🔄 Data Flow Lifecycle

### Complete Document-to-Decision Pipeline

```mermaid
flowchart LR
    A[📄 Document Upload] --> B[🔍 OCR Processing]
    B --> C[🏷️ NER Extraction]
    C --> D[✅ Data Verification]
    D --> E[🗺️ Spatial Mapping]
    E --> F[🤖 AI Asset Detection]
    F --> G[🎯 Scheme Matching]
    G --> H[📊 Decision Support]
    
    subgraph Storage_Layers
        I[(MongoDB<br/>Documents & OCR)]
        J[(PostGIS<br/>Spatial Data)]
        K[(PostgreSQL<br/>Structured Data)]
    end
    
    B --> I
    C --> I
    D --> K
    E --> J
    F --> J
    G --> K
```

### Detailed Sequence Flow

```mermaid
sequenceDiagram
    participant U as User (Dept Officer)
    participant F as Frontend
    participant B as Backend API
    participant M as MongoDB
    participant P as PostgreSQL
    participant G as PostGIS
    participant A as AI Service

    U->>F: Upload FRA Document
    F->>B: POST /documents/upload
    B->>M: Store document metadata
    B->>A: Trigger OCR + NER processing
    A->>M: Store extracted entities
    B->>F: Return extraction results
    
    U->>F: Verify extracted data
    F->>B: PUT /documents/verify
    B->>P: Create FRA claim record
    B->>G: Store spatial geometry
    B->>A: Trigger asset mapping
    A->>G: Update land assets
    
    B->>F: Confirm successful processing
    F->>U: Show verification complete
```

---

## 🧭 Access & Storage Strategy

### Role-Based Access Control (RBAC)

```javascript
// Example Permission Structure
{
  user: {
    id: "USR_MP_001",
    role: "STATE_OFFICER",
    department: "Tribal Welfare",
    state: "Madhya Pradesh",
    permissions: {
      documents: ["upload", "view", "verify"],
      maps: ["view_state", "filter_district"],
      analytics: ["view_state_reports"],
      users: ["view_team"]
    }
  }
}
```

### Data Partitioning Strategy

| Partition Type | Criteria | Benefit |
|---------------|----------|---------|
| **Geographic** | By State (MP, Odisha, etc.) | Faster spatial queries, localized management |
| **Temporal** | By upload year/month | Efficient archives and backups |
| **Departmental** | By user department | Data isolation and security |

---

## ⚙️ Scalability and Maintenance

### Scaling Strategies

```mermaid
graph TB
    A[Scaling Approach] --> B[Horizontal Scaling]
    A --> C[Vertical Scaling]
    A --> D[Performance Optimization]
    
    B --> B1[Database Sharding]
    B --> B2[Microservices]
    B --> B3[Load Balancing]
    
    C --> C1[Hardware Upgrades]
    C --> C2[Query Optimization]
    
    D --> D1[Caching Strategy]
    D --> D2[Index Optimization]
    D --> D3[Connection Pooling]
```

### Maintenance Operations

| Operation | Frequency | Automation | Recovery Time |
|-----------|-----------|------------|---------------|
| **Backups** | Daily (incremental) + Weekly (full) | ✅ Automated scripts | 2-4 hours |
| **Index Rebuild** | Monthly | ✅ Scheduled jobs | During maintenance window |
| **Data Archiving** | Quarterly | ✅ Automated | N/A |
| **Performance Review** | Monthly | ⚠️ Manual + Automated alerts | N/A |

---

## 🚀 Performance Optimization

### Indexing Strategy

```sql
-- Spatial indexes for GIS queries
CREATE INDEX idx_fra_parcels_geom ON fra_parcels USING GIST(geometry);
CREATE INDEX idx_village_boundaries_geom ON village_boundaries USING GIST(geometry);

-- Performance indexes for common queries
CREATE INDEX idx_fra_data_state_district ON extracted_fra_data(state, district);
CREATE INDEX idx_users_department_state ON users(department, state);
CREATE INDEX idx_documents_status_date ON fra_documents(status, upload_date);
CREATE INDEX idx_assets_village_type ON asset_mapping(village_id, asset_type);
```

### Caching Strategy

| Cache Type | Data Cached | TTL | Technology |
|------------|-------------|-----|------------|
| **Session Cache** | User sessions, permissions | 24 hours | Redis |
| **Map Tile Cache** | Rendered map tiles, WMS responses | 1 week | Redis + CDN |
| **Query Cache** | Frequent spatial queries, analytics | 1 hour | Redis |
| **Document Cache** | Recently processed documents | 4 hours | Redis |

---

## 🎯 Summary

### 🏆 Key Architecture Achievements

| Aspect | Achievement | Impact |
|--------|-------------|--------|
| **Data Model** | Hybrid relational + document + spatial | Best tool for each data type |
| **Scalability** | Cloud-native with horizontal scaling | Ready for national deployment |
| **Security** | RBAC with comprehensive audit trails | Meets government security standards |
| **AI Integration** | Structured pipelines for model outputs | Future-proof for new AI capabilities |
| **Performance** | Strategic indexing and caching | Responsive user experience |

### 🚀 Implementation Readiness

```mermaid
graph LR
    A[Phase 1: Core Setup] --> B[Phase 2: Data Migration]
    B --> C[Phase 3: AI Integration]
    C --> D[Phase 4: Optimization]
    
    style A fill:#4caf50
    style B fill:#8bc34a
    style C fill:#cddc39
    style D fill:#ffeb3b
```

**Phase 1 Deliverables:**
- ✅ PostgreSQL + PostGIS database setup
- ✅ MongoDB document store configuration  
- ✅ User management and RBAC system
- ✅ Basic document upload and storage

**Future Enhancements:**
- 🔄 Real-time satellite data integration
- 🔄 IoT sensor data streams
- 🔄 Advanced AI/ML model serving
- 🔄 Mobile data collection interfaces

---

## 📞 Database Support & Contacts

For database-specific queries or schema discussions, the development team can contact:

**State Coordination:**
- Madhya Pradesh: `dirtadp@mp.gov.in`
- Odisha: `stscdev@gmail.com`
- Tripura: `twdtripura@gmail.com`
- Telangana: `secretary_tw@telangana.gov.in`

> 📋 **Note:** This database organization provides a **production-ready foundation** that balances flexibility with structure, enabling rapid development while maintaining data integrity and security across all FRA management operations.

---

<div align="center">

**🏛️ Ministry of Tribal Affairs**  
**🌳 Forest Rights Act Management System**  
**📅 Database Architecture Document**

</div>
