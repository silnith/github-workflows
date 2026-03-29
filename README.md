# GitHub Reusable Workflows

Standard GitHub workflows that use best practices for building and testing software.

## .NET

```yaml
jobs:
  get-date:
    name: Date
    uses: silnith/github-workflows/.github/workflows/bash-get-date.yaml@main
  dotnet-cache:
    name: .NET Cache
    needs:
      - get-date
    uses: silnith/github-workflows/.github/workflows/dotnet-cache-dependencies.yaml@main
    with:
      directory: .
      solution-file: foo.sln
      dotnet-versions: 5.x
      year: ${{ needs.get-date.outputs.year }}
      month: ${{ needs.get-date.outputs.month }}
      day: ${{ needs.get-date.outputs.day }}
    secrets: inherit
    permissions:
      contents: read
  dotnet-build:
    name: .NET
    needs:
      - get-date
      - dotnet-cache
    strategy:
      matrix:
        configuration:
          - Debug
          - Release
    uses: silnith/github-workflows/.github/workflows/dotnet-build.yaml@main
    with:
      artifact-prefix: csharp
      directory: .
      solution-file: foo.sln
      configuration: ${{ matrix.configuration }}
      dotnet-versions: 5.x
      year: ${{ needs.get-date.outputs.year }}
      month: ${{ needs.get-date.outputs.month }}
      day: ${{ needs.get-date.outputs.day }}
    secrets: inherit
    permissions:
      contents: read
  dotnet-test-results:
    name: .NET Publish Test Results
    needs:
      - dotnet-build
    uses: silnith/github-workflows/.github/workflows/dotnet-publish-test-results.yaml@main
    with:
      artifact-prefix: csharp
      directory: .
    secrets: inherit
    permissions:
      checks: write
```

## Java

```yaml
jobs:
  get-date:
    name: Date
    uses: silnith/github-workflows/.github/workflows/bash-get-date.yaml@main
  maven-cache:
    name: Maven Cache
    needs:
      - get-date
    uses: silnith/github-workflows/.github/workflows/maven-cache-dependencies.yaml@main
    with:
      directory: .
      year: ${{ needs.get-date.outputs.year }}
      month: ${{ needs.get-date.outputs.month }}
      day: ${{ needs.get-date.outputs.day }}
    secrets: inherit
    permissions:
      contents: read
  maven-verify:
    name: Maven
    needs:
      - get-date
      - maven-cache
    uses: silnith/github-workflows/.github/workflows/maven-verify.yaml@main
    with:
      artifact-prefix: java
      directory: .
      year: ${{ needs.get-date.outputs.year }}
      month: ${{ needs.get-date.outputs.month }}
      day: ${{ needs.get-date.outputs.day }}
    secrets: inherit
    permissions:
      contents: read
      checks: write
  maven-deploy:
    name: Maven Deploy
    needs:
      - get-date
      - maven-cache
      - maven-verify
    uses: silnith/github-workflows/.github/workflows/maven-deploy.yaml@main
    with:
      artifact-prefix: java
      directory: .
      year: ${{ needs.get-date.outputs.year }}
      month: ${{ needs.get-date.outputs.month }}
      day: ${{ needs.get-date.outputs.day }}
    secrets: inherit
    permissions:
      contents: read
      packages: write
```
