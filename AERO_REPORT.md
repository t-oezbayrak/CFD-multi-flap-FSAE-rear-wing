# Aerodynamic Analysis: Multi-Element Rear Wing CFD Study

**Author:** Turan — M.Sc. Materials Engineering (Structural Integrity), RWTH Aachen University  
**Date:** June 2026  
**Platform:** SimScale (Community Plan) — OpenFOAM Backend  
**Status:** Complete

---

## Executive Summary

This module documents a steady-state CFD analysis of a Formula Student–class rear wing configuration consisting of a Selig S1223 main element and a NACA 6412 flap. The study was conducted in SimScale using the incompressible RANS solver with k-ω SST turbulence modeling at a freestream velocity of 15 m/s (Re ≈ 300,000 based on main element chord). The converged solution yields a lift coefficient of Cl = 3.46 and a drag coefficient of Cd = 2.06, consistent with a high-lift, maximum-downforce configuration typical of FSAE rear wings.

---

## 1. Wing Configuration and Profile Selection

### 1.1 Design Philosophy

The rear wing of an FSAE vehicle is designed for maximum downforce, not maximum aerodynamic efficiency (L/D). Unlike aircraft wings where minimizing drag is critical, a race car rear wing tolerates high drag in exchange for greater mechanical grip through increased tire normal force. This philosophy directly informed the profile and flap angle selection.

### 1.2 Profile Selection Rationale

**Main Element — Selig S1223:**

The S1223 is a high-lift, low-Reynolds-number airfoil developed by Michael Selig at the University of Illinois. It is one of the most widely used profiles in FSAE rear wings due to its exceptionally high maximum lift coefficient (Cl,max ≈ 2.2 at Re = 200,000). The profile features extreme camber (~17.8% max camber at 42% chord) and a distinctive reflexed lower surface near the leading edge.

Key characteristics at Re = 200,000–300,000:
- Cl,max ≈ 2.2 (single element, clean)
- High camber produces strong pressure differential
- Well-documented experimental data available (UIUC Low-Speed Airfoil Tests)
- Coordinates sourced from the UIUC Airfoil Coordinates Database

**Flap — NACA 6412:**

The NACA 6412 was selected as the flap element. It is a 4-digit series airfoil with 6% camber at 40% chord and 12% maximum thickness. Its moderate thickness provides structural stiffness at the flap scale, and its camber supplements the overall circulation of the multi-element system.

### 1.3 Geometric Configuration

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| Main element profile | Selig S1223 | High Cl, FSAE standard |
| Main element chord | 300 mm | Scaled for FSAE regulations |
| Flap profile | NACA 6412 | Moderate camber, good structural thickness |
| Flap chord | 100 mm (projected ~87 mm at 30°) | Typical flap/main ratio 0.3–0.4 |
| Flap deflection angle | 30° | Aggressive high-lift setting |
| Flap position | 5 mm overlap, 8 mm gap below main TE | Slot flow between elements |
| Wing span (extrusion) | 300 mm | Sufficient for 2.5D analysis |
| Total configuration width | ~332 mm | Main chord + flap projection |

The 30° flap deflection represents an aggressive, maximum-downforce setting. In practice, flap angles of 15°–25° would provide better L/D ratios, but the current configuration prioritizes peak downforce consistent with FSAE philosophy.

---

## 2. CAD Geometry Generation

### 2.1 Workflow

Profile coordinates were obtained from the UIUC Airfoil Coordinates Database (S1223) and generated analytically using the NACA 4-digit formula (NACA 6412). The modeling workflow:

1. **Coordinate acquisition:** S1223 .dat file from airfoiltools.com; NACA 6412 generated from the standard 4-digit thickness and camber equations
2. **Scaling and formatting:** Coordinates converted to mm-scale CSV with X, Y, Z columns
3. **Fusion 360 import:** Profiles imported as sketch splines via CSV point import
4. **3D extrusion:** Each profile extruded 300 mm along the Z-axis to create solid bodies
5. **STEP export:** Both bodies exported as a single STEP assembly for SimScale import

### 2.2 NACA 4-Digit Generation

The NACA 6412 coordinates were generated using the standard NACA 4-digit series equations:

**Camber line:**

For x < p: yc = (m/p²) × (2px − x²)

For x ≥ p: yc = (m/(1−p)²) × ((1 − 2p) + 2px − x²)

Where m = 0.06 (max camber), p = 0.40 (max camber position).

**Thickness distribution:**

yt = 5t × (0.2969√x − 0.1260x − 0.3516x² + 0.2843x³ − 0.1015x⁴)

Where t = 0.12 (max thickness ratio).

