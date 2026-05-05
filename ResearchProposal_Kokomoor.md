# Coupling of Monte Carlo Neutronics with Two-Phase CFD for Molten Salt Fast Reactor Analysis in Realistic Geometry: Modeling Delayed Neutron and Decay Heat Precursor Drift

**Author:** Samuel Kokomoor  
**Date:** 11/12/2025
**Document:** Research Proposal  
**Department:** MIT Nuclear Science and Engineering

---

## Abstract

> Molten Salt Fast Reactors (MSFRs) present a unique multiphysics challenge: the liquid salt serves as both fuel and coolant and circulates throughout the core, creating highly coupled neutronic and thermal-hydraulic behavior.

This proposal aims to develop a high-fidelity simulation framework that couples the OpenMC Monte Carlo neutronics code with a two-phase computational fluid dynamics (CFD) solver (such as the Multiphase Flow Code, or NekRS) to model an MSFR core. The novelty of the work lies in explicitly tracking delayed neutron precursors (DNPs) and decay heat precursors (DHPs) as they drift with the flowing fuel, and feeding this information back into the neutronics calculation.

This capability will capture spatiotemporal feedback effects that are missed by conventional static or single-phase models. A coupling architecture will be defined that exchanges local temperature, density, and precursor concentrations from the CFD side with power deposition and neutron source updates from the neutronics side on-the-fly.

Quantitative targets: steady-state solutions will use URANS with approximately 3–8 million control volumes (CVs); transients of 10–30 s will use 2–5 million CVs with neutronics updated every 0.25–0.5 s of physical time. On 256–512 CPU cores, end-to-end wall times are expected to be roughly 4–12 hours per case. Flow in the salt is low-Mach (estimated salt velocity 1–3 m/s and speed of sound 2–3 km/s thus Mach number < 10^−2), so a low-Mach variable-density formulation will be used by default; fully compressible two-phase models will be exercised only where gas compressibility or pressure transients materially affect feedback.

The expected outcome is a validated tool capable of simulating steady-state and transient behavior of the MSFR with unprecedented fidelity. This research will provide insights into reactor performance, safety margins, and design optimization for liquid-fueled reactors, and more broadly demonstrate the feasibility of integrating Monte Carlo methods with advanced two-phase flow solvers for nuclear applications.

---

## Background and Motivation

Molten salt reactors (MSRs) are liquid-fueled systems where fuel salt circulation and on-line processing lead to strongly coupled physics phenomena. In particular, the Molten Salt Fast Reactor (MSFR) is a Gen-IV design with a fast neutron spectrum and circulating fluoride-salt fuel. This MSFR design exhibits continuous feedback between neutronics and thermal-hydraulics. The reference MSFR developed in the SAMOFAR project (EU) is a 3000 MW$_{\text{th}}$ design with ~18 m³ of LiF-ThF$_4$-UF$_4$ fuel salt operating at ~725 °C.

Liquid fuel is pumped through the core and external loops, which include heat exchangers and pumps, as well as systems for helium bubbling to remove gaseous fission products. This means that fission power, delayed neutron production, and decay heat deposition all move with the fuel, unlike in solid-fueled reactors. Accurate modeling of an MSFR requires capturing these multiphysics interactions.

Traditional one-way or loosely coupled analyses are insufficient. For example, using a stand-alone thermal-hydraulics code with fixed power profile, or neutronics with static temperature fields, misses key feedbacks. Early MSR studies in the 1960s treated delayed neutron precursor drift analytically (adjusting point-kinetics models based on fuel residence time), but spatial details were neglected.

Recent efforts have moved toward high-fidelity coupling. Deng et al. (2020) developed a coupled OpenMC/OpenFOAM simulation of the MSFR core and fertile blanket, resolving 3-D fluid flow and heat transfer with Monte Carlo neutronics. Their steady-state analysis revealed strong neutronic–thermal interactions: for instance, fuel recirculation zones caused heat to accumulate near the outer blanket region, locally raising temperature and thus reducing reactivity. This demonstrated that a multiphysics approach can identify phenomena (like recirculation-driven hot spots) that would be missed by simpler models. However, that study assumed single-phase, incompressible flow (the fuel salt was liquid and boiling was avoided by design) and did not model the motion of delayed neutron precursors; effectively treating delayed neutron emission as if it occurred at the fission site.

