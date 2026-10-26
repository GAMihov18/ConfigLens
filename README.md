# ConfigLens

Typed configuration loading, provenance, validation, and safe comparison.

> Planning repository — documentation and Git initialization only. No application, package, solution, database, tests, CI workflow, or runnable demo has been implemented yet. Everything below describes the proposed implementation, unless explicitly marked as verified.

## Goals

Provide a small reusable .NET library that explains where each effective configuration value came from, validates it before use, and produces redacted comparisons between versions.

## Target users

Developers of CLI, desktop, and service applications combining defaults, files, environment variables, and overrides.

## MVP scope

- Load JSON configuration with explicitly ordered default/file/environment/override layers.
- Bind to typed models and return path-specific validation diagnostics.
- Expose per-value source provenance and a redacted effective configuration view.
- Compare two validated configurations and classify changed paths.
- Offer optional Microsoft.Extensions.Configuration integration without requiring a UI or database.

## Stretch goals

- YAML provider as a separate package.
- Atomic file reload with rejected-invalid-update diagnostics.
- A CLI inspector and schema generation for stable model contracts.

## Proposed tech stack

C# / .NET 10, System.Text.Json, optional Microsoft.Extensions.Configuration adapter, xUnit. Start with net10.0; evaluate additional target frameworks based on actual consumers.

Select compatible stable dependency versions during milestone 1, record them in project files, and pin the SDK/toolchain after the compatibility spike. No packages or development workloads are installed by this documentation task.

## Architecture

Core defines path/value/provenance and validation contracts. Providers parse sources into immutable nodes. A deterministic merger applies documented precedence; binding and validation produce either a usable snapshot or diagnostics. Redaction occurs before diagnostic/export formatting. Keep framework adapters separate from the core package.

## Proposed folder structure

Only README.md and .gitignore exist initially, alongside Git metadata. The following tree is a plan; create directories when their implementation begins.

```text
ConfigLens/
  README.md
  .gitignore
  ConfigLens.slnx
  src/ConfigLens/
  src/ConfigLens.Extensions.Configuration/
  tests/ConfigLens.UnitTests/
  tests/ConfigLens.ContractTests/
  samples/ConfigLens.ConsoleDemo/
  docs/semantics/
```

## Data models and invariants

ConfigSource(Name, Priority); ConfigPath(Segments); ConfigValue(Value, Source, IsSecret); ConfigDiagnostic(Path, Code, Severity, Message); ConfigSnapshot<T>(Value, Provenance, Version). Define objects as recursively merged, arrays as replaced, and explicit null as an override that can fail validation. Environment mapping must reject ambiguous key collisions.

## Proposed API and contracts

Proposed API: ConfigLoader.Load<T>(sources, validators) returns ConfigResult<T>; result.Snapshot is available only on success; snapshot.Explain(path) returns origin; ConfigDiff.Compare(left, right, redactionPolicy) returns changed paths. API names are proposals, not available package methods.

Version persisted/exported formats from their first implementation. Validate at boundaries and return structured errors rather than leaking internal exceptions. The contracts above remain proposals until backed by code and contract tests.

## Implementation milestones

1. Specify precedence, array/null behavior, case sensitivity, and secret-path rules.
2. Implement JSON provider, merge, provenance, binding, and validation.
3. Add environment adapter, redacted diff, and console sample.
4. Establish package versioning, compatibility fixtures, and a local package smoke test.

Each milestone should end with a small demonstrable result, a written acceptance check, and tests for its important failure modes. Work through a vertical slice before expanding the stretch scope.

## Testing and CI plan

Test precedence matrices, nested merges, array replacement, null overrides, invalid types, case collisions, secret leakage, and deterministic diffs. CI restores/builds/tests on Windows/Linux, checks public API changes, packs without publishing, and tests a sample consumer against the package artifact.

CI is planned, not configured. Once a solution exists, add .github/workflows/ci.yml with read-only default permissions, explicit toolchain setup, dependency restore, build, and the checks above. Keep ordinary CI offline from real services and hardware. Store test reports as artifacts; do not publish packages, create releases, or deploy from the first workflow. Record hardware/manual results separately from automated passes.

## Setup and development

### Current checkout

Open this README in a Markdown viewer. Git is initialized locally; no build or run step exists yet. Check the working tree with `git status --short --branch`. Documentation files start untracked; no initial commit or remote is created.

### Prerequisites and first implementation

After scaffolding, create a class library and test projects, implement merge semantics before framework integration, and add a sample containing fictional secrets marked for redaction. Pack locally and reference the local package from the console demo before adding application consumers.

Install only the dependencies needed for the selected milestone. Add a checked-in sample configuration with harmless defaults; keep local databases, recordings, generated datasets, credentials, and device identifiers outside version control. Pick a license before public distribution; no license is implied by this plan.

### Future commands

These commands are examples for after the referenced projects/files have been created; they will not build this documentation-only repository.

```powershell
dotnet restore ConfigLens.slnx
dotnet test ConfigLens.slnx
dotnet pack src/ConfigLens -c Release -o artifacts/packages
```

## Supporting libraries and boundaries

First consumers: FlowForge, PulseQueue, SignalBench, and AutoDiag profiles. This library has no dependency on the other portfolio libraries.

Consume stable library versions through local package artifacts initially. Avoid absolute ProjectReference paths into sibling repositories and avoid Git submodules for the MVP. Keep a consumer's first vertical slice runnable using a small local interface or fixture while a supporting package is under development.

## Realistic limitations

Not a secrets vault, encryption system, or replacement for deployment-specific secret management. Provenance can reveal paths and environment names; exported diagnostics must be reviewed. No process-wide mutable configuration singleton or automatic live reload in the MVP.

Scope and performance targets are hypotheses until measured. Document reproducible dataset sizes, hardware, dependency versions, and failure cases alongside any benchmark claim. Do not present planned capabilities as shipped features.

## Documentation provenance

Prepared 2026-10-23. The previous chat supplied the project names and the AutoDiag platform change, but its detailed assistant responses were unavailable content references. The user confirmed the seven proposed application themes in this session; individual designs here are newly proposed plans, not a recovered transcript.

