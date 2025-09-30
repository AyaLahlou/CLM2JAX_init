# CLM5.0 → JAX 
**Experimental Design, Team Plan, and Methodology**

**Goal:** Translate **all of CLM5.0** modules from Fortran to **JAX**, matched to Fortran outputs and optimized for accelerators, while distilling a **general methodology** for using AI code editors/agents to translate large Earth system models.

---

## 1) Scope & Success Criteria

### 1.1 Components in Scope (CLM5.0)
- **Vegetation & Biogeochemistry:** phenology (CN), photosynthesis/canopy fluxes, allocation/mortality, crop
- **Surface Fluxes & Energy:** surface radiation, turbulent fluxes, canopy air space
- **Hydrology:** soil water, runoff, groundwater coupling
- **Cryosphere:** snow
- **Disturbance & Land Units:** fire; urban, lake, glacier landunits
- **Driver / Time Integration / I/O:** timestep orchestrator, restart state, input adapters, history writers

> **Fidelity Levels (for speed + completeness)**
- **L0 (Skeleton parity):** APIs, states, and control flow mirror CLM; numerical stubs (identity/minimal) where needed; passes interface tests
- **L1 (Numeric parity):** routine-level parity vs golden I/O within tolerances
- **L2 (Science parity):** module-level budgets/diagnostics pass science checks on multi-week runs
- **L3 (Perf):** accelerator-ready; JIT acceptable; scalable to small grid

By Month 6, every component must be **at least L1**, with **core subsystems L2/L3** (vegetation, energy, hydrology, snow). Urban/lake/glacier may be L1→L2 if needed, but **must** compile and participate in the end-to-end step.

### 1.2 Quantitative Acceptance Gates
- **Parity (routine):** max(abs(JAX − F90)) ≤ `abs_tol_i` **and** rRMSE ≤ `rel_tol_i` for each variable
- **Science (module):** energy and water budgets within 1–2% over 30-day runs (site) and 0.5–1% for small grids
- **Performance:** 
  - JIT compile time per major module < 30 s on A100 (initial)  
  - Steady-state step time ≤ 2.0× Fortran CPU baseline for same tile count (target 1.2–1.5×)
- **Software quality:** 90%+ of translated lines covered by tests; CI green; static shapes; no Python data-dependent control flow (use `lax.cond/select/scan`)
---

## 2) AI-Assisted Translation Methodology (Generalizable)
### 2.1 Tools (subject to change) 
- **AI Code Editors:** Cursor/VSCode + Copilot (inline), Codeium (optional)
- **LLMs:** GPT-class model for agentic refactors, docstring synthesis, test generation
- **Agents:** Local “translator” agent (Fortran→NumPy→JAX), “tester” agent (pytest/golden I/O), “perf” agent (profiling hints)
- **Static/Build:** fparser/fortls to expose Fortran call graphs; clang-format/black/ruff; pyright/mypy for types
- **Data:** xarray/zarr to store **golden I/O** fixtures

### 2.2 Repeatable Translation Cell (per routine)
1. **Scope & Extract**  
   - Identify routine + signatures + common blocks + precision kinds  
   - Export golden inputs/outputs (single-step and multi-step)
2. **Fortran → NumPy (literal)**  
   - Use AI editor to draft line-by-line NumPy port; preserve loop structure and branches verbatim  
   - Enforce `float64`; assert shapes/units at entry
3. **Parity Test (NumPy vs Golden)**  
   - Pytest fixture loads golden I/O; assert tolerances; add edge cases
4. **NumPy → JAX**  
   - Swap to `jax.numpy`; replace loops with `vmap`/`lax.scan` conservatively  
   - Replace early returns with masks; add `where` for safe divisions/roots
5. **JIT & Vectorization**  
   - `jit` with `static_argnames`; choose vmap axis (tile/PFT)  
   - Re-run parity
6. **Doc & Metadata**  
   - Auto-generate docstring from Fortran comments; log quirks (indexing, units, dims)  
   - Commit with “porting notes” template
  
### 2.3 Guardrails Against Hallucination
- Always diff AI output against Fortran logic (manual + tests) before acceptance
- Enforce **ports first, optimizations later**
- Reject hidden state, implicit type promotion, silent broadcasting
- Require **explanatory comments** where AI made non-obvious choices
- Track “AI-assisted changes” label in PRs

### 2.4 Methodology Artifact (for paper/tutorial)
- A **checklist** capturing the 6-step cell, prompt templates, pitfalls catalog, and metrics tables
- A small **reference module** (e.g., CNPhenology) translated three ways (manual, AI-assisted literal, AI-assisted optimized) with time/quality comparisons

## 5) Data, Tests, and Tolerances

- **Golden I/O:** stored as zarr per routine; includes multiple regimes (cold/warm/wet/dry)
- **Tolerance Registry:** YAML mapping variable → abs/rel tolerances; versioned with tests
- **Budgets:** standardized notebook computes E/W closure, runoff partitioning, snow mass balance
- **CI:** run unit tests, parity tests on small fixtures, budget checks, JIT timing; nightly small-grid job

---

## 6) Risk & Mitigation

- **Ambitious Scope:** Use L0→L1 staging to keep whole-model compiling by Week 8  
- **AI Drift/Hallucination:** strict test-first acceptance; “ports before optimize”; reviewer checklists  
- **Performance Regressions:** CI perf gates + nightly trend plots  
- **Autodiff Breakage:** early smoke tests, `custom_vjp` wrappers catalog

---

## 7) Documentation & Deliverables

- **Code:** `clmjax/` modules by subsystem; typed, pure functions
- **CLI:** `clmjax step` (single), `clmjax run` (multi-day), `clmjax validate` (parity/budgets)
- **Docs (MkDocs):** 
  - *User Guide:* run configs, inputs/outputs  
  - *Developer Guide:* translation cell, dims/units, porting notes  
  - *Methodology:* prompts, guardrails, time/quality benchmarks, lessons learned
- **Artifacts:** golden fixtures; validation notebooks; perf dashboards
- **Release:** v0.9 (Month 6), preprint on AI-assisted translation methodology

