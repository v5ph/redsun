# Redsun*

Simulating fusion inside of M-Class Stars.

An analysis of the fusion rates within the stellar core of the object.

---

## What this project is, and what it is not

**It is:** an analysis thesis. MESA is an established, validated 1D stellar structure and evolution code. We run it, extract radial profiles, and characterize where fusion happens and how the burning region responds to stellar mass.

**It is not:** a from-scratch stellar structure solver. An earlier version of this plan called for writing a four-equation BVP solver with a hand-rolled equation of state and Gamow-integrated reaction rates. That scope was cut on advisor recommendation as too large for an undergraduate thesis carried alongside full coursework. Do not reintroduce it.

**It is also not:** an N-body or particle-level simulation. See premise P2 below for why that is physically meaningless here.

The contribution is the characterization and the controlled parameter sweeps, not the solver. This should be stated plainly in the thesis introduction rather than left for a committee member to raise.

---

## 3. Scope

**Subject:** M-dwarf stars, roughly 0.2 to 0.6 solar masses.

**Validation case:** one 1 solar-mass run, used to prove the setup is sound before the subject runs are trusted.

**Three questions:**

### Q1. Where inside the star does fusion actually occur?

The question splits two ways and the split is the result:

- The volumetric energy generation rate eps(r) is monotonically decreasing and peaks at r = 0.
- The shell-integrated contribution dL/dr = 4 pi r^2 rho eps peaks at finite radius, because the r^2 geometric factor beats the falling rate for a while before losing to it.

In the Sun that shell peak sits near 0.09 to 0.10 solar radii, so most of the Sun's luminosity originates in a shell rather than at the central point. Determining the analogous structure in M dwarfs is the core of the thesis.

**Deliverable:** eps(r) and dL/dr on a shared radial axis, with enclosed luminosity fraction L(r)/L_total overlaid.

### Q2. What is the power density, and why is it so low?

The solar center generates roughly 276 W/m^3. A human body runs at roughly 1400 W/m^3. The Sun is luminous because it is enormous, not because its core is intense.

M dwarf central temperatures are well below solar, so the pp rate is lower and the power density is smaller still. The result gets stronger, not weaker.

**Deliverable:** central and volume-averaged power density, total reaction rate per unit time, for both the solar validation case and the M-dwarf runs.

### Q3. How does the burning profile change across the convective boundary?

Below roughly 0.35 solar masses, M dwarfs are fully convective top to bottom. Above it, they develop a radiative core with a convective envelope. There is a sharp qualitative structural change in that range and the burning profile looks different on either side of it.

**Deliverable:** a mass sweep across roughly 0.2 to 0.6 solar masses showing the transition, with eps(r), T(r) and the convective zone boundaries plotted per model.

---

## 4. Physical premises

These justify the interpretation of the output. They are facts about stars, not about the code.

**P1. The interior is not uniform in the general case.** Temperature and density fall steeply with radius. Near solar conditions the pp-chain rate scales approximately as rho^2 T^4, so squaring a factor of several in density and raising a factor of ~2 in temperature to the fourth power puts the volumetric burn rate near the core boundary two to three orders of magnitude below its central value. The radial profile is the subject, not a detail to be averaged away.

**P2. Particles do not orbit; they collide constantly.** The ion-ion Coulomb mean free path in a stellar core is on the order of 1e-10 m. A proton at thermal speed suffers an encounter roughly every 1e-16 s. Against a mean waiting time of billions of years for a given proton to fuse, that is upward of 1e30 encounters before one succeeds. The plasma is collisionally locked into local thermodynamic equilibrium with a Maxwell-Boltzmann velocity distribution. Individual trajectories carry no physical meaning. The correct object is a thermally averaged reaction rate against a local Maxwellian, which is what MESA computes.