Researchers have begun addressing these limitations. Kim et al. (2025), for example, introduced delayed neutron precursor drift and on-the-fly cross-section updates in a Monte Carlo–CFD coupling for MSRs, highlighting improved accuracy over earlier deterministic or point-kinetics-based approaches. In parallel, thermal-hydraulic advances have enabled two-phase flow modeling in reactor cores. The MSFR design itself incorporates a helium injection system to strip out fission gases, creating a dispersed gas–liquid flow in parts of the fuel circuit.

Cervi et al. (2019) developed a multiphysics OpenFOAM model to investigate fuel compressibility and gas void effects in the MSFR. They found that assuming an incompressible single-phase fluid can significantly misjudge reactivity feedback: the expansion of bubbles and the delay in fuel density change alter the transient response, introducing time lags in the negative reactivity feedback during rapid power excursions. These findings underline the importance of modeling two-phase, compressible behavior for realistic MSFR safety analysis.

Most recently, Dalinger et al. (2025) demonstrated a high-fidelity coupled simulation of an MSFR using Argonne's Cardinal code, which links OpenMC with the NekRS CFD solver. Their model included convective transport of DNPs and DHPs in the fuel, alongside full 3-D Navier–Stokes and heat transport with turbulence modeling. This study showed that the distribution of delayed neutron sources and decay heat in the fuel salt can be tracked in a coupled simulation. However, a critical limitation was identified: the current OpenMC code could not modify the location of delayed neutron emission in response to flowing precursors. In other words, although the CFD calculated where precursors convect, the Monte Carlo neutronics still assumed delayed neutrons were born where fissions occurred, due to software limitations.

> **This highlights a key gap in current capabilities:** no existing tool fully couples Monte Carlo neutronics with two-phase CFD while dynamically updating neutron source based on precursor drift.

The proposed research will fill this gap. By enabling OpenMC to ingest a time- and space-dependent distribution of DNPs (and associated neutrons) from the CFD, we will capture the true spatiotemporal neutronic feedback of fuel motion. This extends beyond prior steady-state studies (like Deng et al. 2020) by incorporating transient physics of moving fuel, and goes further than Cardinal's demonstration by implementing the missing feedback of delayed neutron source relocation.

Relative to prior DNP-focused studies, this work (i) relocates delayed neutron birth sites inside OpenMC based directly on CFD-resolved precursor fields (closing the Cardinal limitation), (ii) models decay-heat precursor drift and spatial deposition in tandem with DNPs, and (iii) quantifies the error incurred when precursor drift is ignored under representative MSFR flow conditions.

The impact of this advancement is wide-ranging. It will improve predictions of effective delayed neutron fraction and reactivity in MSFRs (crucial for control and safety margins), and allow analysis of phenomena like flow-induced reactivity oscillations or the consequences of gas entrainment in the core. Moreover, developing a general Monte Carlo–CFD coupling approach has broader significance: it can be applied to other reactor types (e.g. thermal MSRs with graphite moderators, or even boiling water reactors where void drift is important) and will push the envelope of high-fidelity reactor simulation.

Even outside reactors, multiphysics simulations in molten salt systems are proving valuable. For example, Zhao et al. (2022) coupled fluid flow, species transport, and electrochemistry to study molten-salt pyroprocessing in an electrorefiner. This illustrates the growing demand for integrated simulation tools in the nuclear field. This work, focused on the MSFR core, is at the nexus of this trend, combining advanced neutronics and two-phase CFD to provide unprecedented detail and realism in modeling a promising Gen-IV reactor concept.

---

## Proposed Research Objectives

The overall goal is to develop and demonstrate a coupled Monte Carlo-CFD simulation capability for the MSFR that includes precursor drift effects. The specific objectives of the proposed research are:

### 1. **Develop a Coupling Framework**

Design and implement a solver-agnostic software interface to tightly couple OpenMC with a two-phase CFD solver (MFC and NekRS are shortlisted). This includes mesh data exchange, parallel synchronization, and runtime control to iterate between neutronics and fluid dynamics solvers each time step or outer iteration.

### 2. **Incorporate Precursor Transport in Neutronics**

Extend OpenMC (or its driver routines) to allow spatially and temporally varying delayed neutron sources. In practice, this means feeding OpenMC a map of delayed neutron precursor concentrations computed by the CFD solver so that delayed neutron emissions are sampled from the correct locations in the fuel salt. This may involve modifying OpenMC's source routine or using a custom tally-based approach to weight emissions according to precursor distribution.

