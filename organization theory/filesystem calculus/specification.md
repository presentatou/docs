# File System Calculus (FSC)

**Version:** v0.1
**Status:** Stable core invariants defined

---

## 0. Purpose

Encode program semantics, composition, and effects structurally in the filesystem.

- The directory tree is the proof.
- Navigating the tree is equivalent to reading a typed, sequential composition.

---

## 1. Core Objects

### 1.1 Directory

A directory represents a typed transformation (a morphism).

Formal judgment:

```
Γ ⊢ D : A → B ▷ ε
```

- `Γ` = local context (what is in scope)
- `A` = input type(s)
- `B` = output type(s)
- `ε` = effect set (possibly empty)

---

### 1.2 Index File (default.sig)

Every semantic directory must contain exactly one index file:

```
default.sig
```

It declares the local typing judgment:

```
input   : A
output  : B
effects : ε
```

**Rules:**
- No semantic meaning may live in filenames
- Filenames are indices only
- All semantics are carried by directories + default.sig

---

## 2. Composition Rules

### 2.1 Sequential Composition

Subdirectories compose in lexical order.

If:

```
D/
├── f/
│   └── default.sig   (A → B ▷ ε₁)
└── g/
    └── default.sig   (B → C ▷ ε₂)
```

Then:

```
D : A → C ▷ (ε₁ ∪ ε₂)
```

---

### 2.2 No Implicit Wiring

- No registries
- No service locators
- No global environment objects
- Composition is structural, not dynamic

**The filesystem is the wiring.**

---

## 3. Effects System

### 3.1 Effect Definition

Effects represent references outside the pure function.

Minimal effect taxonomy (extensible):
- `Env.Read`
- `Storage.Read`
- `Storage.Write`
- `Network.Call`
- `Network.Listen`
- `External.API`
- `Telemetry.Emit`
- `Callback.Invoke`
- `RNG`
- `Clock`

---

### 3.2 Effect Locality Rule (Critical)

Effects must live at the narrowest possible scope.

Effects are declared in:

```
<step>/
└── effects/
    └── <effect>.sig
```

**Rules:**
- ❌ No top-level effects/ for an entire pipeline
- ❌ No global effect declarations
- ✅ Effects belong to edges, not whole phases

This makes where mutability happens visible by inspection.

---

### 3.3 Effect Visibility Rule

A directory may only declare effects that:
- it uses directly, or
- are required by its immediate children

**No effect smuggling.**

---

## 4. Purity vs Effectfulness

### 4.1 Pure Directories
- No effects/ subdirectory
- Can be reasoned about in isolation
- Refer only to inputs and child outputs

### 4.2 Effectful Directories
- Explicit effects/ subdirectory
- Effect kind and scope are visible
- Still typed and compositional

---

## 5. Phases, Modes, and Locality

### 5.1 Phases = Sequential Composition

"Phase" is not a label — it is encoded by the position in the pipeline.

Examples:
- `extract → normalize → enrich → aggregate`
- `observe → decide → evaluate → update`

**The directory structure is the phase ordering.**

---

### 5.2 Modes vs Phases

- **Modes** = external selection (sum types)
- **Phases** = internal structure (composition)

Pattern:

```
app/
├── modes/
│   ├── train/
│   ├── validate/
│   └── infer/
```

Modes select which pipeline to run.
They do not define pipeline semantics.

---

## 6. Environment & Configuration

### 6.1 Environment Variables

Environment access is an effect:

```
effects/
└── env.read.sig
```

**Rules:**
- Config is input, not global state
- Only input boundaries may read environment
- Downstream steps consume resolved values

This enforces Reader-style semantics structurally.

---

### 6.2 Profiles (External to Project)

Profiles:
- provide effects (Env, Storage, Network)
- do not live inside projects
- satisfy effect requirements

Typing rule:

```
Project.effects ⊆ Profile.provides
```

This is a static validity check.

---

## 7. Projects

### 7.1 What a Project Is

A project is a category of representations and transformations,
parameterized by an external environment.

A project contains:
- domain types
- pipelines (typed composition)
- explicit effect boundaries
- artifacts produced
- an application surface (CLI)

---

### 7.2 Project Grammar

```
Project ::=
  default.sig
  input/
  (pipeline)+
  output/
  app/
  packaging/
```

---

## 8. Artifacts

Artifacts are:
- outputs of pipelines
- data, models, metrics, checkpoints
- never logic

They live in:

```
artifacts/
```

They are not inputs unless explicitly re-introduced via a typed step.

---

## 9. Application (CLI)

The application:
- lives inside the project initially
- selects modes
- parses flags
- provides UX only

It may not:
- implement business logic
- perform domain effects
- bypass the pipeline

**CLI is an interface, not a semantic layer.**

---

## 10. Packaging

Packaging is the terminal morphism:

```
packaging/default.nix
```

It closes the project into a package.

**Rules:**
- No new semantics
- No new effects
- Just closure

---

## 11. Naming Rules (Hard Constraints)

1. Directories encode semantics
2. Files are indices only
3. If you can't derive the type from the tree, the tree is invalid
4. Effects must be visible by path
5. No cross-phase imports
6. No "misc", "utils", or "helpers"
7. Illegal states must be unrepresentable

---

## 12. Verification by Inspection

By running:

```bash
tree project/
```

You must be able to answer:
- What are the inputs?
- What are the outputs?
- Where does IO happen?
- Which steps are pure?
- What environment is required?
- What profile capabilities are needed?
- Where can mutation occur?

**If not, the design fails FSC.**

---

## 13. What FSC Gives You

- Structural type checking
- Effect tracking without code
- Business requirements as types
- Zero architectural refactors
- Proof-carrying project layouts
- Domain-agnostic reuse

---

## 14. Status & Next Versions

**v0.1 defines:**
- core calculus
- effect system
- composition rules
- project grammar

**Possible future extensions:**
- default.sig DSL formalization
- FSC linter
- Visualization tools
- Mapping to HoTT identity paths
- Mapping to Nix flakes mechanically

---

## Final One-Line Summary

**FSC makes the filesystem a typed, effect-aware proof of program behavior.**
