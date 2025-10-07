# CLM5.0 → JAX 
**Experimental Design, Team Plan, and Methodology**

**Goal:** Translate the Community Land Model 5.0 (CLM5.0) from Fortran to JAX while developing and validating a generalizable, replicable methodology for AI-assisted translation of large-scale scientific software.

### Research Questions:

- Can AI code assistants reliably translate complex scientific Fortran to modern Python/JAX?
- What translation patterns succeed vs. fail with current AI tools?
- What guardrails and validation strategies ensure numerical correctness?
- How does AI-assisted translation compare to manual translation in speed, quality, and error patterns?

### Deliverables:

- Complete JAX implementation of CLM5.0 with numerical parity
- Documented methodology applicable to other scientific models
- Empirical analysis of AI tool performance on scientific code translation
- Open-source artifacts: translated code, test fixtures, validation framework


## 1. Project Scope & Success Criteria
### 1.1 CLM5.0 Components (Translation Targets)
CLM5.0 is a comprehensive land surface model consisting of ~300K lines of Fortran90 across multiple coupled subsystems:
#### Core Physical Processes:

- Biogeophysics: Surface energy balance, turbulent fluxes, canopy radiative transfer, soil heat transport
- Hydrology: Soil moisture dynamics, runoff generation, groundwater coupling, infiltration
- Cryosphere: Snow accumulation/melt, snow albedo, thermal properties
- Vegetation Dynamics: Photosynthesis, stomatal conductance, phenology (CN-based)
- Biogeochemistry: Carbon-nitrogen cycling, allocation, mortality, decomposition

#### Specialized Components:

- Crop Model: Prognostic crop growth and management
- Fire Model: Fire occurrence, spread, and emissions
- Land Units: Urban, lake, glacier, wetland representations
- Disturbance: Land use change, harvest

#### Infrastructure:

- Driver/Orchestration: Time integration, component coupling
- I/O System: NetCDF readers/writers, restart/history files
- Spatial Framework: Grid decomposition, subgrid hierarchy (gridcell→landunit→column→PFT)

### 1.2 Fidelity Levels (Incremental Quality Gates)
To manage complexity and ensure continuous progress, we define four fidelity levels:
#### L0 (Structural Parity):

APIs mirror Fortran interfaces (function signatures, argument order)
State variables defined with correct shapes and types
Control flow skeleton implemented (loops, conditionals, module calls)
Numerical stubs used where implementations are incomplete
Test: Runs without crashing, produces output of correct shape
Timeline: All modules L0 by Week 4

#### L1 (Routine-Level Numerical Parity):

Each translated routine produces outputs matching Fortran within specified tolerances
Tests against golden I/O fixtures (single timestep)
Edge cases handled (zeros, negatives, extremes)
Test: max(|JAX - F90|) ≤ abs_tol AND rRMSE ≤ rel_tol for all outputs
Timeline: All modules L1 by Week 12

#### L2 (Module-Level Scientific Validity):

Multi-timestep simulations conserve mass and energy
Budget closures verified (water balance, energy balance, carbon balance)
Physically plausible outputs under varied forcing conditions
Test: 30-day single-site runs with budget errors < 1-2%
Timeline: Core modules (vegetation, hydrology, energy, snow) L2 by Week 18

#### L3 (Production Ready):

JIT compilation succeeds with reasonable compile times
Vectorization across spatial dimensions (tiles, PFTs)
Performance competitive with Fortran (target: 1.2-1.5× CPU baseline)
Autodifferentiable (where scientifically appropriate)
Test: Small grid (e.g., 96×144) runs at acceptable speed
Timeline: Core modules L3 by Week 24


#### Software Quality:

Test coverage: ≥ 90% of translated lines
CI passing: All tests green on merge to main
Type safety: Full type annotations, passes mypy strict mode
No dynamic control flow: All conditionals use lax.cond, lax.select, or where
No data-dependent shapes: Static shapes enforced

---

## 2. AI-Assisted Translation Methodology
### 2.1 Tool Ecosystem

#### AI Code Assistants:
Primary: Cursor IDE / GitHub Copilot (inline suggestions, chat-based refactoring)
Secondary: Claude Code (CLI agent for batch transformations)
Experimental: GPT-4/Claude-3.5 via API (custom translation agents)