### 3. **Implement Two-Phase, Low-Mach Variable-Density Modeling (optional fully compressible mode)**

Configure the CFD solver to model liquid fuel salt with entrained helium gas bubbles in the core. A low-Mach variable-density two-phase formulation (two-fluid or diffuse-interface) will be used by default to capture void fraction and density variations efficiently at Mach numbers well below 10^−2; an optional fully compressible mode will be enabled only for scenarios where gas compressibility or pressure-wave physics measurably impact reactivity feedback. Validation of the fluid model (without neutronics) will be performed against available correlations or lower-fidelity codes (e.g., comparing single-phase results to Deng et al. 2020, and two-phase void physics to Cervi et al. 2019).

### 4. **Simulate Precursor Drift and Heat Deposition**

Implement a convective transport model for delayed neutron precursors and decay heat precursors within the CFD solver. Each group of precursors will be treated as a passive scalar field that convects with the fluid velocity and decays with its characteristic half-life. The solver will produce time-dependent precursor concentration fields, which will be used to (i) adjust the delayed neutron source in OpenMC, and (ii) compute distributed decay heat release in the fluid. This objective includes verifying the implementation by checking mass conservation of precursors and comparing against analytical solutions in simple flow cases.

### 5. **Validation and Demonstration Cases**

Apply the coupled code to benchmark scenarios and MSFR case studies. This will include: (a) a steady-state simulation of the MSFR core to compare against published results (e.g., temperature and power profiles from Deng et al., and delayed neutron fraction estimates from simpler models), and (b) a transient scenario such as a pump coastdown or an inlet temperature perturbation, to observe the dynamic response of power and precursors. The code will be validated to reproduce known qualitative behaviors (e.g., delay in power response due to precursor convection) and, where possible, quantitative trends reported in literature (such as the magnitude of reactivity feedback from thermal expansion or fraction of precursors lost to the external loop as estimated by point kinetics).

### 6. **Sensitivity and Design Analysis**

Using the new tool, evaluations will be performed examining the influence of key design and operation parameters on MSFR behavior. For example, insights can be gained on the effects of varying the fuel circulation rate or gas injection rate on the effective delayed neutron fraction and spatial power profile. Design modifications can also be assessed (such as alternative loop configurations or core shapes) for their impact on hot spot formation and reactivity stability. The objective is to provide feedback to reactor designers on optimizing core geometry and flow conditions to minimize undesirable effects like localized overheating or reactivity oscillations.

---

> **Measurable Outcomes:** Each of these objectives is measurable: software will be produced demonstrating the coupling (Objective 1), new or modified OpenMC capabilities will be demonstrated with respect to precursor transport (Objective 2), CFD results showing two-phase flow features will be produced for cross-examination (Objective 3), verification data for precursor transport will be generated (Objective 4), documented comparison plots/numerical values for benchmark cases will be produced (Objective 5), and sensitivity studies will be conducted (Objective 6). Achieving these objectives will collectively realize the project goal of a coupled high-fidelity MSFR simulation with precursor drift.

---

## Methodology and Work Plan

To accomplish the above objectives, a phased approach will be employed that iteratively builds and tests the coupled simulation capability. The core tasks involve software development for code coupling, physical modeling of the reactor, and progressive validation. The methodology is described below in detail, including the coupling architecture, mathematical models, and planned validation cases.

---

### Coupling Architecture

A tight coupling scheme will be adopted in which OpenMC and the CFD solver run in an iterative or synchronized manner at each time step. A high-level driver (likely written in C++, possibly building on the framework used in Cardinal) will coordinate the two codes.

**Steady-state mode (eigenvalue calculation):**
1. Initialize fluid properties (guess temperature, etc.)
2. Run OpenMC to obtain an initial power distribution
3. Run the CFD solver to steady-state with that power input
4. Update material temperatures/densities in OpenMC and repeat the neutronics calculation
5. Iterate until successive changes in power or temperatures fall below a convergence threshold

**Transient mode:** The coupling will proceed in small time increments: e.g., each time step the CFD advances the flow and precursor equations by $\Delta t$ using the power from the previous neutronics solve, then OpenMC is run with updated conditions to produce a new power map for the next step.