Upper and lower surface coordinates are computed by projecting the thickness perpendicular to the camber line. Cosine spacing was used to concentrate points near the leading edge where curvature is highest.

---

## 3. CFD Simulation Setup

### 3.1 Solver and Turbulence Model

| Setting | Selection | Justification |
|---------|-----------|---------------|
| Solver type | Incompressible (SIMPLE) | Ma = V/a = 15/343 = 0.044 ≪ 0.3 |
| Turbulence model | k-ω SST | Industry standard for external aero; handles adverse pressure gradients and boundary layer separation better than k-ε |
| Steady/Transient | Steady-state | Time-averaged forces sufficient for performance estimation |
| Backend | OpenFOAM (via SimScale) | Open-source, validated, industry-accepted |

**Why k-ω SST:**

The k-ω SST (Shear Stress Transport) model, developed by Menter (1994), combines the strengths of k-ω near walls and k-ε in the freestream. For airfoil flows at Re ~ 300,000, it provides accurate prediction of boundary layer development, transition behavior, and separation onset — all critical for high-camber profiles like the S1223 where trailing edge separation is expected.

### 3.2 Flow Domain

The flow domain was created using SimScale's CAD mode by constructing a rectangular bounding box around the wing assembly and performing a Boolean subtraction of the wing bodies. The resulting volume represents the air domain.

| Dimension | Min | Max | Size | Reasoning |
|-----------|-----|-----|------|-----------|
| X (streamwise) | −5c = −1500 mm | +10c = +3000 mm | 4500 mm | 10c downstream for wake dissipation |
| Y (vertical) | −5c = −1500 mm | +5c = +1500 mm | 3000 mm | Far-field boundary effect minimization |
| Z (spanwise) | Wing span boundaries | — | 300 mm | Matches wing span for 2.5D enforcing |

Domain sizing follows standard practice: upstream distance of at least 5 chord lengths to allow flow development, and downstream distance of at least 10 chord lengths to capture wake behavior without outlet boundary interference.

### 3.3 Boundary Conditions

| Boundary | Type | Value | Physical Meaning |
|----------|------|-------|------------------|
| Inlet (−X face) | Velocity inlet | Ux = 15 m/s, Uy = Uz = 0 | Freestream approach |
| Outlet (+X face) | Pressure outlet | p = 0 Pa (gauge) | Far-field static pressure |
| Top and bottom (±Y faces) | Slip wall | — | Free-stream, no boundary layer |
| Side faces (±Z faces) | Symmetry | — | Enforces 2.5D (infinite span) |
| Wing surfaces | No-slip wall | — | Viscous boundary condition |

**Freestream conditions:**
- Velocity: V∞ = 15 m/s (54 km/h) — representative FSAE cornering speed
- Density: ρ = 1.2 kg/m³ (ISA at 20°C)
- Dynamic viscosity: μ = 1.8 × 10⁻⁵ Pa·s
- Reynolds number: Re = ρV∞c/μ = (1.2 × 15 × 0.3) / 1.8×10⁻⁵ ≈ 300,000

### 3.4 Mesh Strategy

SimScale's automatic hex-dominant mesh generator was used with moderate global fineness and local surface refinement on the wing surfaces (refinement level 2–3). This concentrates cells in regions of high velocity and pressure gradients — the boundary layer, leading edge, trailing edge, and the slot between main element and flap.

---

## 4. Results

### 4.1 Convergence

The simulation was run to steady-state convergence. Force monitors (lift, drag, side force) were tracked throughout the solution process. All three force components reached plateau values by approximately iteration 200 and remained stable through the end of the simulation (1000+ iterations), confirming adequate convergence.

### 4.2 Force Results

Forces were extracted from the converged solution using the Forces and Moments result control applied to all wing surfaces.

| Component | Pressure Force | Viscous Force | Total Force |
|-----------|---------------|---------------|-------------|
| X (Drag) | ~23 N | ~2 N | **~25 N** |
| Y (Lift) | ~41 N | ~1 N | **~42 N** |
| Z (Side) | ~0 N | ~0 N | **~0 N** |

The negligible Z-force confirms the symmetry boundary conditions are working correctly and the 2.5D assumption holds.

### 4.3 Aerodynamic Coefficients

Coefficients are computed using the following reference values:

| Reference | Value |
|-----------|-------|
| Dynamic pressure, q | ½ρV² = 0.5 × 1.2 × 15² = **135 Pa** |
| Reference area, A | chord × span = 0.3 × 0.3 = **0.09 m²** |

**Results:**

| Coefficient | Value | Context |
|-------------|-------|---------|
| **Cl** | **3.46** | S1223 literature (single element): ~2.2. Flap adds ~1.2 additional lift |
| **Cd** | **2.06** | High, consistent with 30° flap deflection |
| **L/D** | **1.68** | Low efficiency, but this is a max-downforce config |

