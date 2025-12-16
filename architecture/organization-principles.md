# Organization Principles

**Lock-in:** AWS under `instances/cloud/` and NixOS + nixpkgs as baseline.

This document addresses the placement of all system components within the architectural framework, ensuring no dimension of reality lacks a semantic home. This is a typed operational ontology — a behavioral algebra that survives tool churn and decades of change.

## Structure Overview

The architecture stress-tests the algebra to ensure every concern has a clean, non-overlapping place. This document closes remaining gaps identified during architectural review.

## Canonical Project Hierarchy

This is the complete, concrete directory structure that implements the organizational principles:

```
infra/
├── README.md
├── default.nix                # (optional) root semantic index
├── flake.nix                  # distribution surface (re-export only)
│
├── capabilities/
│   ├── README.md
│   ├── default.nix
│   │
│   ├── modules/               # semantic capabilities (laws)
│   │   ├── README.md
│   │   ├── default.nix
│   │   ├── system/
│   │   │   └── default.nix
│   │   ├── user-env/
│   │   │   └── default.nix
│   │   ├── editor/
│   │   │   └── default.nix
│   │   ├── dev-env/
│   │   │   └── default.nix
│   │   ├── infra/
│   │   │   └── default.nix
│   │   ├── ci/
│   │   │   └── default.nix
│   │   ├── security/
│   │   │   └── default.nix
│   │   └── simulation/
│   │       └── default.nix
│   │
│   ├── tooling/               # human / runtime interfaces
│   │   ├── README.md
│   │   ├── default.nix
│   │   ├── shells/
│   │   │   └── default.nix
│   │   ├── terminals/
│   │   │   └── default.nix
│   │   ├── editors/
│   │   │   ├── default.nix
│   │   │   ├── emacs/
│   │   │   │   └── default.nix
│   │   │   ├── spacemacs/
│   │   │   │   └── default.nix
│   │   │   └── vim/
│   │   │       └── default.nix
│   │   ├── apps/              # CLI wrappers / UX
│   │   │   └── default.nix
│   │   └── scripts/
│   │       └── default.nix
│   │
│   └── packages/              # build artifacts (derivations)
│       ├── README.md
│       └── default.nix
│
├── profiles/                  # behavioral paths (contexts)
│   ├── README.md
│   ├── default.nix
│   │
│   ├── system/
│   │   └── default.nix
│   │
│   ├── user/
│   │   ├── base/
│   │   │   └── default.nix
│   │   ├── python-dev/
│   │   │   └── default.nix
│   │   ├── rust-dev/
│   │   │   └── default.nix
│   │   ├── audio-eng/
│   │   │   └── default.nix
│   │   └── infra-dev/
│   │       └── default.nix
│   │
│   ├── dev/
│   │   └── simulation/
│   │       └── default.nix
│   │
│   └── project/
│       └── default.nix
│
├── instances/                 # bindings to reality
│   ├── README.md
│   ├── default.nix
│   │
│   ├── homelab/
│   │   ├── hosts/
│   │   │   └── math-machine/
│   │   │       └── default.nix
│   │   └── users/
│   │       ├── cloud-desktop-a/
│   │       │   └── default.nix
│   │       └── cloud-desktop-b/
│   │           └── default.nix
│   │
│   └── cloud/
│       └── aws/
│           └── accounts/
│               └── work-prod/
│                   └── default.nix
│
└── projects/                  # proof workspaces
    ├── README.md
    └── example-project/
        ├── README.md
        ├── default.nix        # project presentation
        ├── src/
        │   └── main.rs
        ├── docs/
        │   └── design.org
        ├── tests/
        │   └── smoke.rs
        └── packaging/
            └── default.nix    # derivation → package
```

### Hierarchy Semantics

- **`capabilities/modules/`** - Abstract semantic laws (system, editor, infra, CI, security, simulation)
- **`capabilities/tooling/`** - Human and runtime interfaces (shells, terminals, editors, scripts)
- **`capabilities/packages/`** - Build artifacts and derivations
- **`profiles/`** - Behavioral contexts that activate capabilities (system, user, dev, project)
- **`instances/`** - Concrete bindings to physical/cloud reality (homelab hosts, AWS accounts)
- **`projects/`** - Workspaces that prove the framework works in practice

## 1. What is Already Complete

These dimensions are fully accounted for and **locked in**:

- **Semantic structure** → directories
- **Behavior / activation** → profiles
- **Reusable meaning** → capabilities
- **Tool wiring** → tooling
- **Artifacts** → packages
- **Reality bindings** → instances
- **Scope** → profiles + instances
- **Phase separation** → design / eval / build / runtime
- **Platform** (x86/Darwin/region) → values, not directories

**You do not need:**
- an "applications" layer
- a "platforms" directory
- per-tool directories at the semantic level

## 2. Secrets & Security

