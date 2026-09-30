# Infinics Studio

Working name. This is the base repository for Infinics Studio: business users describe a task to an AI Analyst, which turns it into a working agent, an application, or both, running under their company's workspace.

## Documents

- [`docs/spec/INFINICS_STUDIO_SPEC.md`](docs/spec/INFINICS_STUDIO_SPEC.md) is the product vision, requirements and fit criteria (v1.0).
- [`docs/features/catalog.md`](docs/features/catalog.md) proposes the Catalog feature: a store of solution templates, adapted from Rome's App Store.
- [`docs/evaluation/rome-fit-report.md`](docs/evaluation/rome-fit-report.md) evaluates [`rome-os/rome`](https://github.com/rome-os/rome) as a foundation, using the procedure in section 0 of the spec.

## Status

**Foundation.** The Rome evaluation returns **Take pieces** (100 of 300 weighted points). Rome is MIT-licensed and can be cloned freely. Its core design conflicts with three things the spec needs:
- one user per instance, where the spec needs a company workspace with roles and approval limits
- generated apps as free-form code, where the spec needs declarative parts
- trust in everything inside the container, where the spec needs governance the platform enforces

The recommended path is to build on the stack in section 8 of the spec. Rome's designs are reused where they fit, as listed in section 7 of the report.

**Next step.** Confirm the foundation decision, then scaffold Phase 1 as set out in section 11 of the spec.
