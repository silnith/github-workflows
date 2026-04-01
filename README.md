# GitHub Reusable Workflows

Standard GitHub workflows that use best practices for building and testing software.

## C++

```yaml
jobs:
  get-date:
    name: Date
    uses: silnith/github-workflows/.github/workflows/bash-get-date.yaml@main
  cpp-nuget:
    name: NuGet Cache
    needs:
      - get-date
    uses: silnith/github-workflows/.github/workflows/nuget-cache-dependencies.yaml@main
    with:
      directory: .
      solution-file: foo.sln
      year: ${{ needs.get-date.outputs.year }}
      month: ${{ needs.get-date.outputs.month }}
      day: ${{ needs.get-date.outputs.day }}
    secrets: inherit
    permissions:
      contents: read
  cpp-build-intel:
    name: C++ Intel
    needs:
      - get-date
      - cpp-nuget
    strategy:
      matrix:
        platform:
          - x86
          - x64
        configuration:
          - Debug
          - Release
    uses: silnith/github-workflows/.github/workflows/msbuild-intel.yaml@main
    with:
      artifact-prefix: cpp
      directory: .
      solution-file: foo.sln
      platform: ${{ matrix.platform }}
      configuration: ${{ matrix.configuration }}
      year: ${{ needs.get-date.outputs.year }}
      month: ${{ needs.get-date.outputs.month }}
      day: ${{ needs.get-date.outputs.day }}
    secrets: inherit
    permissions:
      contents: read
  cpp-build-arm:
    name: C++ ARM
    needs:
      - get-date
      - cpp-nuget
    strategy:
      matrix:
        platform:
          - ARM
          - ARM64
        configuration:
          - Debug
          - Release
    uses: silnith/github-workflows/.github/workflows/msbuild-arm.yaml@main
    with:
      artifact-prefix: cpp
      directory: .
      solution-file: foo.sln
      platform: ${{ matrix.platform }}
      configuration: ${{ matrix.configuration }}
      year: ${{ needs.get-date.outputs.year }}
      month: ${{ needs.get-date.outputs.month }}
      day: ${{ needs.get-date.outputs.day }}
    secrets: inherit
    permissions:
      contents: read
  cpp-test-results:
    name: Publish VSTest Results
    needs:
      - cpp-build-intel
      - cpp-build-arm
    uses: silnith/github-workflows/.github/workflows/publish-test-results.yaml@main
    with:
      check-name: C++ Test Results
      artifact-pattern: cpp-vstest-test-results-*
      artifact-path: .
      test-result-files: |
        cpp/**/TestResults/**/*.trx
    secrets: inherit
    permissions:
      checks: write
```

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
    name: .NET Test Results
    needs:
      - dotnet-build
    uses: silnith/github-workflows/.github/workflows/publish-test-results.yaml@main
    with:
      check-name: .NET Test Results
      artifact-pattern: csharp-dotnet-test-results-*
      artifact-path: .
      test-result-files: ./**/TestResults/**/*.trx
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
  maven-test-results:
    name: Maven Test Results
    needs:
      - maven-verify
    uses: silnith/github-workflows/.github/workflows/publish-test-results.yaml@main
    with:
      check-name: Java Test Results
      artifact-pattern: java-target
      artifact-path: .
      test-result-files: |
        ./**/target/surefire-reports/TEST-*.xml
        ./**/target/failsafe-reports/TEST-*.xml
    secrets: inherit
    permissions:
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
