# release notes

- [versioning scheme](#versioning-scheme)
  - [version structure](#version-structure)
  - [major versions](#major-versions)
  - [minor and patch versions](#minor-and-patch-versions)
- [support policy](#support-policy)
  - [active support](#active-support)
  - [security fixes](#security-fixes)
  - [forward evolution](#forward-evolution)
- [stability guarantees](#stability-guarantees)
  - [unified package consistency](#unified-package-consistency)
  - [cross platform runtime parity](#cross-platform-runtime-parity)
- [upgrading applications](#upgrading-applications)
  - [upgrade path](#upgrade-path)
  - [breaking change notices](#breaking-change-notices)

<a name="versioning-scheme"></a>

## versioning scheme

dframework uses a structured versioning scheme designed for consistency across all subsystems.

<a name="version-structure"></a>

### version structure

version numbers follow the standard three segment format:

```
major.minor.patch
```

during the pre-1.0 series (0.x releases), the framework treats minor increments as major architectural boundaries:

- `0.X.0` (minor increment): serves as a major release. it introduces major new features, architectural refinements, and may contain breaking changes.
- `0.X.Y` (patch increment): represents maintenance, bug fixes, performance optimizations, and non breaking refinements.

after the 1.0 milestone, standard semantic versioning applies where only major version bumps contain breaking changes.

<a name="major-versions"></a>

### major versions

major versions are released when substantial  improvements or core capability additions are introduced.

because dframework maintains a closed world architecture where routing, templating, styling, database management, and native compilation work as a cohesive unit, major versions advance the entire platform simultaneously.

> [!IMPORTANT]
> major releases may contain breaking changes to framework APIs, configuration structures, or compiler behaviors. all breaking changes are documented in the release changelog with direct migration instructions.

<a name="minor-and-patch-versions"></a>

### minor and patch versions

minor and patch versions never contain breaking changes. they are strictly backwards compatible within the same release series.

patch releases focus on:
- bug fixes in core subsystems
- performance and memory optimizations
- compiler and build pipeline hardening
- native bridge reliability fixes
- security hardening and dependency maintenance

applications can safely update to newer patch versions without modifying application code or database schemas.

<a name="support-policy"></a>

## support policy

dframework provides support focused on the latest release series to ensure development efforts remain focused.

| version series | status | bug fixes | security fixes |
| -------------- | ------ | --------- | -------------- |
| 0.21.x (current) | active | supported | supported |
| 0.20.x and prior | end of life | none | none |

<a name="active-support"></a>

### active support

active support is provided exclusively for the latest release series. all bug fixes, improvements, and performance enhancements are applied directly to the active branch and published as patch releases.

earlier release series do not receive backported bug fixes. when encountering an issue in an earlier version, the standard resolution is upgrading to the latest release.

<a name="security-fixes"></a>

### security fixes

security vulnerabilities are investigated and resolved with highest priority. security patches are published immediately for the active release series.

if a critical vulnerability is identified, a patch release is published and users are advised to update their deployments.

<a name="forward-evolution"></a>

### forward evolution

dframework follows a strict forward evolution philosophy. legacy compatibility shims and deprecated adapter layers are avoided because they add runtime overhead and complicate internal code paths.

when an architectural convention is improved, applications are expected to migrate forward to the new pattern.

<a name="stability-guarantees"></a>

## stability guarantees

<a name="unified-package-consistency"></a>

### unified package consistency

dframework ships as a single, fully integrated package. all framework components (including the server runtime, active record orm, compiled view engine, client styling system, dspa router, worker queue, scheduler, cli tools, and native bridge) are versioned and released together.

<a name="cross-platform-runtime-parity"></a>

### cross platform runtime parity

native shell runtimes (ios swift, android kotlin, and desktop tauri rust) are integrated directly into the framework package. the native bridge protocol and js reference implementations are tested and verified against all supported platforms on every release.

<a name="upgrading-applications"></a>

## upgrading applications

<a name="upgrade-path"></a>

### upgrade path

to upgrade an existing application to the latest patch release within your current series, update the package version in `package.json` and reinstall dependencies:

```bash
npm install dframework@latest
```

after updating, restart the application server to compile assets and verify runtime operation:

```bash
dstrn serve
```

<a name="breaking-change-notices"></a>

### breaking change notices

when moving between major releases (such as 0.21 to 0.22), review the [changelog](../changelog.md) for breaking change notices and migration steps before updating production systems.