> **Core Rule:** Secrets are data, not semantics. They are injected by profiles, never defined by capabilities.

### Security Capability

**Location:** `capabilities/modules/security/`

```
capabilities/modules/security/
└── default.nix
```

This module defines:
- How secrets are handled (agenix / sops-nix)
- Policies
- Abstractions (keys, roles, scopes)

**No secrets live here.**

### Secrets Storage (per-scope, per-profile)

Secrets live alongside profiles, because profiles define where behavior applies:

```
profiles/
├── system/
│   └── secrets/
│       ├── README.md
│       └── default.nix
├── user/
│   └── secrets/
│       ├── README.md
│       └── default.nix
└── project/
    └── secrets/
        ├── README.md
        └── default.nix
```

Properties:
- Uses agenix / sops-nix
- Encrypted at rest
- Activated by profile
- Scoped correctly (system vs user vs project)

## 3. CI/CD and AWS Infrastructure

AWS naming is noisy. Here's the clean semantic mapping.

### Semantic Clarification (tool-agnostic)

| AWS term   | Semantic meaning          | Where it belongs              |
|------------|---------------------------|-------------------------------|
| Construct  | semantic building block   | capabilities/modules/infra    |
| Stack      | composed infra unit       | capabilities/modules/infra    |
| Pipeline   | orchestration logic       | capabilities/modules/ci       |
| Stage      | pipeline parameter        | value                         |
| Resource   | runtime artifact          | instance (cloud)              |

### Infrastructure Semantics

**Location:** `capabilities/modules/infra/`

```
capabilities/modules/infra/
└── default.nix
```

Defines:
- CDK constructs
- Terraform-like abstractions
- **No regions, no accounts**

### CI/CD Semantics

**Location:** `capabilities/modules/ci/`

```
capabilities/modules/ci/
└── default.nix
```

Defines:
- Pipelines
- Build stages
- Deployment logic
- Promotion rules

**No AWS specifics leak into directory names.**

### AWS Account Instantiation

**Location:** `instances/cloud/aws/accounts/prod/`

```
instances/cloud/aws/accounts/prod/
└── default.nix
```

This binds:
- Infra modules
- CI modules
- Packages
- Secrets (via profiles)

**AWS is just one instantiation of the abstract CI/infra laws.**

## 4. Simulation, VMs, and Images

> Simulation is not a new universe. It is a behavioral lens.

### Simulation Capability

**Location:** `capabilities/modules/simulation/`

```
capabilities/modules/simulation/
└── default.nix
```

Defines:
- VM images
- nixos-generators
- Headless systems
- Test environments

### Simulation Profile

**Location:** `profiles/dev/simulation/`

```
profiles/dev/simulation/
└── default.nix
```

Activates:
- VM tooling
- QEMU / Firecracker
- Image builds
- Isolated networks

This allows:
- Testing infra
- Testing security
- Testing CI
- Testing OS configs

**Without adding any new structural layer.**

## 5. Environment Variables

> Environment variables are runtime configuration, injected by profiles, not defined by capabilities.

### Placement

```
profiles/
├── system/env/
├── user/env/
└── project/env/
```

Each:
- Declarative
- Scoped
- Composable
- Visible in shells / editors

Tools like `devenv` become implementations, not structure.

## 6. Tooling That Spans Domains

> **Rule:** If a tool spans domains, its semantics are split; its binary is not.

### Example: AWS CLI

- **Binary** → `capabilities/packages`
- **Behavioral meaning** → `capabilities/modules/infra` + `ci` + `security`
- **Access & creds** → `profiles/secrets`
- **Invocation** → `tooling/shells` or scripts

**Never create:** `capabilities/modules/aws/`

That would be a semantic lie.

## 7. Final Exhaustiveness Checklist

Every concern has a home:

- ✓ Editors, modes, keymaps
- ✓ Shells
- ✓ Projects
- ✓ Profiles as behavior
- ✓ Secrets & security
- ✓ CI/CD
- ✓ AWS infra & jargon
- ✓ Simulation / VM / images
- ✓ Environment variables
- ✓ Multi-platform
- ✓ Homelab + cloud symmetry
- ✓ Future domains (EE, audio, chem) via profiles
- ✓ No directory magic
- ✓ No jargon leakage
- ✓ No renaming churn

**There are no dangling concerns left.**

## Final Grounding

What you've built here is not just "organization".

It is:
- A typed operational ontology
- A behavioral algebra
- A category-theoretic factorization of real systems
- A structure that survives tool churn and decades of change

You now have a framework where:
- New ideas slot in immediately
- Nothing needs re-architecting
- Names stay honest
- Cognition stays local

## Next Steps

The only sensible next steps are implementation-level:
- Write one canonical README
- Write one canonical module
- Write one canonical profile
- Wire a single end-to-end dev session