Experiments will start with an explicit coupling (staggered) approach for simplicity, and later evaluate implicit schemes or subcycling if needed for stability. Data exchange will be facilitated by mapping the CFD mesh to the neutronics mesh. Because OpenMC can handle continuous-energy physics on an unstructured mesh (via DAGMC), this methodology can use the exact same geometry for both solvers (importing a CAD model of the MSFR core into both codes). Temperature and density from CFD cells will be averaged or interpolated into the regions (cells or material zones) used in OpenMC. Similarly, OpenMC's fission power tallies will be projected onto the CFD mesh to act as a volumetric heat source in the energy equation.

Mesh-to-mesh transfers will use a conservative, parallel volume-weighted projection (with nearest-cell fallback for discontinuous fields). Exchanges occur in-memory over MPI (no disk I/O). For 3–5 million CVs and ≈12 scalar/vector fields (T, ρ, void fraction, 6–8 DNP groups, DHP groups), each bidirectional exchange is O(0.2–0.6) GB in double precision; with neutronics updates every 0.25–0.5 s of simulated time (every O(50–100) CFD steps), the wall-clock overhead of data exchange is expected to remain <5% of total runtime.

### Monte Carlo Neutronics with Moving Precursors

OpenMC will be used in eigenvalue (criticality) mode to compute the neutron flux and fission power distribution in the core. OpenMC's continuous-energy cross section library and the on-the-fly Doppler broadening capability will be utilized to update cross sections according to local temperatures.

A crucial extension will be handling delayed neutrons with drifting precursors. Normally, Monte Carlo criticality calculations distribute delayed neutron emission proportional to the fission rate in each region, assuming precursors are immobile. This will be modified by splitting the neutron source into prompt and delayed components:

- **Prompt neutrons:** Emitted at the point of fission as usual
- **Delayed neutrons:** A custom source routine will be implemented that samples emission sites from the precursor concentration field provided by the CFD solver

Practically, the CFD will output the concentration $C_i(\mathbf{r})$ of each precursor group $i$ on a mesh; the driver will construct a probability density function for delayed neutron emission proportional to $C_i(\mathbf{r}) \nu_i \lambda_i$ (where $\nu_i$ is the number of neutrons per precursor decay and $\lambda_i$ the decay constant). OpenMC can then sample from this distribution for the delayed neutrons in the source bank for each batch.

This approach effectively "teleports" delayed neutron birth to where precursors have moved at that time, as opposed to where they were born. If direct modification of OpenMC's source code is required, it will be implemented (OpenMC is open-source and accessible for research needs).

By including this capability, the neutronic calculation will properly account for delayed neutrons introduced into the core from precursors that may have drifted out of the high-flux region or even into external loops. The effect will be quantified by comparing the effective delayed neutron fraction ($\beta_{\text{eff}}$) calculated in the model versus the conventional assumption. Differences are expected, as fuel circulation in liquid-fueled reactors can cause delayed neutron precursors to decay outside the core, reducing their contribution to reactivity.

### Two-Phase CFD Model

On the fluids side, a compressible two-phase flow model will be used to represent the circulating fuel salt and injected helium bubbles. 

**Code selection and rationale (solver-agnostic coupling):**
- Shortlist: MFC (Multiphase Flow Code) and NekRS (with Nek-2P capabilities).
- Selection criteria: availability of a low-Mach two-phase formulation with robust positivity/maximum-principle enforcement, mature GPU/CPU scalability on available clusters, ease of mesh/field exchange via MPI, and stability on long time horizons.
- The coupling layer is designed to be solver-agnostic; the deliverable is the OpenMC coupling interface plus validation cases, independent of which CFD backend is chosen.

The model will solve the Navier-Stokes equations for the liquid-gas mixture, along with separate continuity equations for each phase or a volume-fraction transport equation for helium. A realistic equation of state (EOS) will be employed: the liquid fuel salt will be treated as compressible via a stiffened gas EOS (to account for its small but nonzero compressibility and to allow pressure wave propagation), and helium as an ideal gas. This enables simulation of, for example, pressure transients from rapid power changes and the effect of thermal expansion of the fuel.

