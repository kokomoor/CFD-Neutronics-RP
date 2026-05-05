# Coupled Monte Carlo Neutronics and Two-Phase CFD for the Molten Salt Fast Reactor

**Author:** Samuel Kokomoor
**Affiliation:** MIT Nuclear Science and Engineering
**Document type:** Research proposal

This repository holds the research proposal for a multiphysics simulation effort focused on the Molten Salt Fast Reactor (MSFR). The full proposal is in `ResearchProposal_Kokomoor.md` (with PDF and slide versions also included). This README summarizes the problem, the proposed approach, and the expected contribution.

---

## Problem Statement

The MSFR is a Gen-IV concept in which the fuel salt is also the coolant and circulates through the core, heat exchangers, and pumps. Fission power, delayed neutron production, and decay heat all move with the fuel rather than staying where they are produced. Helium bubbling, used to strip out gaseous fission products, introduces a dispersed gas phase in parts of the loop. None of this is captured by static or single-phase models.

Existing high-fidelity efforts have made progress but each leaves a gap. Deng et al. (2020) coupled OpenMC with OpenFOAM at steady state but assumed single-phase incompressible flow and emitted delayed neutrons at the fission site. Cervi et al. (2019) showed that ignoring fuel compressibility and gas voids materially distorts reactivity feedback. Dalinger et al. (2025) demonstrated DNP and DHP convection inside Argonne's Cardinal but could not relocate the delayed neutron birth sites in OpenMC, because the underlying source routine did not allow it.

> No existing tool fully couples Monte Carlo neutronics with two-phase CFD while updating the neutron source on the basis of where precursors have actually drifted. This proposal targets that gap.

---

## Proposed Approach

The work develops a tightly coupled simulation framework that exchanges fields between OpenMC and a two-phase CFD solver (MFC and NekRS are shortlisted) at each outer iteration or coupling interval. The CFD side returns local temperature, density, void fraction, and precursor concentrations. The neutronics side returns power deposition and an updated neutron source. The coupling layer is solver-agnostic so the CFD backend can change without rewriting the OpenMC interface.

The defining technical contribution is treating delayed neutron precursors (DNPs) and decay heat precursors (DHPs) as convected scalar fields in the CFD, then feeding the resulting concentration maps back into OpenMC so that delayed neutron emission samples from where the precursors actually are at that moment, not from where the fissions originally occurred. This closes the limitation identified in the Cardinal demonstration and makes spatially resolved precursor drift a first-class part of the simulation.

A low-Mach variable-density formulation is the default on the CFD side, since salt velocities of roughly 1 to 3 m/s against a sound speed of 2 to 3 km/s give Mach numbers well below 10^-2. A fully compressible mode is reserved for cases where gas compressibility or pressure transients meaningfully change the feedback.

---

## Objectives

1. **Coupling framework.** Build a solver-agnostic interface that handles mesh exchange, parallel synchronization, and coupling control between OpenMC and the chosen CFD backend.
2. **Precursor-aware neutronics.** Modify the OpenMC source routine (or add a driver-level wrapper) so delayed neutron emission is sampled from a CFD-supplied precursor field.
3. **Two-phase, low-Mach modeling.** Configure the CFD solver to represent liquid salt with entrained helium bubbles, with an optional fully compressible mode for targeted studies.
4. **Precursor and decay-heat transport.** Add convective transport equations for each DNP and DHP group inside the CFD, verify mass conservation, and check simple analytical limits.
5. **Validation cases.** Reproduce the steady-state results of Deng et al. (2020), an MSRE-like benchmark for delayed neutron behavior, and at least one transient (such as a pump coastdown or inlet temperature perturbation).
6. **Sensitivity and design analysis.** Use the validated tool to study how flow rate, gas injection rate, and core geometry shape the effective delayed neutron fraction, hot-spot formation, and reactivity stability.

---

## Performance Targets

| Mode | Mesh size (CVs) | Time horizon | Neutronics cadence | Wall time on 256 to 512 cores |
|---|---|---|---|---|
| Steady state (URANS) | 3 to 8 million | n/a | per outer iteration | 2 to 6 hours |
| Transient (URANS / low-Mach mixture) | 2 to 5 million | 10 to 30 s | every 0.25 to 0.5 s | 4 to 12 hours |

