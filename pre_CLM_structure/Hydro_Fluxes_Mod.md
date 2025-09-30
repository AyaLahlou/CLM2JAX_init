---
config:
  layout: elk
---
flowchart TD
    %% --- 样式定义 ---
    classDef main_style fill:#cde4ff,stroke:#5b97d4,color:#000
    classDef biogeophys_style fill:#d5f5e3,stroke:#58d68d,color:#000

    %% --- 模块与数据类型定义 ---
    subgraph "Core Modules"
        CHM["CanopyHydrologyMod"]
        SNHM["SnowHydrologyMod"]
        SHoM["SoilHydrologyMod"]
        SWM["SurfaceWaterMod"]
        HHM["HillslopeHydrologyMod"]
        LHM["LakeHydrologyMod"]
        SRM["SurfaceRadiationMod"]
        SAM["SurfaceAlbedoMod"]
        CFM["CanopyFluxesMod"]
        PSM["PhotosynthesisMod"]
        BGM["BareGroundFluxesMod"]
        STM["SoilTemperatureMod"]
    end
    subgraph "Shared Data Types"
        A2L["atm2lndType"]
        ST["SoilStateType"]
        CST["CanopyStateType"]
        LST["LakeStateType"]
        TT["TemperatureType"]
        LT["LandunitType"]
        WFB["WaterFluxBulkType"]
        WDB["WaterDiagnosticBulkType"]
        WSB["WaterStateBulkType"]
        EF["EnergyFluxType"]
        SA["SolarAbsorbedType"]
        SALB["SurfaceAlbedoType"]
        SCFB["SnowCoverFractionBaseMod"]
        CIAT["CanopyInterceptionAndThroughfall"]
        SWR["SoilWaterRetentionCurve"]
        SWMov["SoilWaterMovementMod"]
        IERM["InfiltrationExcessRunoffMod"]
        SERM["SaturatedExcessRunoffMod"]
        PP["Photosynthesis_Products"]
        CVNS["CNVegnitrogenstateType"]
    end

    %% --- 数据连接 ---
    A2L --> SAM & SRM & PSM & CHM & SNHM
    CST --> SAM & CFM & PSM & CHM
    LST --> SAM
    SAM --> SALB
    SALB --> SRM
    SRM --> SA
    SA --> CFM & BGM & PSM
    ST --> CFM & BGM & PSM & STM & SHoM & HHM
    WFB --> CFM & SWM
    TT --> CFM & BGM & PSM & CHM & SNHM & SHoM
    PSM --> CFM & PP
    CFM --> EF & WFB
    BGM --> EF
    CVNS --> PSM
    EF --> STM & SHoM
    STM --> TT
    CHM --> CIAT & WFB
    SCFB --> SNHM
    SNHM --> WFB
    CIAT --> SWM
    SWM --> SHoM
    LT --> SHoM
    SWR --> SHoM
    SWMov --> SHoM
    SHoM --> ST & WFB
    IERM --> SWM
    SERM --> SWM
    HHM --> WFB

    %% --- 样式应用 ---
    %% 应用蓝色样式到 'main' 文件夹的类型
    class A2L,LT main_style
    
    %% 应用绿色样式到 'biogeophysics' 文件夹的模块和类型
    class CHM,SNHM,SHoM,SWM,HHM,LHM,SRM,SAM,CFM,PSM,BGM,STM,ST,CST,LST,TT,WFB,WDB,WSB,EF,SA,SALB,SCFB,CIAT,SWR,SWMov,IERM,SERM,PP,CVNS biogeophys_style