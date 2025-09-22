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
      # No inputs needed - uses smart defaults

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

## Using outputs in subsequent steps

### Docker image tagging example

```yaml
name: Build and Tag Docker Image

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Generate CalVer tag
      id: calver
      uses: Nvision-x/gh-actions-calver@main
      with:
        prefix: 'v'

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Login to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}

    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: |
          myorg/myapp:${{ steps.calver.outputs.tag }}
          myorg/myapp:latest
        labels: |
          org.opencontainers.image.version=${{ steps.calver.outputs.tag }}

    - name: Update Kubernetes manifests
      run: |
        echo "Updating deployment with tag: ${{ steps.calver.outputs.tag }}"
        sed -i "s|image: myorg/myapp:.*|image: myorg/myapp:${{ steps.calver.outputs.tag }}|" k8s/deployment.yaml
```

### Version file update example

```yaml
- name: Generate version tag
  id: version
  uses: Nvision-x/gh-actions-calver@main

- name: Update version files
  run: |
    echo "${{ steps.version.outputs.tag }}" > VERSION
    echo "export VERSION=${{ steps.version.outputs.tag }}" > version.sh
    jq '.version = "${{ steps.version.outputs.tag }}"' package.json > package.json.tmp
    mv package.json.tmp package.json

- name: Create artifact with version
  run: |
    echo "Building artifact version: ${{ steps.version.outputs.tag }}"
    tar -czf myapp-${{ steps.version.outputs.tag }}.tar.gz dist/
```

### Multi-stage deployment example

```yaml
jobs:
  tag:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.calver.outputs.tag }}
      should_deploy: ${{ steps.calver.outputs.create_release }}
    steps:
    - uses: actions/checkout@v4
    - name: Generate version
      id: calver
      uses: Nvision-x/gh-actions-calver@main

  build:
    needs: tag
    runs-on: ubuntu-latest
    steps:
    - name: Build with version
      run: |
        echo "Building version: ${{ needs.tag.outputs.version }}"
        docker build -t myapp:${{ needs.tag.outputs.version }} .

  deploy:
    needs: [tag, build]
    if: needs.tag.outputs.should_deploy == 'true'
    runs-on: ubuntu-latest
    steps:
    - name: Deploy version
      run: |
        echo "Deploying version: ${{ needs.tag.outputs.version }}"
        kubectl set image deployment/myapp myapp=myapp:${{ needs.tag.outputs.version }}
```

## Required permissions

Make sure your workflow has the necessary permissions:

```yaml
permissions:
  contents: write  # To create tags and releases
  actions: read    # To read workflow metadata
```
