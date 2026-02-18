# `model400.py` 
```mermaid
flowchart TD
    A((Precipitation)) -->B{Temperature > Thres ?}
    B -->|No|C[Snow Storage]
    C -->D((Snowmelt))
    D -->SS[Static Storage]
    B -->|Yes|F((Rainfall))
    F-->SS
    SS-->ET((Evapotranspiration))
    SS-->SF[Surface Storage]
    SF-->SFLOW((Surface Flow))
    SF-->I((Infiltration))
    I-->US[Upper Soil]
    US-->PERC((Percolation))
    US-->IF((Interflow))
    PERC-->LS[Lower Soil]
    LS-->BF((Base Flow))
    SFLOW-->R((Total Runoff))
    IF-->R
    BF-->R


```
## Description
**`model400.py`**  
Is an implementation of the TETIS model structure (Frances et al 2007), (Quintero & Velasquez 2022).

Francés, F., Vélez, J. I., & Vélez, J. J. (2007). Split-parameter structure for the automatic calibration of distributed hydrological models. Journal of Hydrology, 332(1-2), 226-240.

Quintero, F., & Velásquez, N. (2022). Implementation of TETIS hydrologic model into the hillslope link model framework. Water, 14(17), 2610.

- Uses two forcings: precipitation and evapotranspiration
- Includes snow processes
- Uses constant infiltration and percolation rates
- Estimates three flow components: surface flow, interflow and baseflow
- Tile drainage flow can be represented via interflow
- This formulation includes routing across the river network created by the hillslopes

---

## Inputs

- **Forcings :**
  - **Precipitation** – rainfall rate applied to each hillslope, in mm/hour  
  - **Evapotranspiration** – evapotranspiration rate, in mm/month.  

- **Hillslope / channel properties:**
  - **Hillslope area** – contributing surface area draining into the channel, in square kilometers.
  - **Channel accumulated drainage area** – total upstream area contributing to a channel link, in square kilometers.  
  - **Channel length** – physical length of the channel link, in kilometers.  

- **Initial conditions:**
  - Discharge in the channel [m3/s]  
  - SWE in snow storage [m]
  - Water stored in the static storage [m]
  - Water stored in the soil surface [m]
  - Water stored in the upper layer of soil [m]
  - Water stored in the bottom layer of soil [m] 
  

- **Parameter and default values**

    - Channel reference velocity $v_r$=0.3\,m/s
    - Exponent of channel velocity discharge $\lambda_1$ =0.3  (dimensionless)
    - Exponent of channel velocity area $\lambda_2$ =-0.1  (dimensionless)
    - Maximum static storage $H_{u}$ = 100 (mm)
    - Temperature threshold $T = 0 (\degree C) $ 
    - Melting factor $MF = 5 (mm / \degree C / day)$
    - Velocity of water on the hillslope surface $v_h=0.1m/s$
    - Infiltration rate $ir=3(mm/hr)$
    - Percolation rate  $pr=2(mm/hr)$
    - Residence time in upper soil layer $rtus = 10 (day)$
    - Residence time in lower soil layer $rtus = 100 (day)$

    Other parameters used internally in the formulation are
    - Reference area $Ar=1km^2$
    - Reference discharge $q_r=1m^3/s$ 

  

---

## Outputs

- Discharge in the channel [m3/s]  
- Water ponded in the surface [m]
- Water stored in the upper layer of soil [m]
- Water stored in the bottom layer of soil [m] 
- Accumulated precipitation [m]
- Accumulated runoff [m]
- Baseflow [m3/s]

---

## Equations
In the formulation equations $L$ is the length of the channel, $A_h$ is the area of the hillslope. $s_p$ is the water stored in the ponds, $s_t$ is the water stored in the top layer of soil. $s_s$ is the water stored in the soil subsurface. $q$ is the discharge in the channel. 

**Surface Soil:**
$$
\frac{ds_p}{dt}=p-q_{pc}-q_{pt}-e_p
$$
$p$ is the precipitation in the hillslope


$q_{pc}$ is the flux of water ponded on the surface to the channel and is defined by $q_pc = k_2s_p$, where $k_2=v_h(L/A_h) ×10^{-3}[1/min]$

 $q_{pt}$ is the flux of water ponded on the surface to the top layer storage and is defined by $q_{pt} = k_t\,s_p$, where $k_t=k_2 [A+B(1-s_t/s_L )^\alpha ][1/min]$

$e_p$ is the evapotranspiration in the surface of soil

**Top soil layer:**
$$
\frac{ds_t}{dt}=q_{pt}-q_{ts}-e_t
$$
$q_{ts}$ is the flux of water from the top layer storage to the subsurface is defined by $q_{ts} = k_i\,s_t$ , where $k_i=k_2\beta$

$e_t$ is the evapotranspiration in the top layer of soil


**Subsurface layer:**
   
$$
\frac{ds_s}{dt}=q_{ts}-q_{sc}-e_s
$$

 $q_{sc}$ is the flux of water from the subsurface to the channel is defined by $q_{sc} = k_3\,s_s$   

**Nonlinear channel routing:**

The mass transport equation for each channel link in the network is given by
$$
   \frac{dq}{dt} =L\,\frac{v_{r}}{1-\lambda_1}\,\frac{q}{q_r}^{\lambda_1}\frac{A}{A_r}^{\lambda_2}\,[-q+(q_{pc}+q_{sc})\frac{A_h}{60}+q_{in}] 
$$

where $q_{in}$ is the flux from upstream channels.
 
  
**Appendix**

Fluxes representing evaporation are given by:

$c =(s_p)+(s_t/s_L)+(s_s/(h_b-s_L))$

$e_p=e_{pot}(s_p)/c$

$e_t=e_{pot}(s_t/s_L)/c$

$e_s=e_{pot}(s_s/(h_b-s_L))/c$



---

## Dependencies

- Python ≥3.8  
- `numpy`  
- Internal `ifc_hlm` modules  

---

## Example Usage

```python
from ifc_hlm.models import model254

# Define parameters (see lines 62–73 for parameter set)
params = {
    "hillslope_area": [...],   # km²
    "channel_area": [...],     # km²
    "channel_length": [...],   # km
    "infiltration": {...},
    "routing": {...},
    "initial_state": {...}
}

# Initialize model
hlm = model254.HLMModel(params)

# Forcings in mm/hour
precip = [2.5, 5.0, 0.0, ...]   # precipitation time series
et     = [0.1, 0.1, 0.2, ...]   # evapotranspiration series

# Run model (dt = 1.0 hour)
results = hlm.run(precip, et, dt=1.0)

Qout = results["discharge"]
```


