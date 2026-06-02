# Pushrod Rocker Bell-Crank: End-to-End Engineering Case Study

**Author:** Turan — M.Sc. Materials Engineering (Structural Integrity), RWTH Aachen University  
**Date:** June 2026  
**Status:** Phase 1 Complete (Structural) · Phase 2 In Progress (Aerodynamic)

---

## Executive Summary

This project documents the complete engineering workflow for a Formula Student–class pushrod suspension rocker (bell-crank), from concept through parametric CAD, finite element validation, and iterative weight optimization. The goal is to demonstrate practical, industry-relevant engineering decision-making — not academic theory — through a single, well-executed component.

The rocker was designed parametrically in Fusion 360, analyzed under a cornering + bump load scenario derived from FSAE vehicle dynamics, and iteratively optimized by removing material from low-stress regions identified by FEA. Maximum Von Mises stress remained at 35.6 MPa against a 275 MPa yield strength (Al 6061-T6), confirming a safety factor of 7.7 while achieving measurable weight reduction.

---

## 1. Subsystem Selection Rationale

### Why a Suspension Rocker?

The rocker (bell-crank) was selected over alternative subsystems (front wing mounts, DRS actuation brackets) based on the following criteria:

| Criterion | Rocker | Aero Bracket | DRS Bracket |
|-----------|--------|--------------|-------------|
| Load path clarity | Fully traceable via FBD | Requires CFD-derived loads | Niche, hard to validate |
| FEA suitability | Bending + torsion combination | Thin-plate dominated | Limited stress complexity |
| Manufacturability scope | Clevis, bearing seat, tolerances | Simple geometry | Limited |
| Portfolio narrative | Strong — links to FSAE experience | Visual but structurally thin | Differentiated but obscure |
| Aerospace transferability | High — same as linkage/bracket design | Moderate | Low |

**Decision:** The rocker provides the richest combination of structural complexity, manufacturing considerations, and defensible load derivation.

### Load Path Overview

The force chain through the inboard suspension:

```
Wheel Contact Patch (Fy + Fz)
    → Upright
        → Pushrod (axial compression)
            → Rocker (bending + torsion at pivot)
                → Damper/Spring (compression)
                → Chassis Mount (reaction force)
```

The rocker is the mechanical amplifier in this chain — it converts pushrod travel into damper travel via a lever ratio, while transmitting all forces through its pivot bearing.

---

## 2. Parametric CAD Design

### 2.1 Design Philosophy

All geometry is driven by a parametric model — no hard-coded dimensions. This enables single-point design changes and supports iterative optimization without rebuilding geometry.

**CAD Platform:** Autodesk Fusion 360 (Education License)

### 2.2 Parameter Table

| Parameter | Value | Expression | Purpose |
|-----------|-------|------------|---------|
| `rocker_ratio` | 1.33 | — | Motion ratio (arm_B / arm_A) |
| `arm_A` | 80 mm | — | Pushrod-side arm length |
| `arm_B` | 106.4 mm | `arm_A × rocker_ratio` | Damper-side arm (auto-calculated) |
| `arm_angle` | 100° | — | Included angle between arms |
| `pivot_bore` | 12 mm | — | Pivot bearing seat diameter |
| `wall_t` | 4 mm | — | Minimum wall thickness |
| `web_t` | 3 mm | — | Internal rib thickness |
| `fillet_r` | 3 mm | — | General fillet radius |
| `clevis_gap` | 10 mm | — | Fork gap for rod end clearance |
| `bolt_d` | 6 mm | — | M6 bolt nominal diameter |
| `bolt_clearance` | 6.5 mm | `bolt_d + 0.5` | Clearance hole with tolerance |

**Key design decisions:**
- `rocker_ratio` drives `arm_B` automatically — changing the motion ratio updates the entire geometry.
- `bolt_clearance` is derived from `bolt_d` — switching from M6 to M8 requires changing one parameter.
- `wall_t` and `web_t` are separated to allow independent tuning of outer shell vs. internal ribs during FEA iteration.

