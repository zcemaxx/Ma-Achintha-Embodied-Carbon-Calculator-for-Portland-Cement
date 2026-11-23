# Embodied Carbon Calculator for Portland Cement

The calculator was designed by Ruiyi and Dr Achintha, from the Faculty of Science and Engineering, University of Manchester.   
The code uses the Python standard library only. It runs in Jupyter or from a terminal.
<br>

## Background

Building and construction industry is a major contributor to global GHG emissions. Decarbonisation efforts have focused on operational carbon, while embodied carbon has received less attention. To balance the trade-off between carbon savings and costs, this study develops a parametric model for quantifying cradle-to-gate embodied carbon, initially applied to cement as one of the most widely used and carbon-intensive construction materials.

Existing databases and tools often provide generalised emission factors without fully disclosing their assumptions, system boundaries, data sources, or calculation processes. This study addresses this gap by offering a transparent and replicable methodology for calculating and customising emission factors under defined assumptions and project contexts.

Flexibility is central to the design methodology. The model allows emission factors to be customized based on project-specific parameter. When measured values are unavailable, pressing 'Enter' automatically substitutes a literature-referenced default value, ensuring that even a partial dataset can generate a complete and traceable result.

<br>

## Table of Contents
- [Calculation Scope](#calculation-scope)
- [Methodology](#methodology)
  - [A1 – Raw Material Supply](#a1--raw-material-supply)
  - [A2 – Transportation](#a2--transportation)
  - [A3 – Manufacturing](#a3--manufacturing)
- [Worked Example](#worked-example) 
- [Default Parameters](#default-parameters)
- [Emission Factors and Sources](#Emission-factors-and-sources)
- [References](#references)
- [How to Run](#how-to-run)

<br>

## Calculation Scope
### System Boundary

This calculator covers cradle-to-gate embodied carbon, which includes:

| Stage | Description | Included |
|---|---|---|
| A1 | Raw material supply: quarrying, crushing and screening, purchased gypsum and SCM | ✅ |
| A2 | Transportation (of quarried rock, gypsum, SCM and kiln fuel to the plant) | ✅ |
| A3 | Manufacturing (calcination, kiln fuel combustion, electricity) | ✅ |
| A4 | Transport to site | ❌ |
| A5 | Installation | ❌ |

### Functional Unit

All results are expressed in **kgCO₂e per tonne of finished cement**.

For consistency, emissions expressed per unit mass of cement produced are converted to functional unit, such as per 1 tonne of CEM I cement.
  
### Cement Types
  
To classify different cements, EN 197-1:2011 [1] defined 27 distinct common cement types by specific proportions of clinker and supplementary cementitious materials (SCMs). These 27 cement types are grouped into five main categories, namely CEM I to CEM V, based on their clinker contents.

| Type | Clinker Ratio |
|---|---| 
| Ordinary Portland Cement (CEM I) | 95% |  
Portland Composite Cement (CEM II) | 65–94% |
Blast Furnace Cement (CEM III) | 5–64% |
Pozzolanic Cement (CEM IV) | 45–89% |
Composite Cement (CEM V) | 20–64% | 

<br>

## Methodology

Emissions are calculated using the following general formula:

```
Total EC = ∑ (Activity data or material quantity × Emission factor)   
EC_total = EC_A1 + EC_A2 + EC_A3 (kgCO2e/tonne cement) 
```

Each life cycle stage breaks down emission sources as follows:  

![Figure 1](https://github.com/zcemaxx/Cement-embodied-carbon-calculator/blob/main/Figures/Full%20cement%20manufacturing%20process.jpg)


### A1 – Raw Material Supply

A1 covers quarrying (A1.1), crushing and screening (A1.2), and purchased materials (A1.3). Emission sources include diesel combustion and ANFO blasting in quarrying, and electricity consumption in crushing & screening.

#### Clinker chemistry and the material balance

The raw material demand is derived from clinker chemistry rather than assumed.

Portland cement clinker is primarily composed of four mineral phases: tricalcium silicate (C₃S), dicalcium silicate (C₂S), tricalcium aluminate (C₃A), and tetracalcium aluminoferrite (C₄AF). Industry report shows clinker composition varies between narrow limits according to raw-material composition and manufacturing conditions [2].

The user enters the clinker phase composition (C₃S, C₂S, C₃A, C₄AF, % of clinker, normalised to 100). The CaO mass fraction of each phase is derived from molecular weights, and the CaO content of clinker is the phase-weighted sum.  

```
CaO fraction of phase = (n x M_CaO) / M_phase   
CaO in clinker (%)    = ∑ (phase share (%) x CaO fraction of phase)
```  

For example, in tricalcium silicate (3CaO·SiO₂, molecular weight = 228)
```
CaO weight = (56 × 3) / 228 = 0.74
```

The default clinker phase proportions are derived from the mean Bogue phase composition of ASTM C150 Type I Portland cement in the 2014 US survey [3] in Table 3: C₃S 57.5%, C₂S 13.0%, C₃A 9.3% and C₄AF 7.6% by mass of cement.    

These sum to 87.4% because the Bogue calculation is applied to the oxide analysis of the finished cement, which also contains gypsum, free lime and other minor constituents.    
The calculator takes proportions of clinker, so the four values are normalised to 100%, giving 65.8%, 14.9%, 10.6% and 8.7%.   

| Phase | Formula | Default proportion | CaO fraction |
|---|---|---|---|
| C3S – alite | 3CaO·SiO₂ | 65.8% | 0.7368 |
| C2S – belite | 2CaO·SiO₂ | 14.9% | 0.6512 |
| C3A – aluminate | 3CaO·Al₂O₃ | 10.6% | 0.6226 |
| C4AF – ferrite | 4CaO·Al₂O₃·Fe₂O₃ | 8.7% | 0.4616 |
| Gypsum | CaSO₄·2H₂O | 0 | 0.00 |

Gypsum does not appear in this table. It is interground after the kiln and is not a clinker phase, so it is accounted separately as 5% of the finished cement. Including it here would understate the clinker phases and, through them, the calcination CO₂.

Limestone demand is then back-calculated through the CaCO₃/CaO stoichiometric ratio (100/56 = 1.785) and the quarried mineral purity:
```
CaCO₃     = cement mass × clinker% × CaO in clinker% × 1.785
Limestone = CaCO₃ required / mineral purity
```
  
Non-carbonate raw materials include clay, sand, iron ore, which are entered for overall material balance:
```
Clay, sand, iron ore = clinker share x t per t clinker
Quarried rock        = limestone + clay + sand + iron ore
```
   
#### A1.1 – Quarrying

Emissions arise from mobile plant diesel and from blasting, both applied to the total quarried rock:

```
CO₂_diesel   = quarried rock (t) × diesel (L/t) × EF_diesel (kgCO₂/L)
CO₂_blasting = quarried rock (t) × ANFO (kg/t) × EF_ANFO (kgCO₂/kg)
```

#### A1.2 – Crushing & Screening

Crushing energy is derived from equipment throughput and motor rating, so the specific energy follows from the plant's own machines:

```
Specific energy (kWh/t) = Σ [P_i (kW) / throughput_i (t/h)]
CO₂_crushing = quarried rock (t) × specific energy × EF_electricity (kgCO₂/kWh)
```
where EF_electricity = 0.233 kgCO₂/kWh is applied in the UK.

The number of crusher stages is entered by the user, then throughput and power for each. Defaults values of equipment:

| Equipment | Power (kW) | tonne/hr |
|---|---|---|
| Primary (jaw) crusher | 220 | 600 |
| Secondary (impact) crusher | 150 | 400 |
| Screening unit | 50 – 150 | 5 |

#### A1.3 – Purchased Materials

Gypsum and any SCM are bought in rather than quarried by the plant, so the emissions are calculated as:
```
CO₂_Gypsum   = mass (kg) x supply EF
```
   
#### Total A1
Total A1 emissions are the sum of extraction and crushing & screening, in formula of:
```
EC_A1 = EC_A1.1 + EC_A1.2 + EC_A1.3
```

<br>

### A2 – Transportation

A2 covers four legs. Each takes its own mode and one-way distance.

| Leg | From | To | Default mode | Default distance |
|---|---|---|---|---|
| 1 | Quarry | Cement plant | Truck | 50 km |
| 2 | Gypsum supplier | Cement plant | Truck | 200 km |
| 3 | SCM supplier | Cement plant | Electric rail | 300 km |
| 4 | Fuel supplier | Cement plant | Diesel rail | 300 km |

Emissions are calculated with general formula, for truck, electric rail, diesel rail:
```
CO₂_transport = distance (km) × mass (tonne) × EF_transport (kgCO₂/tonne·km)
```

For conveyor belt:
```
CO₂_conveyor = P (kW) × t (hrs) × EF_electricity (kgCO₂/kWh)
```

<br>

### A3 – Manufacturing

A3 covers three emission sources: kiln fuel combustion (A3.1), electricity consumption (A3.2), and calcination (A3.3).

#### A3.1 – Kiln Fuel Combustion

Heat required to raise raw meal to kiln temperature (~1200°C) is calculated with:

```
Q = m × c × ΔT
```
Where:      
  `m` = mass of raw meal (kg)    
  `c` = 0.84 kJ/kg·°C (specific heat capacity of cement raw meal, fixed constant)       
  `ΔT` = kiln temperature − ambient temperature (°C)     

The default heat required is 3,000 MJ per tonne of clinker, within the 3.0–3.4 GJ/t range typical of modern dry-process kilns with preheater and precalciner.

The fuel consumed and CO₂ produced from the heat generated:
```
Fuel consumed (kg) = Q / calorific value of fuel (KJ/kg)
CO₂_combustion = fuel consumed (kg) × EF_fuel (kgCO₂/kg)
```
    
#### A3.2 – Electricity Consumption

Electricity consumed by each piece of equipment is calculated separately and then summed up:

```
CO₂_electricity = Σ [P_i (kW) × t_i (hrs/tonne)] × EF_electricity
```

Reference values of equipment user inputs:

| Equipment | Default (kWh per tonne cement) |
|---|---|
| Raw mill | 25 |
| Kiln drive and preheater fans | 25 |
| Clinker cooler fans | 5 |
| Cement (finish) mill | 40 |
| Conveyors, pumps and packing | 8 |
| **Total** | **103** |

This total sits in the 90–120 kWh/t range reported for modern plants. The finish mill is the single largest electrical load and must not be omitted.

`EF_electricity` defaults to 0.233 kgCO₂e/kWh (UK grid, DESNZ 2023) and is editable, so non-UK or contracted-supply scenarios can be modelled.

#### A3.3 – Calcination

CO₂ released from limestone decomposition (CaCO₃ → CaO + CO₂):

```
CO₂_calcination = CaCO₃ required (tonne) × (44/100) × 1000
```

The mass entering this equation is pure CaCO₃, not the as-quarried limestone. Limestone at 95% purity contains 5% non-carbonate material that does not calcine; using the quarried mass would overstate this term by the reciprocal of the purity.

<br>

## Worked Example

This worked example runs the notebook with the default inputs and gives the results below, CEM I, 1 tonne of cement:

| Line item	| kgCO₂e |
| --- | --- |
| A1.1 quarrying, mobile plant diesel | 3.94 |
| A1.1 quarrying, blasting (ANFO) | 0.09 |
| A1.2 crushing and screening | 0.19 |
| A1.3 gypsum supply | 2.65 |
| A2 raw materials, quarry to plant | 9.09 |
| A2 gypsum, supplier to plant | 1.24 |
| A2 fuel, supplier to plant | 0.88 |
| A3.1 kiln combustion, coal (70%) | 188.67 |
| A3.1 kiln combustion, natural gas (30%) | 47.92 |
| A3.2 raw mill | 4.28 |
| A3.2 kiln drive and preheater fans | 4.28 |
| A3.2 clinker cooler fans | 0.86 |
| A3.2 cement (finish) mill | 6.84 |
| A3.2 conveyors, pumps and packing | 1.37 |
| A3.3 calcination of CaCO₃ | 512.96 |
| **Total** | **785.24** |

| Stage | kgCO₂e/t | Share |
|---|---|---|
| A1 | 6.87 | 0.9% |
| A2 | 11.20 | 1.4% |
| A3 | 767.17 | 97.7% |
| **Total A1–A3** | **785.24** | **100%** |

<br>

## Default Parameters

`list_params()` prints the full set with current values listed below:

| Group | Parameter | Default |
|---|---|---|
| Composition | Clinker / gypsum / SCM | 95 / 5 / 0 % |
| Clinker phases | C₃S / C₂S / C₃A / C₄AF | 65.8 / 14.9 / 10.6 / 8.7 % |
| Raw materials | Limestone purity | 95% CaCO₃ |
| | Clay, sand, iron ore | 0.20, 0.03, 0.02 t per t clinker |
| Quarrying | Diesel | 1.0 L per t rock |
| | ANFO | 0.35 kg per t rock |
| Crushing | Stage 1 | 600 t/h, 220 kW |
| | Stage 2 | 400 t/h, 150 kW |
| Transport | Raw materials | truck, 50 km |
| | Gypsum | truck, 200 km |
| | SCM | rail (diesel), 300 km |
| | Fuel | rail (diesel), 300 km |
| Kiln | Specific heat consumption | 3,000 MJ per t clinker |
| | Fuel mix | coal 70%, natural gas 30% |
| Electricity (kWh per t cement) | Raw mill / kiln drive and preheater fans / cooler fans / finish mill / conveyors, pumps and packing | 25 / 25 / 5 / 40 / 8 |
| Grid | Emission factor | 0.17109 kgCO₂e/kWh |
      
The kiln heat, fuel mix, electricity and transport distance defaults are modelling assumptions for a modern dry-process plant. They are inputs, and plant-specific data should replace them where available.    

<br>

## Emission factors and sources
 
- The emission factors used by the model from different sources are listed below.  

| Item | Value | Source |
|---|---|---|
| Grid electricity, UK, 2025 | 0.17109 kgCO₂e/kWh | DESNZ (2026), Table 8, electricity consumed (includes transmission and distribution losses), excluding imports |
| Diesel, quarry plant | 2.69 kgCO₂/L | EIA (2024), distillate fuel oil, 10.19 kg/gal |
| ANFO | 0.18 kgCO₂/kg | Stoichiometry for 94% ammonium nitrate and 6% fuel oil (0.188), used as 0.18 |
| Gypsum, supply | 53.0 kgCO₂e/t | Saint-Gobain Formula EPD S-P-10284, GWP-fossil, A1 to A3 |
| GGBS, supply | 70 kgCO₂e/t | Arup (2023), upper end of the 42 to 70 range for A1 to A3 |
| Truck, artic under 34 t | 0.124 kgCO₂e/t·km | GLEC Framework v3.2, well-to-wheel, incl. empty running, return legs not counted separately |
| Rail, diesel | 0.0307 kgCO₂e/t·km | GLEC Framework v3.2, well-to-wheel |
| Rail, electric | 0.0095 kgCO₂e/t·km | GLEC Framework v3.2, well-to-wheel |
| Calcination | 44.01 / 100.09 | Molecular weight ratio of CO₂ to CaCO₃ |
| Clinker phases | 65.8 / 14.9 / 10.6 / 8.7 % | PCA (2016), Table 3, ASTM C150 Type I means normalised to 100% |
| Kiln specific heat consumption | 3,000 MJ/t clinker | Typical modern dry kiln with preheater/precalciner |
           
The indicative CO₂e emission factor for ground granulated blast-furnace slag (GGBS), is 155 kgCO₂e/t, based on the 2025 version of MPA Fact Sheet 18 [4]. GGBS used to be treated as a by-product of steelmaking, with little or no emissions allocated to it. Its reclassification as a co-product requires a share of steelmaking emissions to be allocated based on economic value, increasing its reported emission factor from 70 to 155 kgCO₂e/t [5]. Replace it with your own supplier value where one exists.
         
- Kiln fuel factors, derived from IPCC (2006) Volume 2. Fuel is entered as a mix, given as percentage shares of kiln heat that are normalised to 100%. This allows co-firing scenarios rather than forcing a single fuel. Net Calorific values (NCV) and emission factors are listed below:  
         
| Fuel | EF (kg CO₂/TJ) | NCV (MJ/kg) | EF (kgCO₂/kg) |
|---|---|---|---|
| Coal (other bituminous) | 94,600 | 25.8 | 2.44 |
| Petroleum coke | 97,500 | 32.5 | 3.17 |
| Natural gas | 56,100 | 48.0 | 2.69 |
| Fuel oil (residual) | 77,400 | 40.4 | 3.13 |
| Diesel (gas/diesel oil) | 74,100 | 43.0 | 3.19 |
         
The 2019 Refinement to the IPCC Guidelines leaves these defaults unchanged [6], so the 2006 values remain current.
     
- For different cement types, all three fractions remain editable and are validated to sum to 100%.
     
| Type | Clinker | Gypsum | SCM |
|---|---|---|---|
| CEM I | 95% | 5% | – |
| CEM II/B-S | 65-94% | 5% | 25% GGBS |
| CEM III/A | 45% | 5% | 50% GGBS |
      
<br>  
   
## References

[1] ‘BS EN 197-1:2011’. Available: https://knowledge.bsigroup.com/products/cement-composition-specifications-and-conformity-criteria-for-common-cements  
[2] Christidis, G. E., Dimitriadi, M., Triantafyllou, G., & Tsoumeleas, C. (2021). Quantitative Analysis of Portland Cement Clinker with Rietveld Refinement: Implications of the Amorphous Matter. Materials Proceedings, 5(1), 75. https://doi.org/10.3390/materproc2021005075  
[3] Paul D. Tennis, ‘Chemical and Physical Characteristics of US Hydraulic Cements:  2014’. Available: https://www.cement.org/wp-content/uploads/2024/09/SN3284.pdf    
[4] MPA (2025). *Fact Sheet 18, Embodied CO₂e of UK cements.*   
[5] Astle P, (2026). Available from: https://www.linkedin.com/posts/paul-astle-ramboll_decarbonisation-lowercarbonconcrete-share-7467207404255035392-giTo/    
[6] A. Garg and M. M. Weitz, ‘2019 Refinement to the 2006 IPCC Guidelines for National Greenhouse Gas Inventories’, Volume 2: Energy | Chapter 2: Stationary Combustion. Available: https://www.ipccnggip.iges.or.jp/public/2019rf/pdf/2_Volume2/19R_V2_2_Ch02_Stationary_Combustion.pdf       
- IPCC (2006). *2006 IPCC Guidelines for National Greenhouse Gas Inventories, Volume 3: Industrial Processes and Product Use.* IPCC.
- DESNZ (2023). *UK Government GHG Conversion Factors for Company Reporting.* Department for Energy Security and Net Zero.
- GLEC Framework (2019). *Global Logistics Emissions Council Framework for Logistics Emissions Accounting and Reporting.* Smart Freight Centre.
- EN 15804:2012+A2:2019. *Sustainability of Construction Works — Environmental Product Declarations.* 
   
<br>
   
## How to Run 

1. Clone the repository:
```
git clone https://github.com/zcemaxx/Ma-Achintha-Embodied-Carbon-Calculator-for-Portland-Cement.git
```

2. Open Cement_Embodied_Calculator.ipynb in Jupyter or VS Code, and run the cells in order. In the last cell, call  
```
inp = main()
```
  
3. Answer the prompts. Press Enter, or type `na`, to accept the default shown in brackets.   
`validate()` runs inside `calculate()` and raises a `ValueError` for shares that do not sum to 100, unknown fuels or transport modes, and negative values.
  
4. Answer the prompts. Press Enter, or type na, to accept the default shown in brackets. Every question shows its default and its permitted range.

<br>

## Licence

Released under the MIT Licence — see [LICENSE](LICENSE). You are free to use, modify and redistribute this calculator, including commercially, provided the copyright notice is retained.

The emission factors and calorific values are published by third parties (DESNZ, IPCC, GLEC, and the sources listed under [References](#references)) and are not covered by this licence. They are reproduced as factual reference data; please cite the original sources when reusing them.