Compressibility justification: expected salt velocities are 1–3 m/s and sound speed 2–3 km/s (Mach number 3×10^−4–1.5×10^−3), indicating that fully compressible solvers are unnecessary for most operating conditions; however, gas compressibility and pressure transients can influence void distribution and feedback in some scenarios. Accordingly, the baseline will use a low-Mach variable-density formulation; the fully compressible mode is reserved for targeted studies that require it (e.g., rapid valve events).

Turbulence will be modeled initially with RANS (a $k$–$\varepsilon$ or $k$–$\omega$ model) to obtain steady solutions; for higher fidelity, Large Eddy Simulation (LES) can be switched to as in Cardinal's demonstration for transient analyses if computational resources permit. The two-phase model will include drag and momentum exchange between helium and liquid, but since the helium bubble residence time in the core is short (gas is buoyant and quickly separated), a simpler homogeneous model with an effective density reduction in regions of helium injection may also be considered for steady-state calculations.

### Delayed Neutron and Decay Heat Precursor Transport

A distinguishing feature of this work is the explicit modeling of precursor transport in the CFD solver. Transport equations will be added for each group of delayed neutron precursors: for group $i$ with decay constant $\lambda_i$, the governing equation is (following Dalinger et al., 2025 [5]):

$$\frac{\partial C_i}{\partial t} + \nabla\cdot(\mathbf{u}C_i) = \beta_i S_f - \lambda_i C_i,$$

where $C_i(\mathbf{r},t)$ is the concentration of precursor group $i$ in the fluid, $\mathbf{u}$ is the local fluid velocity, and $S_f(\mathbf{r},t)$ is the fission source (neutrons emitted from fission per unit volume, obtained from OpenMC). The term $\beta_i S_f$ represents the production of new precursors from fission (with $\beta_i$ being the fraction of fission neutrons that belong to group $i$), and $-\lambda_i C_i$ is the loss due to radioactive decay (which yields the delayed neutrons). Similarly, for decay heat precursors (DHPs), fields $H_j(\mathbf{r},t)$ will be introduced representing the concentration of heat-producing radioactive species (these could be the same as the delayed neutron groups or an expanded set including non-neutron-yielding isotopes). They satisfy a similar groupwise production–decay balance (following Dalinger et al., 2025 [5]):

$$\frac{\partial H_j}{\partial t} + \nabla\cdot(\mathbf{u}H_j) = f_j S_f - \lambda_j H_j,$$

where $S_f(\mathbf{r},t)$ denotes the fission source term, $f_j$ is the production fraction (per fission) for decay-heat group $j$, and $\lambda_j$ is its decay constant. These precursor equations will be solved alongside the fluid continuity, momentum, and energy equations in the CFD solver. For numerical stability, operator splitting may be used (advect precursors with the flow, then decay them in a separate step). The solver will output the precursor concentrations $C_i$ and $H_j$ at each coupling step. The $C_i$ fields feed into OpenMC as discussed, while the $H_j$ decays contribute a **distributed heat source** in the fluid energy equation via

 $$\dot{q}_{\text{decay}}(\mathbf{r},t) = \sum_j \lambda_j H_j \, E_j,$$

where $E_j$ is the energy released per decay of group $j$ (cf. Dalinger et al., 2025 [5]). This treatment will enable tracking where heat from delayed fission products is deposited. For example, if fuel exits the core and decays in the external loop or decay tank, that heat is deposited there rather than in the core, affecting the thermal feedback on the reactor. The implementation will be verified by checking simple limits: in the zero-flow limit, the model should reduce to the standard point-kinetics equations for precursors in a stationary core; in the fast-flow limit, many precursors should be convected out and decay externally, reducing $\beta_{\text{eff}}$, consistent with analytical estimates.

### Code Implementation and HPC Considerations

The coupling will be executed on a high-performance computing platform due to the significant computational load (Monte Carlo with millions of particles, CFD with millions of mesh cells). MPI-based parallelism will be used, possibly running OpenMC and the CFD solver concurrently on different subsets of cores (this was demonstrated in OpenMC–OpenFOAM couplings to reduce idle time). Load balancing will be addressed: the CFD solve is typically time-stepping (which can be slower than a single OpenMC eigenvalue solve for steady state), but for transients the neutronics is likely to be relatively fast per time step. If needed, the frequency of neutronics solves will be adjusted (e.g., not call OpenMC every CFD time step if the feedback changes slowly) to save time. The use of adaptive time stepping or subcycling might be implemented: for example, small time steps for CFD to resolve fast fluid transients, but neutronics updates at a larger interval with interpolation of fluid data in between. An efficient strategy will be identified during testing.

