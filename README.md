# Fabric Agentic Workflow Template

This repository is a **template** for using [GitHub Agentic Workflows](https://github.com/github/gh-aw) with Microsoft Fabric Power BI semantic models. It serves as a foundation for building your own specialized instructions to an agent, including building tests, checking for naming conventions, and more, all within the bounds of a GitHub Action. Out of the box, it demonstrates how an AI agent can automatically generate documentation for your semantic models using the [Power BI Modeling MCP Server](https://www.npmjs.com/package/@microsoft/powerbi-modeling-mcp), available as a pre-built container on [Docker Hub](https://hub.docker.com/repository/docker/kerski/powerbi-modeling-mcp/general).

## Getting Started -- Clone to a Private Repository

Because this template involves GitHub Actions secrets (tokens, credentials), we recommend hosting your copy in a **private repository**. GitHub does not allow forking a public repo as private, so use the steps below to mirror it instead.

### Option A: Mirror via the command line

```bash
# 1. Create a new private repo on GitHub (do NOT initialize it with a README)
gh repo create <your-username>/github-agentic-workflow-power-bi-mcp-example --private --confirm

# 2. Clone this template repo locally
git clone --bare https://github.com/clientfirsttech/github-agentic-workflow-power-bi-mcp-example.git
cd github-agentic-workflow-power-bi-mcp-example.git

# 3. Push the mirror to your new private repo
git push --mirror https://github.com/<your-username>/github-agentic-workflow-power-bi-mcp-example.git

# 4. Clean up the bare clone
cd ..
rm -rf github-agentic-workflow-power-bi-mcp-example.git

# 5. Clone your private repo and start working
git clone https://github.com/<your-username>/github-agentic-workflow-power-bi-mcp-example.git
cd github-agentic-workflow-power-bi-mcp-example
```

### Option B: Import via the GitHub UI

1. Go to [github.com/new/import](https://github.com/new/import).
2. Paste the source URL: `https://github.com/clientfirsttech/github-agentic-workflow-power-bi-mcp-example`.
3. Set the **Owner** to your account or organization.
4. Choose a repository name (e.g., `github-agentic-workflow-power-bi-mcp-example`).
5. Select **Private**.
6. Click **Begin import** and wait for it to finish.

Once your private repo is ready, continue with the [Setup Guide](#setup-guide) below.

## What's Included

| Component | Description |
|---|---|
| **Agentic Workflow** (`.github/workflows/generate-pbi-docs.md`) | A GitHub Agentic Workflow that uses Copilot to generate Markdown documentation for every `*.SemanticModel` in the repo. |
| **Sample Semantic Models** | Example Power BI project files (`SampleModel`, `Relationships`) so you can test the workflow immediately. |
| **Documentation Output** (`documentation/`) | Directory where the agent writes generated documentation and opens a pull request. |

## Prerequisites

- A GitHub repository with **admin access** (to configure secrets, permissions, and Actions)
- Access to [GitHub Copilot](https://github.com/features/copilot) (required by the agentic workflow engine)

**Note:** This is optional. If you want to modify this agentic flow, you will need the GitHub CLI installed locally with the agentic extensions.

1. Install the [GitHub CLI](https://cli.github.com/)
2. Install the [`gh-aw` extension](https://github.com/github/gh-aw):
```bash
curl -sL https://raw.githubusercontent.com/github/gh-aw/main/install-gh-aw.sh | bash
```

## Setup Guide

### Step 1: Set Up the `COPILOT_GITHUB_TOKEN`

The agentic workflow uses Copilot as its engine, which requires a `COPILOT_GITHUB_TOKEN` secret. This is a **fine-grained personal access token** that grants the workflow access to Copilot.

1. Go to **GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens**.
2. Click **Generate new token**.
3. Give it a descriptive name (e.g., `Copilot Agentic Workflow`).
4. Set an **Expiration** date appropriate for your use case.
5. Under **Resource owner**, select your account or organization.
6. Under **Repository access**, select the repository (or repositories) where the agentic workflow will run.
7. Expand **Account permissions** and set the following:
   - **Copilot** → **Read-only** -- required for the Copilot engine to function
8. Click **Generate token** and copy the value.
9. In your repository, go to **Settings → Secrets and variables → Actions → Secrets**.
10. Click **New repository secret**.
11. Set the name to `COPILOT_GITHUB_TOKEN` and paste the token value.

> **Note:** The compiled workflow (`.lock.yml`) also references `GH_AW_GITHUB_TOKEN`, `GH_AW_CI_TRIGGER_TOKEN`, and `GH_AW_GITHUB_MCP_SERVER_TOKEN`. These are automatically managed by the `gh aw` infrastructure. You only need to manually configure `COPILOT_GITHUB_TOKEN`.

### Step 2: Grant GitHub Actions Permission to Create Pull Requests

The agentic workflow uses the `create-pull-request` safe output to open PRs with the generated documentation. For this to work, GitHub Actions must have permission to create pull requests.

1. Go to your repository **Settings → Actions → General**.
2. Scroll down to the **Workflow permissions** section.
3. Select **Read and write permissions**.
4. Check the box: **Allow GitHub Actions to create and approve pull requests**.
5. Click **Save**.

Without this permission, the agent will be able to generate documentation but will fail when attempting to open a pull request.

> **Note:** This template uses the pre-built [Power BI Modeling MCP Server container from Docker Hub](https://hub.docker.com/repository/docker/kerski/powerbi-modeling-mcp/general) (`kerski/powerbi-modeling-mcp:latest`). No container build or publishing step is required -- the agentic workflow pulls the image directly from Docker Hub at runtime.

### Step 3: Test the Agentic Workflow

You can trigger the agentic workflow in two ways:

- **Automatically:** Push a change to any `*.SemanticModel/**` path on the `main` branch.
- **Manually:** Go to **Actions → Generate Semantic Model Documentation → Run workflow**.

When the workflow runs, the Copilot agent will:
1. Discover all `*.SemanticModel` directories in the repository.
2. Connect to each model using the Power BI Modeling MCP Server container.
3. Generate comprehensive Markdown documentation (relationships, measures, RLS, data sources).
4. Open a pull request with the generated documentation files in the `documentation/` directory.

## Repository Structure

```
.
├── .github/
│   ├── aw/
│   │   └── actions-lock.json          # Pinned action SHAs for agentic workflows
│   └── workflows/
│       ├── generate-pbi-docs.md        # Agentic workflow definition (source of truth)
│       └── generate-pbi-docs.lock.yml  # Compiled workflow (auto-generated, do not edit)
├── SampleModel.SemanticModel/          # Example semantic model (TMDL format)
├── SampleModel.Report/                 # Example report
├── SampleModel.pbip                    # Power BI project file
├── Relationships.SemanticModel/        # Example semantic model with relationships
├── Relationships.Report/               # Example report
├── Relationships.pbip                  # Power BI project file
├── ThinReportSampleModel.Report/       # Example thin report
├── ThinReportSampleModel.pbip          # Power BI project file
├── documentation/                      # Agent-generated documentation output
└── README.md
```

## How Agentic Workflows Work

GitHub Agentic Workflows use Markdown files (`.md`) with YAML frontmatter as the source definition. The key components:

- **`on:`** -- Standard GitHub Actions trigger events.
- **`permissions:`** -- Read-only permissions for the agent job itself.
- **`safe-outputs:`** -- Write operations (like `create-pull-request`) that are handled securely outside the agent sandbox. This is how the agent can create PRs without needing direct write access.
- **`engine:`** -- Specifies `copilot` as the AI engine.
- **`tools:`** -- Allowed tools the agent can use (e.g., `edit`, `bash` with allow-listed commands).
- **`mcp-servers:`** -- MCP servers available to the agent, specified as container images.
- **Prompt body** -- The Markdown body (below the frontmatter) serves as the agent's instructions.

The `.md` file is compiled into a `.lock.yml` file using `gh aw compile`. The `.lock.yml` is the actual GitHub Actions workflow that runs. **Never edit the `.lock.yml` directly** -- always edit the `.md` source and recompile.

## Customizing for Your Own Models

To use this template with your own semantic models:

1. Replace the sample `*.SemanticModel`, `*.Report`, and `*.pbip` files with your own Power BI project files.
2. Follow the [Setup Guide](#setup-guide) to configure your secrets and permissions.
3. Push to `main` and let the agentic workflow generate documentation for your models.

## Troubleshooting

| Problem | Solution |
|---|---|
| Agent fails to connect to MCP server | Ensure the [Docker Hub container image](https://hub.docker.com/repository/docker/kerski/powerbi-modeling-mcp/general) is accessible and the workflow has network access to Docker Hub. |
| Agent cannot create a pull request | Check that **Workflow permissions** allow creating PRs (see [Step 2](#step-2-grant-github-actions-permission-to-create-pull-requests)). |
| Workflow doesn't trigger on push | Verify the push was to `main` and the changed files match `*.SemanticModel/**`. |
| `COPILOT_GITHUB_TOKEN` error | Ensure the secret is set correctly and the token has the **Copilot** account permission set to **Read-only** (see [Step 1](#step-1-set-up-the-copilot_github_token)). |

* If you've edited the generate-pbi-docs.md file

| Problem | Solution |
|---|---|
| Compiled `.lock.yml` is outdated | Run `gh aw compile` after editing the `.md` file and commit both files. |

## Acknowledgments

- [Getting Started with GitHub Agentic Workflows: A Hands-On Guide](https://josh-ops.com/posts/github-agentic-workflows/) by josh-ops -- for the excellent walkthrough on configuring GitHub Agentic Workflows.

## License

This project is licensed under the MIT License - see the LICENSE file for details.

---

*Note: Some content in this repository was generated with AI assistance and then subsequently reviewed and revised.*

For additional insights, tutorials, and best practices related to Microsoft Fabric and Power BI, visit [blog.kerski.tech](https://blog.kerski.tech).
