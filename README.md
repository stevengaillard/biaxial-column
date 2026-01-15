# RC Column 3D Interaction - Biaxial Bending Capacity

**[English](#) | [繁體中文](./README.zh-TW.md)**

## Overview

**RC Column 3D Interaction** is a comprehensive web-based structural engineering application designed to analyze and verify reinforced concrete columns under biaxial bending and axial loads according to **Taiwan Building Code (112)** (closely aligned with ACI 318). Developed to provide immediate visual feedback, this tool generates 3D interaction failure surfaces to verify capacity ratios (D/C) for arbitrary sections.

## Core Features

### Calculation Engine

* **Biaxial Interaction**: Exact integration of stress-strain compatibility for 3D P-M-M surfaces
* **Arbitrary Rebar Layouts**: Support for regular grids or custom coordinate-based reinforcement
* **Shear Design**: Automatic calculation of capacities (X and Y directions)
* **P-Delta Effects**: Moment magnification factors for slender columns
* **Seismic Provisions**: Special Moment Frame checks and confinement requirements

### User Interface

* **Dual Language Support**: Instant toggling between English and Traditional Chinese (繁體中文)
* **Real-time Visualization**: 3D Interaction Surface (Three.js) and 2D Section rendering
* **Interactive Graphs**: Live P-M curve generation based on input changes
* **Cloud Synchronization**: Firebase-powered user history and design storage
* **PDF Report Generation**: Professional calculation summary via native browser printing
* **Dark Mode**: High-contrast interface for low-light environments

### Advanced Features

* **Custom Rebar Import**: Batch paste coordinates (`x, y, db`) from Excel
* **Batch Load Processing**: Import multiple load cases (`Pu, Mux, Muy`) simultaneously
* **Code Warnings**: Real-time validation for rebar ratios, spacing, and tie detailing
* **Load Visualization**: 3D plotting of applied load points relative to the capacity surface

---

## Design Code Compliance

### Taiwan Building Code (112) / ACI 318 Alignment

| Component | Description | Methodology |
| --- | --- | --- |
| **Axial** | Pure Compression | *φP*<sub>n</sub> (Strength Reduction) |
| **Flexure** | Biaxial Bending | Strain Compatibility (*ε*<sub>cu</sub> = 0.003) |
| **Shear** | Shear Strength | *V*<sub>c</sub> + *V*<sub>s</sub> ≤ *φV*<sub>n</sub> |
| **Slenderness** | Moment Magnification | Moment magnification factor (*δ*) |
| **Seismic** | Ductile Detailing | Confinement checks for Special Moment Frames |

---

## Calculation Methods

### 1. Axial & Flexural Strength (P-M Interaction)

The nominal strength is calculated using the **strain compatibility method**, assuming a linear strain distribution across the section.

#### Concrete Compression (Whitney Stress Block)

**Equivalent Rectangular Stress Block:**

$$\text{Stress} = 0.85f'_c$$

$$\text{Depth} = \beta_1 c$$

where ***β*<sub>1</sub>** is determined by ***f'*<sub>c</sub>**:

* If ***f'*<sub>c</sub> ≤ 280 kgf/cm²**: 

$$\beta_1 = 0.85$$

* If ***f'*<sub>c</sub> > 280 kgf/cm²**: 

$$\beta_1 = \max\left(0.65, 0.85 - 0.05\frac{f'_c - 280}{70}\right)$$

#### Steel Stress

Steel stress is calculated based on strain compatibility:

$$f_s = \varepsilon_s \times E_s$$

where:
- ***ε*<sub>s</sub>** = steel strain at distance from neutral axis
- ***E*<sub>s</sub>** = 2.04 × 10⁶ kgf/cm² (modulus of elasticity)

Subject to yielding constraint:

$$-f_y \leq f_s \leq f_y$$

#### Nominal Capacity

Integration is performed by slicing the cross-section into **30×30 fibers** and rotating the neutral axis through **72 angles** (5° increments) to determine:

$$P_n = \sum F_c + \sum F_s$$

$$M_{nx} = \sum(F_c \cdot y_i) + \sum(F_s \cdot y_i)$$

$$M_{ny} = \sum(F_c \cdot x_i) + \sum(F_s \cdot x_i)$$

where:
- ***F*<sub>c</sub>** = concrete fiber force
- ***F*<sub>s</sub>** = steel bar force
- ***x*<sub>i</sub>, *y*<sub>i</sub>** = coordinates of fiber/bar

**Check:** $P_u \leq \phi P_n$, $M_{ux} \leq \phi M_{nx}$, $M_{uy} \leq \phi M_{ny}$

---

### 2. Strength Reduction Factors (*φ*)

The design strength is ***φP*<sub>n</sub>** and ***φM*<sub>n</sub>**. The factor ***φ*** varies based on the net tensile strain (***ε*<sub>t</sub>**) in the extreme tension steel.

#### Tied Columns:

| Strain Condition | Net Tensile Strain (*ε*<sub>t</sub>) | *φ* Value |
| --- | --- | --- |
| Compression-Controlled | *ε*<sub>t</sub> ≤ *ε*<sub>y</sub> | 0.65 |
| Transition Region | *ε*<sub>y</sub> < *ε*<sub>t</sub> < 0.005 | Linear Interpolation |
| Tension-Controlled | *ε*<sub>t</sub> ≥ 0.005 | 0.90 |

**Transition Zone Interpolation:**

$$\phi = 0.65 + 0.25 \times \frac{\varepsilon_t - \varepsilon_y}{0.005 - \varepsilon_y}$$

where:
- ***ε*<sub>y</sub>** = *f*<sub>y</sub> / *E*<sub>s</sub> (yield strain)

*Note: For pure compression check, maximum axial load is limited to **0.80*φ*<sub>compression</sub>*P*<sub>o</sub>**.*

**Nominal Pure Compression Capacity:**

$$P_o = 0.85f'_c(A_g - A_{st}) + f_y A_{st}$$

where:
- ***A*<sub>g</sub>** = gross concrete area
- ***A*<sub>st</sub>** = total steel area

---

### 3. Shear Strength

#### Concrete Capacity (*V*<sub>c</sub>):

$$V_c = \left[0.53\sqrt{f'_c} + \frac{P_u}{6A_g}\right] \times \frac{b_w \times d}{1000}$$

where:
- ***b*<sub>w</sub>** = width of section (cm)
- ***d*** = effective depth (cm)
- ***P*<sub>u</sub>** = factored axial load (kgf)
- ***A*<sub>g</sub>** = gross concrete area (cm²)

*(Note: *V*<sub>c</sub> = 0 for seismic regions if *P*<sub>u</sub> < *A*<sub>g</sub>*f'*<sub>c</sub>/20)*

#### Steel Capacity (*V*<sub>s</sub>):

$$V_s = \frac{A_v \times f_y \times d}{s \times 1000}$$

Subject to maximum limit:

$$V_s \leq \frac{2.12\sqrt{f'_c} \times b_w \times d}{1000}$$

where:
- ***A*<sub>v</sub>** = area of shear reinforcement = *n*<sub>legs</sub> × *A*<sub>bar</sub> (cm²)
- ***s*** = tie spacing (cm)

#### Design Capacity:

$$\phi V_n = \phi(V_c + V_s)$$

where ***φ*** = 0.75 (shear reduction factor)

**Check:** $V_u \leq \phi V_n$

---

### 4. Slenderness Effects

#### Unbraced Length Check (*L*<sub>u</sub>):

Slenderness is checked against the radius of gyration:

$$r = 0.3h$$

Slenderness criterion:

$$\frac{kL_u}{r} > 22 \rightarrow \text{Column is Slender}$$

where:
- ***k*** = effective length factor (assumed 1.0 for unbraced)
- ***L*<sub>u</sub>*** = unbraced length (m)
- ***h*** = section depth (cm)

#### Moment Magnification:

If slender, moments are magnified:

$$\delta = \frac{1}{1 - \frac{P_u}{0.75P_c}} \geq 1.0$$

$$M_{\text{magnified}} = \delta \times M_{\text{original}}$$

where ***P*<sub>c</sub>** (critical buckling load):

$$P_c = \frac{\pi^2 \times 0.4E_c I_g}{(kL_u)^2}$$

with:
- ***E*<sub>c</sub>** = 15000√*f'*<sub>c</sub> (kgf/cm²)
- ***I*<sub>g</sub>** = gross moment of inertia (cm⁴)

**For X-Direction (about Y-axis):**

$$I_{gx} = \frac{bh^3}{12}$$

**For Y-Direction (about X-axis):**

$$I_{gy} = \frac{hb^3}{12}$$

---

## Material Limits & Warnings

### Concrete & Steel

* **Concrete (*f'*<sub>c</sub>):** Min 140 kgf/cm²
* **Steel (*f*<sub>y</sub>):** Min 2800 kgf/cm²

### Geometric Checks

#### Rebar Ratio (*ρ*):

$$\rho = \frac{A_{st}}{A_g}$$

where:
- ***A*<sub>st</sub>** = total steel area (cm²)
- ***A*<sub>g</sub>** = gross concrete area (cm²)

**Code Limits:**
- **Minimum:** *ρ* ≥ 0.01 (1%)
- **Maximum:** *ρ* ≤ 0.08 (8%)
- **Seismic Practical Limit:** *ρ* ≤ 0.04 (4%)

#### Tie Spacing:

**Gravity Design:**

$$s_{\max} = \min(16d_b, 48d_{\text{tie}}, \text{least dimension})$$

where:
- ***d*<sub>b</sub>** = main bar diameter (cm)
- ***d*<sub>tie</sub>** = tie bar diameter (cm)

**Seismic Design:**

$$s_{\max} = \min\left(\frac{d}{4}, 6d_b, 15 \text{ cm}\right)$$

where ***d*** = effective depth (cm)

#### Seismic Confinement (*A*<sub>sh</sub>):

$$A_{sh} \geq \max\left[0.3\frac{sb_c f'_c}{f_{yt}}\left(\frac{A_g}{A_{ch}} - 1\right), 0.09\frac{sb_c f'_c}{f_{yt}}\right]$$

where:
- ***s*** = tie spacing (cm)
- ***b*<sub>c</sub>** = core dimension (cm)
- ***A*<sub>ch</sub>** = core area measured to outside of ties (cm²)
- ***f*<sub>yt</sub>** = tie yield strength (kgf/cm²)

---

## Technology Stack

* **Frontend:** HTML5, Tailwind CSS, JavaScript (ES6+)
* **Visualization:** Three.js r128 (WebGL 3D), HTML5 Canvas (2D)
* **Backend:** Firebase 9.22.0 (Authentication + Firestore)
* **Icons:** Font Awesome 6.4.0
* **Math Engine:** Custom fiber integration solver implemented in JS

---

## Units Convention

| Parameter | Unit | Description |
| --- | --- | --- |
| **Force (*P*, *V*)** | tf | Ton-force (Metric Ton, 1 tf = 1000 kgf) |
| **Moment (*M*)** | tf-m | Ton-force meter |
| **Length** | cm / m | Cross-section (cm), Unbraced Length (m) |
| **Stress** | kgf/cm² | Concrete/Steel Strength |
| **Area** | cm² | Cross-sectional area |

---

## Safety Notes

⚠️ **Critical Considerations:**

1. **Professional Review Required**: All calculations must be verified by licensed professional engineers.
2. **Local Code Compliance**: While based on Taiwan Code 112/ACI 318, users must verify applicability to local jurisdiction.
3. **P-Delta Limits**: The current implementation approximates moment magnification using the moment magnifier method; second-order analysis may be required for very slender structures.
4. **Shear Direction**: Shear checks assume independent orthogonal action; biaxial shear interaction is not explicitly considered.
5. **Units**: Ensure consistency when inputting custom coordinates (cm) and loads (tf, tf-m).
6. **Material Properties**: Verify actual material properties meet code minimum requirements.

---

## Browser Compatibility

* Chrome/Edge (v90+)
* Firefox (v88+)
* Safari (v14+)
* Mobile: iOS Safari, Chrome Mobile (Touch controls enabled for 3D view)

---

## References

* **Taiwan Building Code**: Concrete Design Specifications (Version 112)
* **ACI 318-19**: Building Code Requirements for Structural Concrete
* **CRSI Design Handbook**: Interaction Diagrams & Tables

---

## Support

**Email:** gacchuguts@gmail.com

---

**Disclaimer**: This software is provided as a calculation aid for professional engineers. The user assumes all responsibility for the accuracy and appropriateness of input data and the interpretation of results. Always verify calculations independently and ensure compliance with applicable building codes and standards.