#### Target Resolution, Throughput, and Time-to-Solution

- Baseline steady-state (URANS): 3–8 million CVs; mesh-to-mesh exchange every outer iteration; expected wall time 2–6 h on 256–512 CPU cores.
- Baseline transient (URANS/low-Mach mixture): 2–5 million CVs; $\Delta t$ = 2–10 ms; simulate 10–30 s of reactor time; neutronics updates every 0.25–0.5 s (20–120 OpenMC solves total); expected wall time 4–12 h on 256–512 CPU cores for 10–15 s transients.
- Comparison to state of the art: Deng et al. (2020) reported single-phase incompressible coupling at similar mesh sizes but without precursor drift; Cardinal (2025) demonstrated two-phase LES without relocating delayed neutron birth sites. The proposed targets match or exceed mesh/time scales while adding DNP/DHP drift and relocated delayed neutron emission in OpenMC.

#### Numerical Stability and Risk Mitigation

- Low-Mach variable-density formulation for efficiency and stability at $M_{\text{salt}} \ll 1$; fully compressible mode only for targeted studies.
- Positivity enforcement for density, pressure, and scalar fields; failure detection with checkpoint/restart.
- Routine NaN/Inf checks with automatic rollback to last good checkpoint to avoid long-run failures.

### Validation and Testing Plan

The research will progress through increasingly realistic tests:

- **Unit Tests and Simple Geometries:** Verification will begin with individual components. For the precursor transport solver, testing will be performed against an analytical 1D case (e.g., plug flow in a pipe with known decay solution). For the coupling interface, a fast reactor pin-cell model with imposed coolant flow can be used to ensure delayed neutron sampling works (a simplified geometry in which flow speed can be varied and the change in neutron source distribution observed). A known benchmark will also be reproduced: the MSRE (Molten Salt Reactor Experiment) had measured data on delayed neutron fraction and dynamics. An MSRE-like model can be set up in the framework as a proof-of-concept (using available dimensions and flow rates from ORNL reports) expecting to see, for example, ~20% of delayed neutrons born outside the core as was observed.

