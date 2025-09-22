# CalVer Tag Generator Action

🏷️ **GitHub Action to automatically generate CalVer tags with increments for same-day releases.**

[![GitHub release (latest by date)](https://img.shields.io/github/v/release/Nvision-x/gh-actions-calver)](https://github.com/Nvision-x/gh-actions-calver/releases)
[![GitHub](https://img.shields.io/github/license/Nvision-x/gh-actions-calver)](https://github.com/Nvision-x/gh-actions-calver/blob/main/LICENSE)

This action automates the generation of CalVer (Calendar Versioning) tags with increments for multiple releases on the same day.

## 🚀 Quick usage

```yaml
- name: Generate CalVer tag
  id: calver
  uses: Nvision-x/gh-actions-calver@main
  # All inputs are optional - uses smart defaults
```

**With custom options:**
```yaml
- name: Generate CalVer tag
  id: calver
  uses: Nvision-x/gh-actions-calver@main
  with:
    # github-token: ${{ secrets.GITHUB_TOKEN }}  # Optional - uses default
    # repository: ${{ github.repository }}       # Optional - uses default
    prefix: 'dev'              # Optional: adds prefix to tag
    use-sequence: 'true'       # Optional: enable/disable sequence numbering
```

## 📥 Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `github-token` | GitHub token for API access | ❌ | `${{ github.token }}` |
| `repository` | Repository name (owner/repo) | ❌ | `${{ github.repository }}` |
| `prefix` | Optional prefix for the tag (e.g., "dev", "prod") | ❌ | `''` |
| `use-sequence` | Whether to use sequence numbering for same-day releases | ❌ | `'true'` |

## 📤 Outputs

| Output | Description |
|--------|-------------|
| `tag` | Generated CalVer tag (format: YYYY.MM.DD-N) |
| `create_release` | Boolean indicating if a release should be created |
| `increment` | Increment number for the day |

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
      actions: read
    outputs:
      tag: ${{ steps.calver.outputs.tag }}
      create_release: ${{ steps.calver.outputs.create_release }}
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      with:
        fetch-depth: 0
        token: ${{ secrets.GITHUB_TOKEN }}

    - name: Generate CalVer tag
      id: calver
      uses: Nvision-x/gh-actions-calver@main
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}
        repository: ${{ github.repository }}
        prefix: 'prod'
        use-sequence: 'true'

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

## 📚 More examples

For more usage examples, see [EXAMPLES.md](EXAMPLES.md).

## 🔧 Development

If you want to contribute to this project:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-feature`)
3. Commit your changes (`git commit -am 'Add new feature'`)
4. Push to the branch (`git push origin feature/new-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for more details.

## 🤝 Contributing

Contributions are welcome. For major changes, please open an issue first to discuss what you would like to change.

## 🐛 Report bugs

If you find a bug, please open an [issue](https://github.com/Nvision-x/gh-actions-calver/issues) with:
- Description of the problem
- Steps to reproduce
- Expected vs actual behavior
- Relevant logs
