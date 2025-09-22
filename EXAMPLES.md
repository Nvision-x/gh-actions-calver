# Usage Examples

This document contains practical examples of how to use the CalVer Tag Generator in different scenarios.

## Basic example (no additional configurations)

```yaml
name: Auto Release

on:
  push:
    branches: [ main ]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
        token: ${{ secrets.GITHUB_TOKEN }}

    - name: Generate CalVer tag
      id: calver
      uses: Nvision-x/gh-actions-calver@main
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}
        repository: ${{ github.repository }}

    - name: Show generated tag
      run: echo "Generated tag: ${{ steps.calver.outputs.tag }}"
```
**Result**: Tags like `2024.09.22-1`, `2024.09.22-2`, etc.

## Example with development prefix

```yaml
name: Development Release

on:
  push:
    branches: [ develop ]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0

    - name: Generate CalVer tag with dev prefix
      id: calver
      uses: Nvision-x/gh-actions-calver@main
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}
        prefix: 'dev'
        use-sequence: 'true'

    - name: Create Development Release
      uses: softprops/action-gh-release@v1
      with:
        tag_name: ${{ steps.calver.outputs.tag }}
        name: Development Release ${{ steps.calver.outputs.tag }}
        prerelease: true
```
**Result**: Tags like `dev2024.09.22-1`, `dev2024.09.22-2`, etc.

## Example without sequence (one tag per day)

```yaml
name: Daily Release

on:
  schedule:
    - cron: '0 9 * * MON-FRI'  # Every weekday at 9 AM
  workflow_dispatch:

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0

    - name: Generate single daily tag
      id: calver
      uses: Nvision-x/gh-actions-calver@main
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}
        use-sequence: 'false'

    - name: Create Daily Release
      if: steps.calver.outputs.create_release == 'true'
      uses: softprops/action-gh-release@v1
      with:
        tag_name: ${{ steps.calver.outputs.tag }}
        name: Daily Build ${{ steps.calver.outputs.tag }}
```
**Result**: Tags like `2024.09.22` (one per day)

## Multi-environment example

```yaml
name: Multi-Environment Release

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        type: choice
        options:
        - dev
        - staging
        - prod

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0

    - name: Generate environment-specific tag
      id: calver
      uses: Nvision-x/gh-actions-calver@main
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}
        prefix: ${{ inputs.environment }}
        use-sequence: 'true'

    - name: Deploy to environment
      run: |
        echo "Deploying ${{ steps.calver.outputs.tag }} to ${{ inputs.environment }}"
        # Here would go the environment-specific deployment commands
```
**Result**: Tags like `dev2024.09.22-1`, `staging2024.09.22-1`, `prod2024.09.22-1`

## Example with release creation

```yaml
name: Release with CalVer

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0

    - name: Generate CalVer tag
      id: calver
      uses: Nvision-x/gh-actions-calver@main
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}

    - name: Create Release
      if: steps.calver.outputs.create_release == 'true'
      uses: softprops/action-gh-release@v1
      with:
        tag_name: ${{ steps.calver.outputs.tag }}
        name: Release ${{ steps.calver.outputs.tag }}
        body: |
          🎉 Automatic release ${{ steps.calver.outputs.tag }}
          
          Release #${{ steps.calver.outputs.increment }} for today.
        draft: false
        prerelease: false
```

## Example with build and deploy

```yaml
name: Build, Tag & Deploy

on:
  push:
    branches: [ main ]

jobs:
  build-and-release:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test

    - name: Build project
      run: npm run build

    - name: Generate CalVer tag
      id: calver
      uses: Nvision-x/gh-actions-calver@main
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}

    - name: Create Release with build artifacts
      if: steps.calver.outputs.create_release == 'true'
      uses: softprops/action-gh-release@v1
      with:
        tag_name: ${{ steps.calver.outputs.tag }}
        name: Release ${{ steps.calver.outputs.tag }}
        files: |
          dist/*
        body: |
          🚀 Production release ${{ steps.calver.outputs.tag }}
          
          **Build info:**
          - Build number: #${{ github.run_number }}
          - Commit: ${{ github.sha }}
          - Daily increment: ${{ steps.calver.outputs.increment }}
```

## Conditional example based on files with prefix

```yaml
name: Smart Release with Prefix

on:
  push:
    branches: [ main ]
    paths:
      - 'src/**'
      - 'package.json'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0

    - name: Determine environment prefix
      id: env
      run: |
        if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
          echo "prefix=prod" >> $GITHUB_OUTPUT
        else
          echo "prefix=dev" >> $GITHUB_OUTPUT
        fi

    - name: Generate CalVer tag
      id: calver
      uses: Nvision-x/gh-actions-calver@main
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}
        prefix: ${{ steps.env.outputs.prefix }}
        use-sequence: 'true'

    - name: Create Release
      if: steps.calver.outputs.create_release == 'true'
      uses: softprops/action-gh-release@v1
      with:
        tag_name: ${{ steps.calver.outputs.tag }}
        name: Release ${{ steps.calver.outputs.tag }}
        generate_release_notes: true
```

## Hybrid versioning example

```yaml
name: Hybrid Versioning

on:
  push:
    branches: [ main ]
  workflow_dispatch:
    inputs:
      use_sequence:
        description: 'Use sequence numbering'
        type: boolean
        default: true
      prefix:
        description: 'Tag prefix'
        type: string
        default: ''

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0

    - name: Generate flexible CalVer tag
      id: calver
      uses: Nvision-x/gh-actions-calver@main
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}
        prefix: ${{ inputs.prefix || 'auto' }}
        use-sequence: ${{ inputs.use_sequence || 'true' }}

    - name: Handle different tag types
      run: |
        if [[ "${{ steps.calver.outputs.increment }}" == "0" ]]; then
          echo "Single tag for the day: ${{ steps.calver.outputs.tag }}"
        else
          echo "Sequenced tag: ${{ steps.calver.outputs.tag }} (increment: ${{ steps.calver.outputs.increment }})"
        fi
```

## Generated tag formats

### With sequence enabled (use-sequence: true):
- **Without prefix**: `2024.09.22-1`, `2024.09.22-2`, `2024.09.22-3`
- **With "dev" prefix**: `dev2024.09.22-1`, `dev2024.09.22-2`
- **With "prod" prefix**: `prod2024.09.22-1`, `prod2024.09.22-2`

### Without sequence (use-sequence: false):
- **Without prefix**: `2024.09.22` (only one per day)
- **With "dev" prefix**: `dev2024.09.22` (only one per day)
- **With "release" prefix**: `release2024.09.22` (only one per day)

### Special cases:
- If `use-sequence: false` and the tag already exists, no new tag is created
- If `use-sequence: true`, it always finds the next available number

## Available outputs

| Output | Description | Example |
|--------|-------------|---------|
| `tag` | Generated CalVer tag | `dev2024.09.22-1` or `2024.09.22` |
| `create_release` | Whether a release should be created | `true` or `false` |
| `increment` | Day increment number (0 if no sequence) | `1`, `2`, `0` |

## Required permissions

Make sure your workflow has the necessary permissions:

```yaml
permissions:
  contents: write  # To create tags and releases
  actions: read    # To read workflow metadata
```
