# AM Portfolio Organization (.github)

This repository contains organization-wide default configurations, community health files, and global workflows for the **AM Portfolio** ecosystem.

## 🚀 Global Features

### 🤖 Gemini PR Agent (Auto-Enabled)
Every repository in this organization is automatically integrated with a **Google Gemini 1.5 Flash** powered PR reviewer.

- **How it works**: A default workflow in this repository (`.github/workflows/pr-agent.yml`) triggers on every Pull Request in the organization and calls the central logic in [am-pipelines](https://github.com/AM-Portfolio/am-pipelines).
- **Features**:
  - **Auto-Description**: Automatically generates a clear summary of the PR.
  - **Auto-Review**: Provides an AI-powered code review with suggestions.
  - **Manual Commands**: Respond to any PR with `/review`, `/describe`, or `/ask <question>` to interact with the agent.

## 🛠 Setup & Requirements

- **Secrets**: This repo relies on the following Organization Secrets:
  - `GOOGLE_API_KEY`: Required for Gemini model access.
  - `GH_TOKEN` or `GHCR_TOKEN`: Required for the agent to post comments and manage labels.

---
*Managed by the AM-Portfolio Infrastructure Team.*