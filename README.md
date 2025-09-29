# CalVer Tag Generator Action

🏷️ **GitHub Action to automatically generate CalVer tags with increments for same-day releases.**

[![GitHub release (latest by date)](https://img.shields.io/github/v/release/Nvision-x/gh-actions-calver)](https://github.com/Nvision-x/gh-actions-calver/releases)
[![GitHub](https://img.shields.io/github/license/Nvision-x/gh-actions-calver)](https://github.com/Nvision-x/gh-actions-calver/blob/main/LICENSE)

This action automates the generation of CalVer (Calendar Versioning) tags with increments for multiple releases on the same day.

## 🚀 Quick usage

**Basic usage (all inputs are optional):**

```yaml
- name: Generate CalVer tag
  id: calver
  uses: Nvision-x/gh-actions-calver@v2024.09.22-1  # Recommended: use specific version
  # uses: Nvision-x/gh-actions-calver@v2025.09.26-1         # Alternative: latest version
```

**With prefix and custom options:**

```yaml
- name: Generate CalVer tag
  id: calver
  uses: Nvision-x/gh-actions-calver@v2024.09.22-1
  with:
    prefix: 'dev'              # Optional: adds prefix to tag
    use-sequence: true         # Optional: enable/disable sequence numbering
```

> **💡 Version recommendation:** Use a specific version (e.g., `@v2024.09.22-1`) for production workflows to ensure stability. Use `@v2025.09.26-1` only for testing or if you want the latest features.

## 📥 Inputs

All inputs are optional with smart defaults.

| Input            | Description                                             | Default       |
| ---------------- | ------------------------------------------------------- | ------------- |
| `prefix`       | Optional prefix for the tag (e.g., "dev", "prod")       | `''` (none) |
| `use-sequence` | Whether to use sequence numbering for same-day releases | `true`      |

> **Note:** The action automatically uses `${{ github.token }}` and `${{ github.repository }}` from the workflow context.

## 📤 Outputs

| Output | Description |
|--------|-------------|
| `tag` | Generated CalVer tag (format: YYYY.MM.DD-N) |
| `create_release` | `true` if a new tag was created, `false` if tag already existed |
| `increment` | Increment number for the day |

> **Note:** This action only creates Git tags. To create GitHub releases, use the `create_release` output with `softprops/action-gh-release` or similar.

## 🏷️ Tag Format

### With sequence (default):

- **First release of the day**: `2024.09.22-1`
- **Second release of the day**: `2024.09.22-2`
- **With prefix**: `dev2024.09.22-1`, `prod2024.09.22-1`

### Without sequence:

- **Single release per day**: `2024.09.22`
- **With prefix**: `dev2024.09.22`, `prod2024.09.22`

## ✨ Features

- ✅ **Auto-detection**: Automatically finds the next available number
- ✅ **Dual verification**: Checks both local and remote tags
- ✅ **Conflict handling**: Automatically resolves tag conflicts
- ✅ **Rollback safe**: Continues execution even if tag creation fails
- ✅ **Detailed logging**: Provides clear information about the process
- ✅ **Customizable prefixes**: Add prefixes like "dev", "prod", etc.
- ✅ **Optional sequence**: Control whether to use sequential numbering or not

## 📋 Complete example

```yaml
name: Release with CalVer

on:
  push:
    branches: [ main ]

jobs:
  generate-tag:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    outputs:
      tag: ${{ steps.calver.outputs.tag }}
      create_release: ${{ steps.calver.outputs.create_release }}
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      with:
        fetch-depth: 0

    - name: Generate CalVer tag
      id: calver
      uses: Nvision-x/gh-actions-calver@v2024.09.22-1
      with:
        prefix: 'prod'
        use-sequence: true

    - name: Create Release
      if: steps.calver.outputs.create_release == 'true'
      uses: softprops/action-gh-release@v1
      with:
        tag_name: ${{ steps.calver.outputs.tag }}
        name: Release ${{ steps.calver.outputs.tag }}
        generate_release_notes: true

    - name: Use generated tag
      run: echo "Generated tag: ${{ steps.calver.outputs.tag }}"
```

> **Pro tip:** The action automatically handles GitHub token and repository detection, so you only need to specify the options you want to customize.

## 📚 More examples

For more usage examples, see [EXAMPLES.md](EXAMPLES.md).