#### Static Analysis & Parsing:

fparser2: Python-based Fortran parser for AST extraction
fortls: Fortran Language Server (call graph analysis, dependency tracking)
fypp: Fortran preprocessor handling (macro expansion)

#### Testing & Validation:

pytest: Unit and integration testing framework
hypothesis: Property-based testing for edge cases
xarray: Multi-dimensional labeled arrays (golden I/O storage)
zarr: Compressed array storage (golden fixtures)

#### Code Quality:

black: Python code formatting
ruff: Fast Python linter
mypy: Static type checking
pre-commit: Automated quality checks

#### Performance:

JAX profiler: Kernel timing, memory usage
TensorBoard: Timeline visualization
py-spy: Sampling profiler for Python overhead
  
### 2.2 Core Translation Protocol (Per Routine)
Each Fortran routine follows this six-step process:
#### Step 1: Scope & Extract (Manual + AI)
<pre> ```
Inputs:
  - Fortran source file(s)
  - Module dependencies (from fortls)
  
Actions:
  1. Identify routine signature (arguments, intent, dimensions)
  2. Map Fortran types to NumPy dtypes (REAL(r8) → float64)
  3. Document implicit inputs (COMMON blocks, module variables)
  4. Extract inline documentation and citations
  5. Generate golden I/O fixtures:
     - Single timestep: varied inputs covering parameter space
     - Edge cases: zeros, negatives, bounds
     - Multi-timestep: 10-step sequences for state evolution
  
Outputs:
  - Routine specification document (signatures, dimensions, units)
  - Golden I/O zarr dataset
  - Dependency graph snippet ``` </pre>
  
  
AI Role: Minimal (AST parsing, docstring extraction)
Human Role: High (domain knowledge, edge case selection)

#### Step 2: Fortran → NumPy (Literal Translation)

<pre> ```
  Inputs:
  - Fortran source
  - Routine specification
  
Actions:
  1. AI generates line-by-line NumPy translation
  2. Preserve original loop structure (don't vectorize yet)
  3. Preserve branching logic verbatim
  4. Convert 1-based → 0-based indexing
  5. Enforce float64 throughout
  6. Add shape assertions at function entry
  7. Document translation choices in inline comments
  
Prompt Template:
  "Translate this Fortran routine to NumPy, preserving the exact
   computational structure. Do not optimize or vectorize. Use explicit
   loops matching the Fortran DO loops. Maintain 1:1 correspondence
   between Fortran lines and NumPy lines where possible."
  
Outputs:
  - NumPy .py file with extensive inline comments
  - Translation notes (ambiguities, assumptions) ``` </pre>
  
AI Role: High (mechanical translation)
Human Role: Medium (review for semantic equivalence)

#### Step 3: Parity Test (NumPy vs. Golden)
<pre>``` Inputs:
  - NumPy implementation
  - Golden I/O fixtures
  
Actions:
  1. Generate pytest test cases from fixtures
  2. Run NumPy function with golden inputs
  3. Compare outputs against golden outputs
  4. Assert tolerances from registry
  5. Debug failures:
     - Check indexing errors (off-by-one)
     - Verify operator precedence
     - Check initialization values
  6. Add edge case tests
  
Test Template:
  @pytest.mark.parametrize("fixture", golden_io_loader("routine_name"))
  def test_numpy_parity(fixture):
      outputs = numpy_routine(**fixture.inputs)
      for var, expected in fixture.outputs.items():
          np.testing.assert_allclose(
              outputs[var], expected,
              atol=TOLERANCE_REGISTRY[var].abs,
              rtol=TOLERANCE_REGISTRY[var].rel
          )
  
Outputs:
  - Passing test suite (or debugged until passing)
  - Coverage report``` </pre>
  
