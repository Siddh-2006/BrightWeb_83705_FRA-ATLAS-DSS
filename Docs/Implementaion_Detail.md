
# 🔄 Tribal Data Management System Flowchart

## 👤 User Journey & Feature Access with Upload & OCR + Interactive Map
```mermaid
flowchart TD
    %% Login / Signup Flow
    A[No Home Page] --> B[Click on Login]
    
    B --> C{User Exists?}
    C -->|Yes| D[Login Page]
    C -->|No| E[Signup Page]
    
    D --> F{Forgot Password?}
    F -->|Yes| G[OTP Verification]
    G --> H[Change Password]
    H --> D
    F -->|No| I[Enter Credentials]
    I --> J{Valid Credentials?}
    J -->|No| K[Show Error]
    K --> D
    J -->|Yes| L[Login Successful]
    
    E --> M[Fill Registration Details]
    M --> N[OTP Verification]
    N --> O{Valid OTP?}
    O -->|No| P[Show Error]
    P --> M
    O -->|Yes| Q[Create New User]
    Q --> D

    L --> R[Redirect to Home Page]

    %% Home Page & Feature Access
    R --> S[Select Feature]
    
    S --> T1[Upload & OCR]
    S --> T2[Interactive Map]
    S --> T3[Decision Support]
    S --> T4[Chatbot]

    %% User Access Control
    T1 --> U1{"User Type?"}
    U1 -->|"MOTA"| V1["Access Granted"]
    U1 -->|"DAJGUA"| V1
    U1 -->|"Forest / Revenue"| W1["Access Denied"]
    U1 -->|"Planning & Dev Auth"| W1
    U1 -->|"NGO"| W1

    T2 --> V2["Access Granted for All Users"]
    T4 --> V4["Access Granted for All Users"]

    T3 --> U3{"User Type?"}
    U3 -->|"Forest / Revenue"| V3["Access Granted"]
    U3 -->|"Planning & Dev Auth"| V3
    U3 -->|"Others"| W3["Access Denied"]

    %% Upload & OCR Workflow
    V1 --> X1[Upload FRA Document]
    X1 --> X2{File Uploaded Successfully?}
    X2 -->|No| X1
    X2 -->|Yes| X3[Process Document with AI]
    X3 --> X4[Extract Text from Document]
    X4 --> X5[Preview Extracted Text]
    X5 --> X6{Edit Required?}
    X6 -->|Yes| X7[User Edits Text]
    X6 -->|No| X8[Proceed]
    X7 --> X8
    X8 --> X9[Verify Document]
    X9 --> X10[Save Verified Document to Database]

    %% Interactive Map Workflow (Filters Parallel)
    V2 --> Y1[Load Map Interface]
    Y1 --> Y2[Display Data Layers by Scheme]
    
    %% Parallel Filters
    Y2 --> Y3a[Filter by Type of Land]
    Y2 --> Y3b[Filter by State]
    Y3a --> Y4[View Land Holdings]
    Y3b --> Y4
    Y4 --> Y5[Hover on Parcel to See Owner Details]
