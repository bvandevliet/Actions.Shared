# Actions.Shared
Reusable workflows.

## Example
So far, primarily used to test, build and push (dotnet core) projects to a Docker container registry.
```yaml
name: Docker publish

on:
  schedule:
    - cron: '45 0 * * *'
  push:
    branches:
      - '**'
  pull_request:
    branches:
      - '**'

permissions:
  contents: write
  packages: read
  actions: read
  security-events: write

jobs:
  analysis:
    name: Run code analysis and tests
    uses: bvandevliet/Actions.Shared/.github/workflows/dotnet-analysis.yml@master
    with:
      DOTNET_VERSION: 10
      skipTrivyScan: true
  gitversion:
    needs: [analysis]
    name: Execute GitVersion and create tag
    uses: bvandevliet/Actions.Shared/.github/workflows/git-version.yml@master
  build-publish:
    needs: [analysis,gitversion]
    name: Build and publish
    uses: bvandevliet/Actions.Shared/.github/workflows/docker-build-publish.yml@master
    strategy:
      matrix:
        project:
          - my.project.1
          - my.project.2
    with:
      CSPROJ_NAME: ${{ matrix.project }}
      DOTNET_VERSION: 10
      IMAGE_NAME: ${{ github.repository_owner }}/${{ matrix.project }}
      majorMinorPatch: ${{ needs.gitversion.outputs.majorMinorPatch }}
      preReleaseLabel: ${{ needs.gitversion.outputs.preReleaseLabel }}
      semVer: ${{ needs.gitversion.outputs.semVer }}
      assemblySemVer: ${{ needs.gitversion.outputs.assemblySemVer }}
      assemblySemFileVer: ${{ needs.gitversion.outputs.assemblySemFileVer }}
      CR_REGISTRY: docker.io
    secrets:
      CR_USERNAME: ${{ secrets.CR_USERNAME }}
      CR_PASSWORD: ${{ secrets.CR_PASSWORD }}
```