Mesh-to-mesh transfers use a conservative volume-weighted projection with a nearest-cell fallback. Exchanges occur in memory over MPI. Each bidirectional exchange is roughly 0.2 to 0.6 GB in double precision and is expected to remain under 5% of total runtime.

---

## Mathematical Core

For each delayed neutron precursor group $i$ with decay constant $\lambda_i$:

$$\frac{\partial C_i}{\partial t} + \nabla\cdot(\mathbf{u}C_i) = \beta_i S_f - \lambda_i C_i$$

For each decay-heat precursor group $j$:

$$\frac{\partial H_j}{\partial t} + \nabla\cdot(\mathbf{u}H_j) = f_j S_f - \lambda_j H_j$$

The distributed decay heat source returned to the CFD energy equation is:

$$\dot{q}_{\text{decay}}(\mathbf{r},t) = \sum_j \lambda_j H_j E_j$$

The $C_i$ fields are passed to OpenMC each coupling step. The driver constructs a probability density proportional to $C_i(\mathbf{r}) \nu_i \lambda_i$ and samples delayed neutron birth sites from that distribution. Prompt neutrons are still emitted at the fission site as usual.

---

## Tracks

The work is structured so its components are self-contained and can be delivered in either of two scopes.

**Master's track (18 to 24 months).** Single-phase steady-state coupling and baseline validation in months 0 to 6. Two-phase mixture, DNP and DHP transport, and relocated delayed neutron emission on a simplified geometry in months 6 to 12. Full MSFR steady state and a short transient of 5 to 10 s in months 12 to 18. An optional months 18 to 24 block covers sensitivity studies and robustness hardening.

**Ph.D. track (3 to 4 years).** Year 1 mirrors the Master's first year plus an MSRE-like benchmark. Year 2 covers full-geometry steady state, multiple transients, and the optional fully compressible mode. Year 3 adds uncertainty quantification, expanded sensitivity studies, and cross-code comparisons. Year 4 covers advanced turbulence treatment (LES where feasible) and final validation and publication.

---

## Expected Outcomes

The primary deliverable is a validated coupled solver linking OpenMC with a two-phase CFD backend, capable of steady-state and transient analysis with precursor drift handled consistently on both sides. The methodology and any custom OpenMC modifications will be documented and made available so other groups can reuse the coupling driver, either for other MSR designs or for solid-fueled systems with strong void feedback.

The simulation campaign will produce quantitative answers to questions that point-kinetics and single-phase models cannot resolve. These include the fraction of delayed neutrons that are born outside the active core, the spatial distribution of decay heat under nominal flow, the reactor's dynamic response during a flow coastdown, and the sensitivity of $\beta_{\text{eff}}$ and power profile to fuel circulation rate and gas injection rate.

These results are intended to support both reactor design iteration and the licensing case for MSFR technology, and to provide a template for integrated multiphysics analysis applicable to other Gen-IV concepts.

---

## Repository Contents

| File | Description |
|---|---|
| `ResearchProposal_Kokomoor.md` | Full proposal, source of record |
| `ResearchProposal_Kokomoor.pdf` | PDF rendering of the proposal |
| `ResearchProposal.pptx` | Slide version |

---

## Key References

1. Deng et al. (2020), *Nuclear Science and Techniques* 31:85. Coupled OpenMC and OpenFOAM analysis of the MSFR core and blanket.
2. Cervi et al. (2019), *Chemical Engineering Science* 193, 379 to 393. Multiphysics OpenFOAM model of fuel compressibility and gas void effects in the MSFR.
3. Dalinger et al. (2025), NURETH-21, paper 2507. High-fidelity Cardinal-based MSFR model with DNP and DHP convection.
4. Gérardin et al. (2017), FR17 Proceedings. Design evolution of the MSFR.
5. Zhao et al. (2022), *Journal of The Electrochemical Society* 169(7), 046520. Multiphysics simulation of molten salt pyroprocessing.
