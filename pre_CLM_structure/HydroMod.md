---
config:
  layout: elk
---
flowchart TD
 subgraph subGraph0[Modules]
        CHM["CanopyHydrologyMod"]
        SNHM["SnowHydrologyMod"]
        SHoM["SoilHydrologyMod"]
        SWM["SurfaceWaterMod"]
        HHM["HillslopeHydrologyMod"]
        LHM["LakeHydrologyMod"]
        PSM["PhotosynthesisMod"]
        A["AerosolMod"]
  end
 subgraph subGraph1[Shared Data Types]
        A2L["atm2lndType"]
        ST["SoilStateType"]
        CST["CanopyStateType"]
        LST["LakeStateType"]
        WFB["WaterFluxBulkType"]
        WDB["WaterDiagnosticBulkType"]
        WSB["WaterStateBulkType"]
        EF["EnergyFluxType"]
        TT["TemperatureType"]
        LT["LandunitType"]
        AM["Aerosol_Type_or_Masses"]
        PP["Photosynthesis_Products"]
        IERM["InfiltrationExcessRunoffMod"]
        SERM["SaturatedExcessRunoffMod"]
        SCFB["SnowCoverFractionBaseMod"]
        DR["Drainage_Result"]
        CIAT["CanopyInterceptionAndThroughfall"]
        SWR["SoilWaterRetentionCurve"]
        SWMov["SoilWaterMovementMod"]
  end
    A2L --> CHM & SNHM & LHM
    AM --> CHM & SNHM & A
    CST --> CHM & PSM
    TT --> CHM & SNHM & SHoM & PSM
    CHM --> WFB & CIAT
    SCFB --> SNHM
    SNHM --> WFB
    CIAT --> SWM
    WFB --> SWM & IERM & SERM & PSM & A
    SWM --> SHoM
    ST --> SHoM & IERM & SERM & HHM & LHM & PSM
    EF --> SHoM & LHM
    LT --> SHoM & PSM
    SWR --> SHoM
    SWMov --> SHoM
    SHoM --> ST & WFB & DR
    IERM --> SWM
    SERM --> SWM
    HHM --> WFB
    LST --> LHM
    LHM --> WFB
    PSM --> PP & WFB