AI Role: Low (test generation only)
Human Role: High (debugging, tolerance tuning)
#### Step 4: NumPy → JAX (Conservative JAX-ification)
<pre>``` Inputs:
  - Validated NumPy implementation
  
Actions:
  1. Replace numpy → jax.numpy
  2. Replace explicit loops with vmap/scan (only where clearly vectorizable)
  3. Convert early returns → masked operations:
     - if/else → jnp.where()
     - return → update masked variable
  4. Protect divisions/roots: jnp.where(denom != 0, num/denom, fallback)
  5. Remove in-place updates (x[i] += y → x = x.at[i].add(y))
  6. Ensure all arrays are jax.Array
  
Prompt Template:
  "Convert this NumPy code to JAX. Replace loops with vmap/scan only
   where the loop iterations are independent. Convert all conditionals
   to functional form using jnp.where or lax.cond. Do not change the
   computational logic."
  
Outputs:
  - JAX implementation
  - Vectorization notes (which loops converted, which preserved)``` </pre>
  
AI Role: Medium (mechanical JAX idioms)
Human Role: High (verify semantic preservation)
#### Step 5: JIT & Vectorization
<pre> ```Inputs:
  - JAX implementation passing NumPy parity tests
  
Actions:
  1. Add @jit decorator with static_argnames for shape params
  2. Test JIT compilation (check for tracer errors)
  3. Choose vectorization axis (typically 'tiles' or 'pfts')
  4. Add vmap for spatial dimensions
  5. Profile compilation time
  6. Re-run parity tests with JIT
  7. Verify determinism (multiple runs → identical outputs)
  
Optimization Targets:
  - Compile time < 30s
  - No Python overhead in inner loops
  - Memory layout friendly to XLA
  
Outputs:
  - JIT-compiled JAX implementation
  - Performance profile (compile time, memory usage)``` </pre>
  
AI Role: Low (boilerplate JIT decorators)
Human Role: High (performance tuning)
#### Step 6: Documentation & Metadata
<pre> ```Inputs:
  - Completed JAX implementation
  - Original Fortran comments
  - Translation notes from steps 2-4
  
Actions:
  1. Generate comprehensive docstring:
     - Purpose and scientific context
     - Arguments (name, shape, units, physical meaning)
     - Returns (name, shape, units, physical meaning)
     - Algorithm description
     - References to papers/documentation
  2. Document deviations from Fortran:
     - Indexing changes
     - Loop→vmap conversions
     - Conditional→where conversions
  3. Add type hints (full signature annotation)
  4. Update translation log:
     - Time spent per step
     - AI vs. human effort breakdown
     - Number of bugs caught in testing
  
Output:
  - Fully documented JAX module
  - Porting notes in docs/ directory``` </pre>
  
AI Role: High (docstring generation from comments)
Human Role: Medium (verify accuracy, add context)

### 2.3 Guardrails Against AI Hallucination
Problem: LLMs may generate plausible-looking code that is semantically incorrect, especially for scientific code with domain-specific conventions.
Mitigation Strategies:

Golden I/O as Ground Truth:

Every routine has comprehensive test fixtures extracted from validated Fortran runs
Tests run automatically on every commit
No code merged without passing parity tests


Ports Before Optimizations:

Step 2 (NumPy) must be literal translation before Step 4 (JAX optimization)
Forbid AI from "improving" algorithms during initial translation
Optimizations only after parity is established


Mandatory Human Review:

Every AI-generated routine reviewed line-by-line against Fortran
Reviewer checklist includes:

 Indexing correct (0-based vs 1-based)
 Loop bounds match Fortran
 Conditional logic equivalent
 Physical units preserved
 Edge cases handled (div-by-zero, sqrt of negative, etc.)




Explanatory Comments Required:

AI must explain non-obvious translation choices
Red flag: "simplified the logic" → requires deep review
Human adds comments for domain-specific knowledge


Incremental Validation:

Test at multiple scales: single routine → module → subsystem → full model
Budget checks at each level (mass/energy conservation)


AI Assistance Tracking:

Git commit messages tag AI-assisted changes: [AI-ASSISTED: Cursor]
PR templates include "AI tool used" and "validation method"
Enables post-hoc analysis of AI-introduced bugs


Type System Enforcement:

Full type annotations required (mypy --strict)
Shape checking with jax.ShapeDtypeStruct
No silent type coercion



Red Flags Triggering Extra Scrutiny:

AI changes loop iteration order
AI combines/splits conditionals
AI introduces new variables not in Fortran
AI uses different numerical methods
Comments like "optimized" or "cleaned up"