### 2.3 Geometry Features

**Bell-crank profile:** Two arms extending from a central pivot at 100°, creating the characteristic V-shape. The included angle falls within the typical FSAE range of 90°–120°.

**Clevis fork ends:** Both pushrod and damper attachment points feature a fork (clevis) geometry — two parallel walls with a gap between them. This provides:
- Physical clearance for rod end bearings
- Double-shear load transfer through the bolt
- A realistic mounting interface

**Pivot bore:** Solid central section with a 12 mm bearing seat. No clevis here — the pivot requires maximum stiffness and a press-fit bearing interface.

**Fillets:** 3 mm radius at arm-to-pivot transitions (stress concentration mitigation), 1 mm on general edges (manufacturing and layer adhesion for 3D printing).

### 2.4 Section Analysis

A cross-section view (YZ plane) was used to verify:
- Consistent wall thickness around clevis pockets
- Adequate material at pivot-to-arm transitions
- No unintended thin sections from intersecting features

---

## 3. Structural Analysis (FEA)

### 3.1 Load Derivation

Forces were derived from FSAE vehicle dynamics, not assumed arbitrarily. The full calculation chain:

**Vehicle Parameters:**
| Parameter | Value | Source |
|-----------|-------|--------|
| Total mass (with driver) | 300 kg | FSAE typical |
| Rear axle load (50/50) | 150 kg | Assumed |
| Per-wheel mass | 75 kg | 150 / 2 |
| Cornering acceleration | 1.5 g | FSAE typical |
| Bump acceleration | 2.0 g | Curb strike scenario |
| Pushrod angle from horizontal | 35° | Geometric assumption |
| Safety factor | 1.5 | Static analysis standard |

**Calculation:**

```
Step 1 — Wheel vertical force:
    F_z = 75 kg × 2.0 g × 9.81 m/s² = 1472 N

Step 2 — Pushrod axial force (geometry):
    F_pushrod = F_z / sin(35°) = 1472 / 0.574 = 2564 N

Step 3 — Design load (with safety factor):
    F_design = 2564 × 1.5 = 3846 N

Step 4 — Damper reaction (moment balance about pivot):
    F_damper = F_pushrod × (arm_A / arm_B) = 3846 × (80 / 106.4) = 2892 N
```

### 3.2 FEA Setup

| Setting | Value |
|---------|-------|
| Software | Fusion 360 Simulation (Static Stress) |
| Material | Aluminum 6061-T6 (σ_yield = 275 MPa) |
| Fixed constraint | Pivot bore inner cylindrical surface |
| Load 1 (Pushrod) | 3846 N, axial along arm_A toward pivot |
| Load 2 (Damper) | 2892 N, axial along arm_B toward pivot |
| Mesh type | Tetrahedral (auto) |
| Mesh refinement | Local refinement at fillet transitions and bore edges |

