graph TD
    subgraph "Core Data Definitions"
        WST[WaterStateType]
        WFT[WaterFluxType]
        WDT[WaterDiagnosticType]
    end

    subgraph "Extended / Bulk Definitions"
        style WSBT fill:#D6EAF8,stroke:#5DADE2
        style WFBT fill:#D6EAF8,stroke:#5DADE2
        style WDBT fill:#D6EAF8,stroke:#5DADE2
        
        WSBT[WaterStateBulkType]
        WFBT[WaterFluxBulkType]
        WDBT[WaterDiagnosticBulkType]
    end

    subgraph "Metadata / Info Classes"
        style WIBT fill:#FEF9E7,stroke:#F4D03F
        WIBT[WaterInfoBaseType]
        WIBkT[WaterInfoBulkType]
        WITT[WaterInfoTracerType]
        WIIT[WaterInfoIsotopeType]
    end
    
    subgraph "High-Level Containers"
        style WT fill:#E8DAEF,stroke:#8E44AD
        WT[WaterType]
        WTCT[WaterTracerContainerType]
    end

    subgraph "Utilities"
        style WTU fill:#D5F5E3,stroke:#58D68D
        WTU[WaterTracerUtils]
    end

    %% --- Relationships ---
    %% Inheritance / Extension
    WST -- "is extended by" --> WSBT
    WFT -- "is extended by" --> WFBT
    WDT -- "is extended by" --> WDBT

    %% Metadata Inheritance
    WIBT --> WIBkT
    WIBT --> WITT
    WITT --> WIIT

    %% Composition / Containment (Arrow points to the container)
    WSBT -- "is part of" --> WT
    WFBT -- "is part of" --> WT
    WDBT -- "is part of" --> WT
    WIBkT -- "is part of" --> WT
    WITT -- "is part of" --> WT

    %% Usage Relationship (Dashed line)
    WTCT -. "uses" .-> WTU
    WST -. "uses" .-> WTCT
    WFT -. "uses" .-> WTCT