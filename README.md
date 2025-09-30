
# Google Vertex AI Agent with MCP Security Tools

Deploy security-focused AI agents to Google Cloud with integrated access to Chronicle, SOAR, Threat Intelligence, and Security Command Center through the Model Context Protocol (MCP).

## Table of Contents

  - [Quick Start](https://www.google.com/search?q=%23quick-start)
  - [Overview](https://www.google.com/search?q=%23overview)
  - [Architecture](https://www.google.com/search?q=%23architecture)
  - [Prerequisites](https://www.google.com/search?q=%23prerequisites)
  - [Setup and Installation](https://www.google.com/search?q=%23setup-and-installation)
  - [Deployment Workflow](https://www.google.com/search?q=%23deployment-workflow)
  - [Running the Application](https://www.google.com/search?q=%23running-the-application)
  - [Troubleshooting](https://www.google.com/search?q=%23troubleshooting)
  - [Configuration](https://www.google.com/search?q=%23configuration)
  - [Usage](https://www.google.com/search?q=%23usage)
  - [Makefile Reference](https://www.google.com/search?q=%23makefile-reference)
  - [Development](https://www.google.com/search?q=%23development)
  - [FAQ](https://www.google.com/search?q=%23faq)
  - [Best Practices](https://www.google.com/search?q=%23best-practices)
  - [Documentation](https://www.google.com/search?q=%23documentation)
  - [Support](https://www.google.com/search?q=%23support)

## Quick Start

```bash
# Clone the repository and its submodules
git clone --recurse-submodules https://github.com/acidack/agentic_soc_agentspace.git
cd agentic_soc_agentspace

# Create your environment configuration from the template
cp .env.example .env

# IMPORTANT: Edit the .env file with your project details now.
# See the "Setup and Installation" section for critical variables to set.

# Set up a Python virtual environment and install dependencies
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Run the first deployment step
make agent-engine-deploy
```

For the full process, see the [Deployment Workflow](https://www.google.com/search?q=%23deployment-workflow) section below.

## Overview

This project enables you to:

  - Deploy AI agents to Google Vertex AI Agent Engine with security tool access.
  - Integrate with Google Security Operations (Chronicle) for threat detection.
  - Connect to SOAR platforms for automated response workflows.
  - Access Google Threat Intelligence for IOC analysis.
  - Monitor cloud security posture via Security Command Center.

## Architecture

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│    main.py      │───▶│   Vertex AI      │───▶│    MCP Security       │
│  Agent Creator  │     │   Agent Engine   │     │    Tools (stdio)      │
└─────────────────┘     └──────────────────┘     └─────────────────────┘
                                  │                       │
┌─────────────────┐               │              ┌───────▼───────┐
│test_agent_engine│◀──────────────┘              │ • Chronicle   │
│     Testing     │                               │ • SOAR        │
└─────────────────┘                               │ • GTI         │
                                                  │ • SCC         │
                                                  └───────────────┘
```

## Prerequisites

### System Requirements

  - **Python 3.10+**
  - **Git** with submodules support
  - **Google Cloud SDK** installed and authenticated (`gcloud auth login`)

### Google Cloud Requirements

  - An active GCP project with billing enabled.
  - Your Project ID and Project Number.

### Required APIs

Enable these APIs in your project. It is recommended to do this first.

```bash
gcloud services enable aiplatform.googleapis.com discoveryengine.googleapis.com storage.googleapis.com cloudbuild.googleapis.com
```

### Required IAM Permissions

To avoid errors, ensure your **user account** (the one you log into the console with) has the following IAM roles:

  - **Project Owner** or **Editor** (simplest for setup).
  - If using granular roles, you need at least:
      - `Vertex AI Administrator`
      - **`Vertex AI Search Admin`** - **CRITICAL\!** This is required to access the AgentSpace UI and will fix `403 Permission Denied` errors.
      - `Storage Admin` - To manage the GCS staging bucket.
      - `Service Account Admin` - To manage service accounts.

## Setup and Installation

### 1\. Clone Repository

Clone the repository and its `mcp-security` submodule.

```bash
git clone --recurse-submodules https://github.com/acidack/agentic_soc_agentspace.git
cd agentic_soc_agentspace
```

### 2\. Configure Your Environment (`.env` file)

This is the most critical step to prevent deployment errors.

1.  Create your personal configuration file:
    ```bash
    cp .env.example .env
    ```
2.  Open and edit the `.env` file, paying close attention to these variables:
      - `PROJECT_ID` & `PROJECT_NUMBER`: Your Google Cloud project details.
      - `LOCATION`: Your chosen deployment region (e.g., `us-central1`).
      - `GOOGLE_CLOUD_LOCATION`: **(Required)** Add this variable and set it to the same value as `LOCATION`.
      - `STAGING_BUCKET`: **(Required)** Must start with the `gs://` prefix (e.g., `gs://my-soc-agent-bucket`).
      - `AGENTSPACE_PROJECT_ID` & `AGENTSPACE_PROJECT_NUMBER`: **(Required)** Do not leave blank. Set these to the same values as your main `PROJECT_ID` and `PROJECT_NUMBER`.
      - Fill in other Stage 1 variables for any security tools you intend to use.

### 3\. Install Dependencies

Use a Python virtual environment to avoid conflicts.

```bash
# Create and activate the environment
python3 -m venv venv
source venv/bin/activate

# Install required packages
pip install -r requirements.txt
```

## Deployment Workflow

The deployment is a two-stage process. You will run a `make` command, then update your `.env` file with the output.

### Stage 1: Deploy the Agent Engine

This command packages and deploys your agent to a managed backend on Vertex AI.

```bash
make agent-engine-deploy
```

After it completes, it will output a `REASONING_ENGINE` ID and a full `AGENT_ENGINE_RESOURCE_NAME`. **Immediately update your `.env` file** with these new values in the **STAGE 2** section.

### Stage 2: Register the Agent with AgentSpace

This command links your deployed backend to the AgentSpace UI.

```bash
make agentspace-register
```

After it completes, it will output an `AGENTSPACE_AGENT_ID`. Update your `.env` file with this new value in the **STAGE 3** section.

## Running the Application

Once everything is deployed and configured, run the local Streamlit user interface.

```bash
make run-ui
```

This will provide a local URL (e.g., `http://localhost:8501`) to access the agent's chat interface.

## Troubleshooting

| Issue | Solution |
|-------|----------|
| **403 Permission Denied in UI** | Your **user account** is missing the **`Vertex AI Search Admin`** IAM role. See the Prerequisites section. |
| **How to Update / "Restart" Agent?** | To apply any change (`.env` or code), you must re-run `make agent-engine-deploy`. This creates a **new** engine, so you **must** update the `REASONING_ENGINE` and `AGENT_ENGINE_RESOURCE_NAME` in your `.env` file afterward. |
| **403/401 Auth Error in Terminal** | Run `gcloud auth application-default login` to refresh your credentials. |
| **KeyError or Missing Variable** | Ensure your `.env` file is correctly created from `.env.example` and all required Stage 1 variables are filled out. |
| **`mcp-security` module missing** | The submodule was not cloned correctly. Run `git submodule update --init --recursive`. |
| **Agent not in AgentSpace** | Run `make agentspace-verify` to check the status, then `make agentspace-link-agent` if needed. |
| **Agent not responding** | Check the logs of the deployed agent: `gcloud logging tail "resource.type=aiplatform.googleapis.com/ReasoningEngine"` |

## Configuration

See `.env.example` for all available environment variables. The variables are grouped into three stages, corresponding to the deployment workflow.

## Usage

### Test Deployed Agent via CLI

```bash
# Ensure REASONING_ENGINE is set in .env from deployment output
python test_agent_engine.py
```

### Example Queries

```
"List active SOAR cases"
"What are the recent high-severity security alerts in Chronicle?"
"Analyze the IOCs for the domain malicious.com using GTI"
```

## Makefile Reference

Run `make help` to see all available commands with descriptions.

  - `make full-deploy`: Runs `agent-engine-deploy` and `agentspace-register` sequentially.
  - `make redeploy-all`: Re-deploys the agent and updates its registration in AgentSpace.

## Development

### Customizing the Agent

Edit `main.py` to customize the agent's model, instructions, or active tools.

```python
# Change the model used
model = "gemini-1.5-flash-001"

# Modify the system prompt
instruction = "You are a helpful security analyst..."

# Select specific tools to activate
tools = [secops_siem_tools, soar_tools]
```

Remember to run `make agent-engine-deploy` to apply any changes.

## FAQ

**Can I use this without a SOAR platform?**
Yes. All security tool integrations are optional. The agent will only use the tools that are configured in the `.env` file.

**How do I update the agent after pulling new code?**

```bash
git pull
make agent-engine-deploy
# Remember to update your .env with the new engine ID
```

**What are the estimated costs?**
Costs are primarily driven by Vertex AI API calls (per token usage). If you use other Google Cloud services (Chronicle, SCC), they have their own pricing. Set budget alerts for your project.

## Best Practices

  - **Security**: Use Google Cloud Secret Manager for sensitive credentials instead of storing them in `.env` for production use. Apply least-privilege IAM roles.
  - **Development**: Use separate Google Cloud projects for development and production environments.
  - **Operations**: Monitor API usage and quotas in the Google Cloud Console.

## Documentation

  - [MCP Security Docs](https://www.google.com/search?q=mcp-security/README.md) - Documentation for the underlying security tools.
  - [Google Vertex AI Docs](https://cloud.google.com/vertex-ai/docs) - Official platform documentation.
  - [MCP Protocol](https://modelcontextprotocol.io/) - Protocol specification.

## Support

For issues with this specific project, please open a [GitHub Issue](https://www.google.com/search?q=https://github.com/acidack/agentic_soc_agentspace/issues). For general support on the underlying technologies, see the official documentation or community forums like Stack Overflow.
