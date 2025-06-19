# Dotnet Cut Tag

![Latest Release](https://img.shields.io/github/v/release/p6m-actions/dotnet-cut-tag?style=flat-square&label=Latest%20Release&color=blue)

## Description

A GitHub Action for cutting versioned tags for .NET projects. This action will:

1. Bump the version in `Directory.Build.props` based on the version level
2. Create a Git tag using the format "vMajor.Minor.Patch"
3. Push the new version and tag to the repository

This action is designed for .NET projects that use MSBuild's Directory.Build.props file with a `<VersionPrefix>` element for centralized version management.

## Usage

Add this action to your workflow after checking out your code:

```yaml
- name: Cut Tag
  uses: p6m-actions/dotnet-cut-tag@v1
  with:
    version-level: "patch" # Options: patch, minor, major
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version-level` | Version level to bump (patch, minor, or major) | No | `patch` |
| `directory-build-props-path` | Path to Directory.Build.props file | No | `Directory.Build.props` |

## Outputs

| Output | Description |
|--------|-------------|
| `version` | The new version number |
| `tag` | The new tag that was created |

## Examples

### Build and Release Workflow

```yaml
name: Build and Release

on:
  push:
    branches: [main]

permissions:
  contents: write

jobs:
  build-and-release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'

      - name: Build
        run: dotnet build

      - name: Test
        run: dotnet test

      - name: Cut Tag
        uses: p6m-actions/dotnet-cut-tag@v1
        with:
          version-level: "patch"
```

### Manual Release Workflow

```yaml
name: Release New Version

on:
  workflow_dispatch:
    inputs:
      version-level:
        description: "Version level to bump"
        required: true
        type: choice
        options:
          - patch
          - minor
          - major
        default: patch

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Cut Tag
        id: cut-tag
        uses: p6m-actions/dotnet-cut-tag@v1
        with:
          version-level: ${{ inputs.version-level }}

      - name: Output Results
        run: |
          echo "New version: ${{ steps.cut-tag.outputs.version }}"
          echo "Created tag: ${{ steps.cut-tag.outputs.tag }}"
```

### Custom Directory.Build.props Path

```yaml
- name: Cut Tag
  uses: p6m-actions/dotnet-cut-tag@v1
  with:
    version-level: "minor"
    directory-build-props-path: "src/Directory.Build.props"
```

## Prerequisites

Your repository must have a `Directory.Build.props` file with a `<VersionPrefix>` element:

```xml
<?xml version="1.0"?>
<Project>
    <PropertyGroup>
        <VersionPrefix>1.0.0</VersionPrefix>
        <TargetFramework>net8.0</TargetFramework>
        <!-- other properties -->
    </PropertyGroup>
</Project>
```

## How It Works

1. **Version Extraction**: The action parses the current version from the `<VersionPrefix>` element in Directory.Build.props
2. **Version Bumping**: Based on the `version-level` input:
   - `patch`: Increments the patch version (1.0.0 → 1.0.1)
   - `minor`: Increments the minor version and resets patch (1.0.1 → 1.1.0)
   - `major`: Increments the major version and resets minor and patch (1.1.0 → 2.0.0)
3. **File Update**: Updates the Directory.Build.props file with the new version
4. **Git Operations**: Commits the version change and creates a new tag (e.g., `v1.0.1`)
5. **Push**: Pushes both the commit and the new tag to the repository