- **Steady-State MSFR Benchmark:** Next, the MSFR core will be simulated at nominal full power steady-state. Geometric details will be based on the SAMOFAR design (including core cavity, 16 loop channels, etc., possibly simplified by omitting the fertile blanket in initial runs and/or simulating a radially-symmetric subset of the core). Results will be compared to Deng et al. (2020) and Dalinger et al. (2025). Key metrics: core outlet temperature (~998 K expected), temperature distribution patterns (e.g., is there a hot region near the top as in Deng's single-phase study?), and the $k_{\text{eff}}$ and power profile. Because Deng's study did not include moving precursors, one novel result from the simulation will be the spatial distribution of delayed neutron emission. For validation, the total fraction of delayed power produced outside the core might be compared to estimates from simpler calculations, and the code will be ensured to predict criticality ($k_{\text{eff}} \approx 1.0$) at the expected operating temperature with the given fuel composition. A mesh sensitivity study will also be performed (refine CFD mesh and increase number of neutrons) to check solution convergence.

- **Transient Simulation:** Finally, a coupled transient will be demonstrated to showcase the capabilities of the new tool. A candidate scenario is a **pump slowdown (flow coastdown)**, which causes fuel residence time in the core to increase. A transient will be initiated by reducing the inlet velocity boundary condition in the CFD solver and observing the reactor power response. Physically, as flow slows, more delayed neutron precursors remain in the core (increasing the effective delayed neutron fraction), which tends to make the reactor slightly more stable; however, slower flow also means less heat removal, raising core temperature, which provides negative reactivity feedback. The simulation will capture the competition of these effects. The transient behavior will be compared to point-kinetics predictions using traditional models (a point-kinetics model with flow-dependent $\beta_{\text{eff}}(t)$ can be constructed for comparison). The goal is to show that the spatially resolved model provides additional insight; for instance, identifying any power redistribution or localized hot spots during the transient, which point models cannot predict. Throughout these steps, parts of the model will also be validated against independent codes or data, for instance the SERPENT Monte Carlo code (if available) with a static precursor model can be used to ensure the OpenMC modifications produce consistent steady-state results. Experimental data or correlations for heat transfer in molten salts will also be consulted to verify that the Nusselt number and friction factor in the CFD align with expectations (ensuring the turbulence model is reasonable).

### Work Plan and Timeline

To align scope with degree pathways and reduce schedule risk, two tracks are defined. The modular structure ensures components are self-contained and sequentially integrable in either track.

#### **Master's track (18–24 months)**
- Months 0–6: Reproduce single-phase OpenMC↔CFD steady-state coupling; establish solver-agnostic interface; mesh-to-mesh mapping; baseline validation versus Deng et al. (2020).
- Months 6–12: Implement low-Mach two-phase mixture with helium volume fraction; implement and verify DNP/DHP transport; demonstrate relocated delayed neutron emission in OpenMC on simplified geometry.
- Months 12–18: End-to-end steady state and a short transient (5–10 s) in full MSFR geometry; quantify impact of precursor drift on $\beta_{\text{eff}}$ and power distribution; deliver quantitative performance targets (mesh size, update cadence, wall time).
- Optional months 18–24: Sensitivity study (flow rate, gas injection rate) and robustness hardening (restart, bounds/positivity checks); thesis write-up and dissemination.

#### **Ph.D. track (3–4 years)**
- Year 1: As Master's Months 0–12, plus expanded validation set (MSRE-like benchmark).
- Year 2: Full-geometry MSFR steady state; multiple transients (pump coastdown, inlet temperature perturbation); add optional fully compressible mode for targeted studies.
- Year 3: Uncertainty quantification (nuclear data, turbulence/void models), extended sensitivity/design studies, and cross-code comparisons (e.g., SERPENT static checks).
- Year 4 (as needed): Advanced turbulence (LES where feasible), additional features (e.g., chemistry/radiolysis if relevant), and comprehensive validation/publication.

**Dissemination:** Throughout the project, progress will be disseminated in the form of conference papers (e.g., PHYSOR for coupling methods, NURETH for thermal-hydraulics) and ultimately a journal article summarizing the completed multiphysics tool and key findings for MSFR behavior. Experts in the MSR community (such as collaborators from the SAMOFAR project or Argonne) will be engaged for periodic review and feedback, ensuring the model remains relevant to ongoing design efforts.

---

## Expected Outcomes and Impact

Upon completion, this research will deliver both a new simulation capability and critical engineering insights for molten salt reactors:

---

### **Coupled Monte Carlo–CFD Solver**

The primary tangible outcome is a validated code system that links OpenMC with a two-phase CFD solver for full-core MSFR analysis. This tool will be able to simulate steady-state and transient conditions accounting for 3-D neutronics, thermal hydraulics, and precursor transport in tandem.

It will be one of the first of its kind. Previous high-fidelity MSR simulations have either used deterministic neutronics or ignored moving precursors. By overcoming these limitations, the solver sets a new benchmark for realism in reactor modeling.

The methodology (and any custom software developed) will be documented and made available to the research community, potentially as an extension to OpenMC or as an open-source coupling driver. This ensures that other researchers can build upon the work, applying it to similar problems (e.g., coupling Monte Carlo to subchannel codes for BWR analysis or to other Gen-IV reactor concepts).

---

### **Enhanced Understanding of MSFR Physics**

From the simulations, a **detailed picture of the MSFR core's behavior** under various conditions will be obtained. For example, how delayed neutron precursors redistribute in the core and external loops will be quantified. This will reveal the fraction of delayed neutrons born in regions of lower importance (like the pump or heat exchanger) and how that affects reactivity feedback.

The decay heat deposition will also be mapped out. Understanding what fraction of fission energy is deposited outside the core can inform the design of the primary loop and decay heat removal systems. The project will clarify the dynamic response of the reactor: for instance, the time lag between a perturbation (like a change in flow rate or insertion of reactivity) and the reactor's power response will be observed, as influenced by precursor drift and thermal feedback.

Such information is vital for **safety analysis**, as it relates to the inherent stability of the reactor and the design of control systems. Claims about the MSFR's safety features (e.g., strongly negative temperature coefficient) will be evaluated in a space- and time-resolved manner rather than relying on point kinetics.

---

### **Design and Operational Insights**

The high-fidelity results can be analyzed to suggest improvements or verify design choices for the MSFR. For example, if certain flow recirculation patterns are found to lead to localized high power density (hot spots), the reactor designers could consider geometric modifications (such as the curvature of flow guide vanes, which Deng et al. already explored).

If the two-phase model indicates that helium bubbles noticeably reduce reactivity in certain zones, this might affect the placement or rate of helium injection (to ensure uniform void distribution or to avoid excessive local reactivity depression). By providing a tool that can virtually "test" these what-if scenarios, this work contributes to a more robust design process.

In terms of operation, the transient simulations will shed light on how quickly the reactor can respond to load changes or how it behaves in off-normal situations (like an inadvertent loss of pump power). These insights directly support the **licensing and safety case** for MSFR technology, as regulators and designers will require demonstrated understanding of such behavior.

---

### **Scientific and Academic Contribution**

From a research perspective, this project pushes the state-of-the-art in multiphysics coupling. Novel findings will be produced on the interplay of turbulence, flow field, and neutronics in a fast-spectrum MSR. The inclusion of **decay heat precursor drift** is especially novel; while delayed neutron transport has been studied, the spatial distribution of decay heat (and its removal) in a flowing-fuel reactor is not well-explored in literature.

The results can reveal, for instance, whether the bulk of decay heat is deposited in the core or carried out (which affects how the passive decay heat removal systems are designed). These findings are anticipated to be published in journals covering reactor physics and thermal-hydraulics.

---

### **Relevance to Industry and Future Reactors**

The timing of this work is advantageous, as interest in MSRs (both fast and thermal) is resurging worldwide. Demonstrating a coupled high-fidelity MSFR simulation aligns with the goals of initiatives like the U.S. ARDP (Advanced Reactor Demonstration Program) and international Gen-IV collaborations.

The methodology could eventually be used by reactor vendors or national labs to perform high-resolution simulations supporting the design of MSR demonstrators. Moreover, the framework established could be extended to related systems, e.g., other molten salt reactor designs (like the one Southern Company is exploring with TerraPower, which involves fast-spectrum chloride salt fuel) or even to solid-fueled reactors that have significant coolant void feedback (boiling reactors).

In essence, the project provides a template for **integrated multiphysics analysis** in nuclear engineering, leveraging modern computational tools to tackle problems that were previously intractable due to their complexity. By validating this approach on the MSFR, confidence is built in applying similar techniques to other advanced reactors, ultimately contributing to improved reactor performance predictions and safety evaluations.

---

> **Summary:** This research will produce a state-of-the-art simulation capability and new knowledge that together reduce the uncertainties in MSFR design and analysis. It directly addresses the Gen-IV goal of improving simulation tools for advanced reactor concepts, and positions the research team at MIT NSE at the forefront of **multiphysics reactor simulation**. The insights gained will help ensure that as MSFR technology progresses, it does so with a deep and quantitative understanding of the coupled phenomena at play, paving the way for safer and more efficient reactor designs.

---

## References

1. D. Zhao, L. Yan, T. Jiang, S. Peng, and B. Yue (2022). *"Multiphysics Simulation Study of the Electrorefining Process of Spent Nuclear Fuel from LiCl-KCl Eutectic Molten Salt."* Journal of The Electrochemical Society, 169(7), 046520.

2. B. Deng, Z. Zhang, Y. Liu, **et al.** (2020). *"Core and blanket thermal–hydraulic analysis of a molten salt fast reactor based on coupling of OpenMC and OpenFOAM."* Nuclear Science and Techniques, 31:85.

3. D. Gérardin, M. Allibert, D. Heuer, A. Laureau, E. Merle-Lucotte, and C. Seuvre (2017). *"Design Evolutions of the Molten Salt Fast Reactor."* Proceedings of the International Conference on Fast Reactors and Related Fuel Cycles (FR17), Yekaterinburg, Russia.

4. E. Cervi, S. Lorenzi, A. Cammi, and L. Luzzi (2019). *"Development of a multiphysics model for the study of fuel compressibility effects in the Molten Salt Fast Reactor."* Chemical Engineering Science, 193, 379–393.

5. M. Dalinger, E. Merzari, S. Lee, and C. Emler (2025). *"High-Fidelity Modelling of the Molten Salt Fast Reactor."* Proceedings of NURETH-21 (21st Int. Topical Meeting on Nuclear Reactor Thermal Hydraulics), Paper 2507.
