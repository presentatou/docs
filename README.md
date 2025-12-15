# Documentation

This repository contains architectural and operational documentation for the NixOS-based system framework.

## Architecture

The architecture is built on NixOS + nixpkgs as the baseline, with AWS as the cloud provider under `instances/cloud/`.

### Key Documents

- **[Organization Principles](architecture/organization-principles.md)** - Core architectural principles defining where every system component belongs. This is the foundational document that establishes the typed operational ontology for the entire system.

## Framework Overview

The system is organized around several key dimensions:

- **Semantic structure** → directories
- **Behavior / activation** → profiles
- **Reusable meaning** → capabilities
- **Tool wiring** → tooling
- **Artifacts** → packages
- **Reality bindings** → instances

This creates a framework where:
- New ideas slot in immediately
- Nothing needs re-architecting
- Names stay honest
- Cognition stays local

## Structure

```
├── architecture/          # Architectural documentation and principles
│   └── organization-principles.md
```

## Status

This is a living documentation repository that captures the operational ontology and organizational principles of the system. The architecture is complete and locked in; implementation-level work is the next phase.
