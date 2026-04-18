# GitHub Reusable Workflows

Standard GitHub workflows that use best practices for building and testing software.

## Why So Complicated?

There are a lot of pieces for a robust build.  Setting up toolchains, fetching
dependency packages, running the build, running the test suites, archiving the
built artifacts, publishing the test results, and more.

### Caching

Perhaps the most frustrating part is needing to implement a fairly complex
sequence of steps just to cache commonly-used data such as dependency packages.
Most dependency management systems like Maven and NuGet have their own caching
mechanism built-in.  Unfortunately, these rely on having a persistent filesystem
to hold the cache, and OCI container-based builds do not provide that, their
filesystems are ephemeral.  GitHub provides a Cache action, but it functions as
a single cache, and dependencies are a web of individual entries that need to
be cached separately.

The standard GitHub advice is to just archive the entire collection under a
generated key based on a fingerprint of the files declaring the dependencies.
This advice is... naive at best.  Complex Java software can have scores, even
hundreds of libraries in its dependency tree.  Normal development often includes
updating one or two of these libraries at a time.  Using the GitHub advice would
mean each of these individual updates would change the fingerprint, causing the
cache to miss and forcing the build to re-download all of the dependencies, not
just the couple that were modified.

Instead, these workflows provide a different cache naming system that offers a
more graceful fallback path.  Caches are named based primarily on the date of the
build in ISO 8601 format.  When checking the cache, it falls back to matching
prefixes so that previous dependency trees can be used as a starting point for
rebuilding the current dependency tree.

Of course, none of this would be necessary if GitHub offered colocated mirrors
for Maven Central and NuGet.org.

## C++

```yaml
jobs:
  get-date:
    name: Date
    uses: silnith/github-workflows/.github/workflows/bash-get-date.yaml@main
  nuget-cache:
    name: NuGet Cache
    permissions:
      contents: read
      packages: read
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
  msbuild-intel:
    name: C++ Intel
    permissions:
      contents: read
      packages: read
    needs:
      - get-date
      - nuget-cache
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
      artifact-prefix: msbuild
      directory: .
      solution-file: foo.sln
      platform: ${{ matrix.platform }}
      configuration: ${{ matrix.configuration }}
      year: ${{ needs.get-date.outputs.year }}
      month: ${{ needs.get-date.outputs.month }}
      day: ${{ needs.get-date.outputs.day }}
    secrets: inherit
  msbuild-arm:
    name: C++ ARM
    permissions:
      contents: read
      packages: read
    needs:
      - get-date
      - nuget-cache
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
      artifact-prefix: msbuild
      directory: .
      solution-file: foo.sln
      platform: ${{ matrix.platform }}
      configuration: ${{ matrix.configuration }}
      year: ${{ needs.get-date.outputs.year }}
      month: ${{ needs.get-date.outputs.month }}
      day: ${{ needs.get-date.outputs.day }}
    secrets: inherit
  vstest-results:
    name: VSTest Results
    permissions:
      contents: read
      issues: read
      checks: write
    needs:
      - msbuild-intel
      - msbuild-arm
    uses: silnith/github-workflows/.github/workflows/publish-test-results.yaml@main
    with:
      check-name: C++ Test Results
      artifact-pattern: msbuild-vstest-*
      artifact-path: .
      test-result-files: "**/TestResults/**/*.trx"
    secrets: inherit
```

## .NET

```yaml
jobs:
  get-date:
    name: Date
    uses: silnith/github-workflows/.github/workflows/bash-get-date.yaml@main
  dotnet-cache:
    name: .NET Cache
    permissions:
      contents: read
      packages: read
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
  dotnet-build:
    name: .NET
    permissions:
      contents: read
      packages: read
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
      artifact-prefix: dotnet
      directory: .
      solution-file: foo.sln
      configuration: ${{ matrix.configuration }}
      dotnet-versions: 5.x
      year: ${{ needs.get-date.outputs.year }}
      month: ${{ needs.get-date.outputs.month }}
      day: ${{ needs.get-date.outputs.day }}
    secrets: inherit
  dotnet-test-results:
    name: .NET Test Results
    permissions:
      contents: read
      issues: read
      checks: write
    needs:
      - dotnet-build
    uses: silnith/github-workflows/.github/workflows/publish-test-results.yaml@main
    with:
      check-name: .NET Test Results
      artifact-pattern: dotnet-test-*
      artifact-path: .
      test-result-files: "**/TestResults/**/*.trx"
    secrets: inherit
```

## Java

```yaml
jobs:
  get-date:
    name: Date
    uses: silnith/github-workflows/.github/workflows/bash-get-date.yaml@main
  maven-cache:
    name: Maven Cache
    permissions:
      contents: read
      packages: read
    needs:
      - get-date
    uses: silnith/github-workflows/.github/workflows/maven-cache-dependencies.yaml@main
    with:
      directory: .
      year: ${{ needs.get-date.outputs.year }}
      month: ${{ needs.get-date.outputs.month }}
      day: ${{ needs.get-date.outputs.day }}
    secrets: inherit
  maven-verify:
    name: Maven
    permissions:
      contents: read
      packages: read
    needs:
      - get-date
      - maven-cache
    uses: silnith/github-workflows/.github/workflows/maven-verify.yaml@main
    with:
      artifact-prefix: maven
      directory: .
      year: ${{ needs.get-date.outputs.year }}
      month: ${{ needs.get-date.outputs.month }}
      day: ${{ needs.get-date.outputs.day }}
    secrets: inherit
  maven-deploy:
    name: Maven Deploy
    permissions:
      contents: read
      packages: write
    needs:
      - get-date
      - maven-cache
      - maven-verify
    uses: silnith/github-workflows/.github/workflows/maven-deploy.yaml@main
    with:
      artifact-prefix: maven
      directory: .
      year: ${{ needs.get-date.outputs.year }}
      month: ${{ needs.get-date.outputs.month }}
      day: ${{ needs.get-date.outputs.day }}
    secrets: inherit
  maven-test-results:
    name: Maven Test Results
    permissions:
      contents: read
      issues: read
      checks: write
    needs:
      - maven-verify
    uses: silnith/github-workflows/.github/workflows/publish-test-results.yaml@main
    with:
      check-name: Java Test Results
      artifact-pattern: maven-target
      artifact-path: .
      test-result-files: |
        ./**/target/surefire-reports/TEST-*.xml
        ./**/target/failsafe-reports/TEST-*.xml
    secrets: inherit
```