**P3. Gravity acts through pressure, not through steering.** Gravity does not aim particles at each other. It compresses the gas; compression sets local T and rho; T and rho set the reaction rate. The mediating relation is hydrostatic equilibrium, dP/dr = -G m(r) rho(r) / r^2. Note the density factor; it is easy to drop and the expression is dimensionally wrong without it.

**P4. The barrier is Coulomb repulsion, defeated by tunneling, and the p+p bottleneck is the weak interaction.** Fusion draws from a narrow window far out on the Maxwell tail (the Gamow peak, near 5.9 keV at solar central conditions against kT of about 1.35 keV). Two protons cannot bind: the diproton is unbound. The reaction only completes if one proton converts to a neutron via the weak interaction during the encounter. That is why p+p is slow enough to give stars billion-year lifetimes. Magnetic fields play no meaningful role in core energetics.

**P5. M dwarfs below roughly 0.35 solar masses are fully convective.** Convection mixes the entire star, so composition stays close to uniform and hydrogen is continuously replenished in the burning region rather than depleting locally. This is the one regime where the naive "uniform interior" picture is approximately correct, and it is worth drawing out explicitly in the thesis.

**P6. M dwarfs do not become red giants.** Low-mass M dwarf main-sequence lifetimes exceed the current age of the universe, so none has ever left the main sequence. Forward-in-time modeling is possible but is modeling something that has never happened. Say so plainly rather than implying observational support.

---

## 5. Setup (the blocking task)

Platform is macOS.

1. `xcode-select --install` if not already done.
2. Download the MESA SDK matching the chip (Apple Silicon and Intel are different builds; grabbing the wrong one is the most common wasted hour).
3. Set `MESASDK_ROOT`, source `mesasdk_init.sh`, set `MESA_DIR`, and export `SDKROOT=$(xcrun --sdk macosx --show-sdk-path)`.
4. `cd $MESA_DIR && ./install`

Minimum requirements: macOS or Linux, 64-bit, 8 GB RAM, 20 GB free disk.

**Two known snags:**

- **Gatekeeper quarantine.** macOS flags the downloaded SDK and the compile fails with errors that look like compiler problems but are not. Clear the quarantine attribute on the SDK directory. If `./install` dies early complaining about an unverified or damaged binary, this is it.
- **Shell profile.** Do not source `mesasdk_init.sh` permanently in `.zshrc`. The SDK's `gcc` will shadow the system one for every other project. Put the init in a small script sourced only when working on redsun.

**Fallback if the install fails hard:** MESA-Web (mesa-web.asu.edu) runs models on their hardware with no install. More limited, and the project is mid-transfer to new hosting, so treat it as a backup rather than the plan. It means an install failure cannot kill the thesis.

---

## 6. Build order

Each step is done when its stated condition is met. Do not skip ahead.

1. **Compile MESA.** Done when `./install` prints its success message.
2. **Run an unmodified test case.** `$MESA_DIR/star/test_suite/1M_pre_ms_to_wd`. Do not touch the inlist. Done when profile files appear in `LOGS/`.
3. **Read one profile file in Python.** Use `mesa_reader`. Confirm the expected columns exist (`eps_nuc`, `logT`, `logRho`, `mass`, `radius`, and the `pp` / `cno` rate columns; check `profile_columns.list` for what is actually being written).
4. **Produce the Q1 figure for the solar case.** eps(r) and dL/dr on shared axes. Confirm the shell peak appears near 0.1 solar radii. This is the first real result and the first proof the pipeline works end to end.
5. **Validate.** Compare central temperature, central density, luminosity and radius against the Standard Solar Model (Bahcall, Serenelli and Basu 2005; the BS05(OP) model with Grevesse and Sauval abundances is the preferred one). State agreement as explicit percentages, never as "good agreement." Cross-check total reaction rate against the measured solar neutrino flux at Earth (about 6.5e10 per cm^2 per s), since each pp reaction emits exactly one neutrino. This is a genuinely independent check.
6. **Switch to M dwarfs.** Change the initial mass in the inlist. Start with a single 0.3 solar-mass run.
7. **Mass sweep.** Roughly 0.2 to 0.6 solar masses. Capture the convective transition.
8. **Controlled experiments** (optional, in priority order): toggle electron screening off and rerun to show the effect on central temperature and lifetime; plot pp and CNO contributions against temperature to show the crossover near 17 to 18 MK; vary the pp S-factor within its published uncertainty as a rate sensitivity study.