**Boundary condition rationale:** Both forces point toward the pivot because the pushrod pushes inward (compression member) and the damper reacts against the rocker (Newton's third law). This creates a bending moment at the elbow — the primary structural loading of the rocker.

### 3.3 Results — Baseline (Solid Geometry)

| Metric | Value |
|--------|-------|
| Max Von Mises stress | 35.64 MPa |
| Location of max stress | Clevis fork walls at bolt holes |
| Safety factor (σ_yield / σ_max) | 7.72 |
| Mass | 790 g |

**Observations:**
- Peak stress concentrated at clevis ends — thin walls carrying bearing loads from bolt holes.
- Arm bodies and elbow region showed low stress (< 10 MPa), indicating over-designed cross-sections.
- Pivot region showed near-zero stress — confirming the fixed constraint is absorbing reaction loads as expected.

**Interpretation:** Safety factor of 7.7 is far above the motorsport target of 1.5–2.5. This means significant material can be removed from low-stress regions without compromising structural integrity.

### 3.4 Results — Optimized Geometry (Weight-Reduced)

Based on the baseline stress distribution, arc-shaped pockets were introduced on the flat faces of both arms — targeting the blue (low-stress) regions. Clevis ends were left untouched due to their higher stress state.

| Metric | Baseline | Optimized | Change |
|--------|----------|-----------|--------|
| Max Von Mises stress | 35.64 MPa | 35.63 MPa | −0.03% |
| Safety factor | 7.72 | 7.72 | No change |
| Mass | 790 g | 779 g | −1.4% |

**Key finding:** Peak stress did not increase after material removal. This validates that the removed material was structurally non-contributing — the load path was already bypassing those regions. The optimization was data-driven, not arbitrary.

**Portfolio takeaway:** _"Iterative FEA-driven weight optimization: identified low-stress regions from baseline analysis, removed material via surface pockets, and confirmed zero stress penalty through re-analysis."_

---

## 4. Engineering Decisions Log

This section documents the reasoning behind key design choices — the "why", not just the "what".

| Decision | Options Considered | Choice | Rationale |
|----------|-------------------|--------|-----------|
| Subsystem | Rocker / Aero bracket / DRS bracket | Rocker | Best FEA complexity, load traceability, FSAE relevance |
| CAD platform | Fusion 360 / SolidWorks / FreeCAD | Fusion 360 | Integrated CAD + FEA, parametric, free student license |
| Parametric approach | Fixed dimensions / Parameter-driven | Parameter-driven | Enables single-point iteration, documents design intent |
| Arm angle | 90° / 100° / 120° | 100° | Mid-range for FSAE, creates meaningful bending at elbow |
| Clevis vs. lug | Fork (clevis) / Single lug | Clevis fork | Double-shear, realistic mounting, demonstrates DfM |
| Weight reduction method | Topology optimization / Manual pockets | FEA-guided manual pockets | More transparent decision-making, better portfolio narrative |
| Pocket shape | Rectangular / Arc-shaped | Arc-shaped | Smooth stress flow, no sharp corners, no stress concentration |
| Fillet sizing | Uniform / Zone-based | Zone-based (3mm structural, 1mm general) | Larger fillets where stress concentrates, smaller elsewhere |

---

## 5. Project Structure

```
rocker-case-study/
├── README.md                  ← This document
├── cad/
│   ├── Rocker_v1.f3d          ← Fusion 360 source file (baseline)
│   ├── Rocker_v2_optimized.f3d ← Optimized version
│   └── Rocker_v2.step          ← Export for SimScale / other tools
├── fea/
│   ├── load-derivation.md      ← Full calculation with free body diagram
│   ├── baseline-results/       ← Screenshots, Von Mises plots
│   └── optimized-results/      ← Comparison screenshots
├── aero/                       ← Phase 2 (upcoming)
│   ├── wing-profile/
│   └── cfd-results/
├── manufacturing/              ← Phase 3 (upcoming)
│   ├── print-orientation.md
│   └── stl-files/
└── portfolio/                  ← Final presentation assets
    ├── figures/
    └── case-study.pdf
```

---

## 6. Next Steps

- [ ] **Phase 2 — Aerodynamic Analysis:** External CFD of a rear wing section (S1223 + NACA 6412 flap) at FSAE-relevant Reynolds numbers using SimScale/OpenFOAM
- [ ] **Phase 3 — Manufacturing:** 3D print preparation (orientation analysis, support strategy, DfAM considerations), physical prototype
- [ ] **Phase 4 — Portfolio Assembly:** Combined case study document with engineering narrative

---

## Tools & Resources

| Tool | Purpose |
|------|---------|
| Fusion 360 (Education) | Parametric CAD + FEA |
| SimScale (Community) | CFD for aero module (upcoming) |
| UIUC Airfoil Database | Wing profile coordinates |
| Python / MATLAB | Data processing and visualization |

---

*This case study is an independent portfolio project demonstrating end-to-end mechanical engineering workflow — from requirements derivation through analysis-driven design iteration.*
