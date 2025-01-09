# Actions.Shared
Reusable workflows.

## Example
So far, primarily used to test, build and push .net core projects to a Docker container registry.
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

  gitversion:
    needs: [analysis]
    name: Execute GitVersion and create tag
    uses: bvandevliet/Actions.Shared/.github/workflows/git-version.yml@master

  build-push:
    needs: [analysis,gitversion]
    name: Build and push
    uses: bvandevliet/Actions.Shared/.github/workflows/docker-build-push.yml@master
    strategy:
      matrix:
        project:
          - my.project.1
          - my.project.2
    with:
      CR_REGISTRY: docker.io
      PROJECT: ${{ matrix.project }}
      IMAGE_NAME: ${{ github.repository_owner }}/${{ matrix.project }}
      majorMinorPatch: ${{ needs.gitversion.outputs.majorMinorPatch }}
      preReleaseLabel: ${{ needs.gitversion.outputs.preReleaseLabel }}
      semVer: ${{ needs.gitversion.outputs.semVer }}
      assemblySemVer: ${{ needs.gitversion.outputs.assemblySemVer }}
      assemblySemFileVer: ${{ needs.gitversion.outputs.assemblySemFileVer }}
    secrets:
      CR_USERNAME: ${{ secrets.CR_USERNAME }}
      CR_PASSWORD: ${{ secrets.CR_PASSWORD }}
```