---

## 7. Repo conventions

Nothing is fixed yet. Suggested layout:

```
redsun/
  PLAN.md           this file
  inlists/          MESA inlists, one directory per run, named by mass
  analysis/         Python. Reads LOGS/, produces figures
  figures/          Generated output. Do not hand-edit
  data/             Reference data (BS05 tables for validation)
  thesis/           Prose draft
```

MESA output directories (`LOGS/`, `photos/`) are large. Add them to `.gitignore`. Commit inlists, not output.

Every run must be reproducible from a committed inlist. If a figure cannot be regenerated from what is in the repo, it does not count.

---

## 8. Thesis draft status

A partial prose draft exists. It was written against the earlier solar-type scope and has known defects:

- The abstract claims the thesis develops a 1D hydrostatic model and couples the four structure equations. **This is now false.** MESA does that. Rewrite the methods sentence. The structure-equation exposition is good and should move into the body as a description of what MESA solves.
- Most specific numbers in the draft are solar (15.7 MK, 5.9 keV Gamow peak, 276 W/m^3, the rho^2 T^4 scaling). They stay only in the validation section. M-dwarf numbers must come from our own models.
- The title refers to a "solar-type" core and is no longer accurate. Leave it as a placeholder; retitle once there are results to name. Candidates point at the convective structure rather than at the subject.
- A citation to Khalid et al. on charged compact stars in f(R,T) gravity is irrelevant to main-sequence stellar structure. Cut it.
- The hydrostatic equation is written without the density factor. Fix.
- The prose says tunneling lets the strong force bind two protons. Add the weak-interaction step (see P4). This is the actual answer to why fusion is slow and it is currently missing.
- The tunneling description says the proton "can exist within the coulomb barrier." Reword to transmission probability rather than existence inside the barrier.
- The source list is written in second person as notes to self. Rewrite as a real bibliography.

**A methods section does not exist yet and is now required.** It must state the MESA version, the inlist settings (mass, metallicity, stopping condition, output cadence), which physics options were left at default, and which were deliberately changed. Without it the work is unreproducible and a reader cannot tell what we did versus what MESA did.

---

## 9. References

- Acharya et al. (2025), "Solar fusion III: New data and theory for hydrogen-burning stars," Rev. Mod. Phys. 97. DOI 10.1103/8lm7-gs18. Current S-factor evaluation; also covers electron screening and opacities. Primary nuclear-physics reference.
- Adelberger et al. (2011), "Solar fusion cross sections II," Rev. Mod. Phys. 83, 195. Prior decadal evaluation. Cite alongside SF III, not instead of it.
- Bahcall, Serenelli and Basu (2005). Standard Solar Model, validation target. Tables at sns.ias.edu/~jnb/SNdata/
- Kippenhahn and Weigert, *Stellar Structure and Evolution*. Standard reference for the four structure equations.
- Christensen-Dalsgaard (2021), "Solar structure and evolution," Living Reviews in Solar Physics. Modern review, good for the literature section.
- MESA instrument papers (Paxton et al., multiple). Required citation for using the code.
- MESA docs: https://docs.mesastar.org/en/26.4.1/

---

## 10. Standing notes for an agent working in this repo

- Do not propose writing a stellar structure solver. That scope was deliberately cut.
- Do not propose N-body or particle-level simulation. See P2.
- Do not reintroduce the red giant phase. M dwarfs do not have one. See P6.
- Prefer finishing the blocking task over planning further ahead. This project has repeatedly re-scoped instead of running a model.
- State validation results as percentages, not adjectives.