### 4.4 Flow Field Analysis

**Velocity field (Velocity Z contour):**
The spanwise velocity component is near-zero across the domain, confirming the 2.5D nature of the simulation. Minor spanwise velocity appears in the slot region between main element and flap, indicating three-dimensional slot flow effects.

**Pressure field (Total Pressure contour):**
- Leading edge stagnation region shows highest total pressure (~170 Pa), as expected at the flow attachment point
- Upper (suction) surface of the main element shows a strong low-pressure region, extending from approximately 10% chord to 60% chord — this is the primary lift-generating mechanism
- The slot between main element and flap shows pressure recovery, confirming the multi-element slot effect: high-energy freestream air energizes the boundary layer on the flap upper surface
- Wake region behind the trailing edge shows total pressure deficit, indicating momentum loss (drag)

**Turbulent Kinetic Energy (TKE):**
- Low TKE in the freestream (laminar approach flow)
- TKE production concentrated in the wake region behind the main element trailing edge and the flap
- The boundary layer transition region on the upper surface shows moderate TKE increase, consistent with laminar-to-turbulent transition at this Reynolds number
- Domain boundaries show elevated TKE, which is a known artifact of inlet turbulence boundary condition specification

### 4.5 Physical Interpretation

The S1223 + NACA 6412 configuration at 30° flap deflection produces a Cl of 3.46, which is approximately 57% higher than the S1223 alone (Cl ≈ 2.2). This lift augmentation is due to two multi-element effects:

1. **Circulation enhancement:** The flap increases the effective camber of the overall system, increasing circulation around the main element
2. **Slot effect:** The gap between main element and flap allows high-energy freestream air to re-energize the boundary layer on the flap, delaying separation

The high Cd of 2.06 is the cost of the aggressive 30° flap deflection. At this angle, significant pressure drag is generated by the large projected frontal area of the deflected flap, and flow separation is likely occurring on the flap upper surface near the trailing edge.

---

## 5. Critical Assessment and Limitations

### 5.1 Mesh Sensitivity

A formal mesh independence study was not conducted due to computational budget constraints (SimScale Community plan). The results should be considered indicative rather than grid-converged. A production-level analysis would require at least three mesh densities with Richardson extrapolation to quantify numerical uncertainty.

### 5.2 Turbulence Modeling

The k-ω SST model, while industry-standard, has known limitations at Re ~ 300,000:
- Transition prediction is approximate (no explicit transition model like γ-Reθ)
- Separation prediction on highly cambered surfaces may underpredict separation extent
- Reynolds stresses in the wake region are modeled, not resolved

### 5.3 2.5D Assumption

The symmetry boundary conditions on the Z-faces enforce an infinite-span assumption, neglecting tip vortex effects. A real FSAE rear wing has finite span (~1.0–1.2 m) with endplates. Tip effects would reduce effective lift and induce additional drag. The Cl and Cd values reported here represent the 2D sectional limit — actual 3D values would be lower.

### 5.4 Potential Next Steps

1. **Flap angle sweep:** Run 15°, 20°, 25°, 30° to find the optimal L/D vs. maximum Cl tradeoff
2. **Mesh convergence study:** Three mesh levels to quantify numerical uncertainty
3. **3D simulation:** Full-span wing with endplates to capture finite-span effects
4. **Experimental validation:** Compare Cl against UIUC published data for S1223 at matching Re

---

## 6. Tools and References

### Tools

| Tool | Purpose |
|------|---------|
| Fusion 360 (Education) | CAD geometry generation, profile import, STEP export |
| SimScale (Community) | CFD solver (OpenFOAM backend), meshing, post-processing |
| Python | Airfoil coordinate generation (NACA 4-digit formula), data processing |
| UIUC Airfoil Database | S1223 profile coordinates |

### Key References

- Selig, M.S., Guglielmo, J.J., Broeren, A.P., and Giguère, P., "Summary of Low-Speed Airfoil Data," Vol. 1, SoarTech Publications, 1995.
- Menter, F.R., "Two-Equation Eddy-Viscosity Turbulence Models for Engineering Applications," AIAA Journal, Vol. 32, No. 8, 1994, pp. 1598–1605.
- Zerihan, J. and Zhang, X., "Aerodynamics of a Single-Element Wing in Ground Effect," Journal of Aircraft, Vol. 37, No. 6, 2000.

---

*This aerodynamic study is Phase 2 of the end-to-end engineering case study. Phase 1 (Structural — Pushrod Rocker FEA) is documented separately.*
