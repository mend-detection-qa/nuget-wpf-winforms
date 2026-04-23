# nuget-wpf-winforms

## Feature exercised

Standard NuGet PackageReference with a Windows-specific target framework (`net8.0-windows`) and `<UseWPF>true</UseWPF>`, exercising Mend's ability to detect packages declared in a `.csproj` and resolved via a `packages.lock.json` lockfile under a Windows TFM.

## Probe metadata

| Field          | Value                                                      |
|----------------|------------------------------------------------------------|
| Pattern        | windows-tfm-wpf-packagereference                           |
| Target PM      | NuGet                                                      |
| Target TFM     | net8.0-windows                                             |
| Lock file      | packages.lock.json (NuGet v2 format)                       |
| Direct deps    | 4                                                          |
| Transitive deps| 3                                                          |
| Purpose        | Validate Mend detects WPF packages via PackageReference under Windows TFM with explicit lock file |

## Expected dependency tree

The lock file lists dependencies under the `net8.0-windows7.0` key (the canonical TFM key NuGet uses for `net8.0-windows`).

### Direct dependencies

| Package | Version | Notes |
|---------|---------|-------|
| CommunityToolkit.Mvvm | 8.2.2 | MVVM toolkit; brings Microsoft.Extensions.DependencyInjection.Abstractions and System.ComponentModel.Annotations |
| MaterialDesignThemes | 5.1.0 | WPF UI library; brings MaterialDesignColors as transitive |
| Newtonsoft.Json | 13.0.3 | No runtime transitives |
| Microsoft.Extensions.DependencyInjection | 8.0.0 | DI container; brings Microsoft.Extensions.DependencyInjection.Abstractions |

### Transitive dependencies

| Package | Version | Brought by |
|---------|---------|------------|
| MaterialDesignColors | 3.1.0 | MaterialDesignThemes |
| Microsoft.Extensions.DependencyInjection.Abstractions | 8.0.0 | CommunityToolkit.Mvvm, Microsoft.Extensions.DependencyInjection |
| System.ComponentModel.Annotations | 5.0.0 | CommunityToolkit.Mvvm |

### Mend detection expectations

- Mend should detect all 4 direct packages as top-level nodes.
- `MaterialDesignColors` 3.1.0 must appear as a child of `MaterialDesignThemes` 5.1.0.
- `Microsoft.Extensions.DependencyInjection.Abstractions` 8.0.0 must appear as a child of both `CommunityToolkit.Mvvm` and `Microsoft.Extensions.DependencyInjection` (deduplication may result in one canonical node).
- `System.ComponentModel.Annotations` 5.0.0 must appear as a child of `CommunityToolkit.Mvvm`.
- All `dependencyType` values must be `NUGET`.
- `dependencyFile` must reference the `packages.lock.json` path.
- No packages should be missing or mis-versioned.
- The Windows-specific TFM must not prevent detection (Mend resolver must parse the `net8.0-windows7.0` bucket).
