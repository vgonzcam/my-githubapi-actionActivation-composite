# GitHub Workflow State Management

A comprehensive GitHub Actions workflow system for managing the state of workflows in your repositories. Enable, disable, or list workflows through an intuitive interface with detailed logging and error handling.

## 📋 Table of Contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Workflow Structure](#workflow-structure)
- [Configuration](#configuration)
- [Examples](#examples)
- [API Reference](#api-reference)
- [Troubleshooting](#troubleshooting)
- [Security Considerations](#security-considerations)
- [Contributing](#contributing)
- [License](#license)

## ✨ Features

- **🔄 Enable Workflows**: Activate disabled workflows to resume automated processes
- **⏸️ Disable Workflows**: Temporarily stop workflows without deleting them
- **📋 List Workflows**: View all workflows with their current state and metadata
- **🎯 Composite Actions**: Reusable actions for modular workflow management
- **📊 Detailed Reporting**: Comprehensive logging and status summaries
- **🔒 Secure**: Token-based authentication with minimal required permissions
- **🚀 Easy to Use**: Simple workflow dispatch interface with dropdown selections

## 🔧 Prerequisites

Before you begin, ensure you have:

- A GitHub account with admin access to the target repository
- Permission to create GitHub Actions workflows
- Ability to create repository secrets

## 📦 Installation

### Step 1: Create Directory Structure

Create the following directory structure in your repository:

```
.github/
├── workflows/
│   └── manage-workflow-state.yml
└── actions/
    ├── enable-workflow/
    │   └── action.yml
    ├── disable-workflow/
    │   └── action.yml
    └── list-workflows/
        └── action.yml
```

### Step 2: Create a GitHub Token

1. Go to **GitHub Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**
2. Click **Generate new token** → **Generate new token (classic)**
3. Give your token a descriptive name (e.g., "Workflow Management Token")
4. Set an expiration date (recommended: 90 days or less)
5. Select the following scopes:
   - ✅ `repo` - Full control of private repositories
   - ✅ `workflow` - Update GitHub Action workflows
6. Click **Generate token** and copy the token immediately

### Step 3: Add Token to Repository Secrets

1. Navigate to your repository on GitHub
2. Go to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Name: `WORKFLOW_MANAGEMENT_TOKEN`
5. Value: Paste the token you copied in Step 2
6. Click **Add secret**

### Step 4: Copy Workflow Files

Copy all the workflow and action files from this repository into your repository following the directory structure above.

## 🚀 Usage

### Running the Workflow

1. Navigate to your repository on GitHub
2. Click on the **Actions** tab
3. Select **Manage Workflow State** from the workflow list
4. Click **Run workflow** button
5. Fill in the required parameters:

   **Action** (required)
   - `list-workflows` - List all workflows
   - `enable-workflow` - Enable a disabled workflow
   - `disable-workflow` - Disable an active workflow

   **Repository** (required)
   - Format: `owner/repo`
   - Example: `octocat/Hello-World`

   **Workflow ID** (required for enable/disable)
   - Workflow filename (e.g., `ci.yml`)
   - Or numeric workflow ID (e.g., `12345`)

6. Click **Run workflow**

### Viewing Results

After the workflow completes:

1. Click on the workflow run to view details
2. Expand the job steps to see detailed logs
3. Check the **Summary** tab for a formatted overview

## 📁 Workflow Structure

```
Manage Workflow State
│
├── list-workflows (job)
│   ├── Checkout repository
│   └── List workflows
│       └── Calls: .github/actions/list-workflows
│
├── enable-workflow (job)
│   ├── Checkout repository
│   ├── Validate workflow_id
│   └── Enable workflow
│       └── Calls: .github/actions/enable-workflow
│
├── disable-workflow (job)
│   ├── Checkout repository
│   ├── Validate workflow_id
│   └── Disable workflow
│       └── Calls: .github/actions/disable-workflow
│
└── summary (job)
    └── Generate Summary
```

## ⚙️ Configuration

### Workflow Inputs

| Input | Description | Required | Type | Default |
|-------|-------------|----------|------|---------|
| `action` | Operation to perform | Yes | Choice | `list-workflows` |
| `repository` | Target repository (`owner/repo`) | Yes | String | - |
| `workflow_id` | Workflow filename or ID | Conditional* | String | - |

*Required for `enable-workflow` and `disable-workflow` actions

### Environment Variables

The workflow uses the following environment variable:

- `GITHUB_TOKEN`: Set to `${{ secrets.WORKFLOW_MANAGEMENT_TOKEN }}`

### Permissions

The workflow jobs require the following permissions:

```yaml
permissions:
  contents: read
```

## 📚 Examples

### Example 1: List All Workflows

```
Action: list-workflows
Repository: myorg/my-repo
Workflow ID: (leave empty)
```

**Output:**
```
⚙️ WORKFLOWS LIST
================================================
🔧 Name: CI
   ID: 12345
   File: .github/workflows/ci.yml
   State: active
   URL: https://github.com/myorg/my-repo/actions/workflows/ci.yml

🔧 Name: Deploy
   ID: 12346
   File: .github/workflows/deploy.yml
   State: disabled_manually
   URL: https://github.com/myorg/my-repo/actions/workflows/deploy.yml

================================================
📊 Total workflows: 2
✅ Active: 1
⏸️  Disabled: 1
```

### Example 2: Enable a Workflow

```
Action: enable-workflow
Repository: myorg/my-repo
Workflow ID: deploy.yml
```

**Output:**
```
🎉 WORKFLOW ENABLED
================================================
The workflow 'deploy.yml' is now active and will run on triggers.
```

### Example 3: Disable a Workflow

```
Action: disable-workflow
Repository: myorg/my-repo
Workflow ID: ci.yml
```

**Output:**
```
⏸️ WORKFLOW DISABLED
================================================
The workflow 'ci.yml' has been disabled and will not run on triggers.
```

### Example 4: Using Workflow ID Instead of Filename

```
Action: enable-workflow
Repository: myorg/my-repo
Workflow ID: 12346
```

## 🔌 API Reference

This workflow uses the GitHub REST API for Actions:

### List Workflows
```
GET /repos/{owner}/{repo}/actions/workflows
```

### Enable Workflow
```
PUT /repos/{owner}/{repo}/actions/workflows/{workflow_id}/enable
```

### Disable Workflow
```
PUT /repos/{owner}/{repo}/actions/workflows/{workflow_id}/disable
```

### API Version
All requests use API version: `2022-11-28`

### Response Codes

| Code | Meaning | Description |
|------|---------|-------------|
| 200 | OK | Successfully retrieved data |
| 204 | No Content | Successfully modified workflow state |
| 403 | Forbidden | Insufficient permissions or rate limited |
| 404 | Not Found | Repository or workflow doesn't exist |
| 422 | Unprocessable Entity | Invalid parameters |

## 🐛 Troubleshooting

### Error: 404 Not Found

**Symptoms:**
```
❌ Repository not found (HTTP 404)
```

**Possible Causes:**
- Repository name is incorrect or misspelled
- Repository doesn't exist
- Token doesn't have access to the repository
- Repository is private and token lacks permissions

**Solutions:**
1. Verify repository name format: `owner/repo`
2. Ensure repository exists and is accessible
3. Check token permissions include `repo` scope
4. Verify you have access to the repository

### Error: 403 Forbidden

**Symptoms:**
```
❌ Forbidden (HTTP 403)
```

**Possible Causes:**
- Token doesn't have required permissions
- API rate limit exceeded
- Token is invalid or expired
- Organization has restricted token access

**Solutions:**
1. Regenerate token with `repo` and `workflow` scopes
2. Wait for rate limit to reset (check headers)
3. Verify token hasn't expired
4. Check organization security policies

### Error: Workflow Not Found (404 on specific workflow)

**Symptoms:**
```
❌ Workflow not found (HTTP 404)
```

**Possible Causes:**
- Workflow ID or filename is incorrect
- Workflow was deleted
- Wrong repository specified

**Solutions:**
1. Run the `list-workflows` action first to see all available workflows
2. Use the exact filename (e.g., `ci.yml`, not `ci`)
3. Verify you're targeting the correct repository

### Error: 422 Unprocessable Entity

**Symptoms:**
```
❌ Failed to enable/disable workflow (HTTP 422)
```

**Possible Causes:**
- Workflow is already in the requested state
- Invalid workflow ID format

**Solutions:**
1. Check current workflow state with `list-workflows`
2. Ensure workflow ID is either a filename or numeric ID
3. Verify workflow exists in the repository

### Workflow Validation Errors

**Symptoms:**
```
❌ Error: workflow_id is required for enable-workflow action
```

**Solutions:**
- Ensure you provide the Workflow ID when using enable/disable actions
- Check that you haven't left the field empty

## 🔒 Security Considerations

### Token Security

- **Never commit tokens to your repository**
- Use repository secrets for all sensitive data
- Rotate tokens regularly (every 90 days recommended)
- Use tokens with minimal required permissions
- Revoke tokens immediately if compromised

### Permissions

The workflow requires:
- `repo` scope for repository access
- `workflow` scope for modifying workflow states

### Best Practices

1. **Principle of Least Privilege**: Only grant necessary permissions
2. **Audit Logs**: Monitor workflow runs in Actions tab
3. **Token Rotation**: Set expiration dates on tokens
4. **Access Control**: Limit who can trigger the workflow
5. **Review Changes**: Always review workflow modifications

### GitHub Security Features

- Enable branch protection rules
- Require pull request reviews
- Use CODEOWNERS file
- Enable security alerts
- Use Dependabot for dependency updates

## 🤝 Contributing

Contributions are welcome! Here's how you can help:

### Reporting Issues

1. Check existing issues first
2. Provide detailed reproduction steps
3. Include workflow logs and error messages
4. Specify GitHub Enterprise Server version if applicable

### Submitting Pull Requests

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Development Guidelines

- Follow existing code style
- Add comments for complex logic
- Update README if adding features
- Test thoroughly before submitting
- Keep commits atomic and well-described

## 📄 License

This project is licensed under the MIT License.

```
MIT License

Copyright (c) 2025

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## 📞 Support

- **Documentation**: [GitHub Actions Documentation](https://docs.github.com/en/actions)
- **API Reference**: [GitHub REST API](https://docs.github.com/en/rest)
- **Community**: [GitHub Community Forum](https://github.community/)

## 🎯 Roadmap

Future enhancements planned:

- [ ] Bulk enable/disable multiple workflows
- [ ] Schedule workflow state changes
- [ ] Workflow state backup and restore
- [ ] Integration with GitHub Apps
- [ ] Support for organization-level management
- [ ] Webhook notifications for state changes
- [ ] Workflow analytics and reporting

## 📊 Changelog

### Version 1.0.0 (2025-10-02)
- Initial release
- Enable/disable/list workflows functionality
- Composite actions architecture
- Detailed logging and error handling
- Comprehensive documentation
