---
config:
  layout: elk
---
flowchart TD
 subgraph subGraph0["Energy & Carbon Modules"]
        SRM["SurfaceRadiationMod"]
        SAM["SurfaceAlbedoMod"]
        CFM["CanopyFluxesMod"]
        PSM["PhotosynthesisMod"]
        BGM["BareGroundFluxesMod"]
        STM["SoilTemperatureMod"]
  end
 subgraph subGraph1["Shared Data Types"]
        A2L["atm2lndType"]
        ST["SoilStateType"]
        CST["CanopyStateType"]
        WFB["WaterFluxBulkType"]
        EF["EnergyFluxType"]
        TT["TemperatureType"]
        SA["SolarAbsorbedType"]
        SALB["SurfaceAlbedoType"]
        PP["Photosynthesis_Products"]
  end
    A2L --> SAM & SRM & PSM
    CST --> SAM & CFM & PSM
    SAM --> SALB
    SALB --> SRM
    SRM --> SA
    SA --> CFM & BGM & PSM
    ST --> CFM & BGM & PSM & STM
    WFB --> CFM
    TT --> CFM & BGM & PSM
    PSM --> CFM & PP
    CFM --> EF & WFB
    BGM --> EF
    EF --> STM
    STM --> TT
