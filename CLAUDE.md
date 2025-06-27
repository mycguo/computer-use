# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Python-based computer use demo application that provides a Streamlit web interface for interacting with Claude's computer use capabilities. The application allows users to give natural language instructions to Claude, which can then control a computer through screenshots, mouse actions, keyboard input, and bash commands.

## Project Structure

```
computer-use/
├── app.py              # Main Streamlit application entry point
├── loop.py             # Core agent sampling loop and API interaction
├── requirements.txt    # Python dependencies
└── tools/              # Tool implementations for computer interaction
    ├── __init__.py     # Tool exports and imports
    ├── base.py         # Abstract base classes and tool result types
    ├── bash.py         # Bash command execution tools
    ├── computer.py     # Computer control tools (mouse, keyboard, screenshot)
    ├── edit.py         # File editing tools
    ├── collection.py   # Tool collection management
    ├── groups.py       # Tool version groups and configurations
    └── run.py          # Command execution utilities
```

## Development Commands

### Setup and Installation
```bash
# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run the application
streamlit run app.py
```

### Environment Configuration
```bash
# Required environment variables
export ANTHROPIC_API_KEY="your-api-key-here"

# Optional configuration
export WIDTH=1280          # Screen width for computer use
export HEIGHT=800          # Screen height for computer use
export HIDE_WARNING=false  # Hide security warning in UI
export API_PROVIDER=anthropic  # anthropic|bedrock|vertex
```

### AWS Bedrock Setup (if using)
```bash
# Configure AWS credentials
aws configure
# or set environment variables
export AWS_ACCESS_KEY_ID="your-key"
export AWS_SECRET_ACCESS_KEY="your-secret"
export AWS_DEFAULT_REGION="us-east-1"
```

### Google Vertex AI Setup (if using)
```bash
# 1. Enable required APIs
gcloud services enable aiplatform.googleapis.com
gcloud services enable cloudresourcemanager.googleapis.com

# 2. Grant required IAM permissions (choose one option):

# Option A: For your user account (local development)
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
    --member="user:YOUR_EMAIL@gmail.com" \
    --role="roles/aiplatform.user"

# Option B: For service account (cloud deployment)
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
    --member="serviceAccount:YOUR_SERVICE_ACCOUNT@YOUR_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/aiplatform.user"

# 3. Set up authentication
# For local development:
gcloud auth application-default login
export CLOUD_ML_REGION="us-central1"         # Optional - defaults to us-central1 in code
export ANTHROPIC_VERTEX_PROJECT_ID="your-project-id"  # Optional - auto-detected from credentials

# For cloud deployment:
# The application automatically:
# - Sets CLOUD_ML_REGION to "us-central1" if not specified
# - Detects project ID from Google Cloud credentials if not specified
# Use Google Cloud service account authentication or workload identity

# Note: Vertex AI with Anthropic models requires billing to be enabled on your project
```

## Architecture Overview

### Core Components

**Streamlit Application (`app.py`)**
- Web interface for interacting with the computer use system
- Handles user input, API key management, and configuration
- Supports multiple API providers (Anthropic, Bedrock, Vertex)
- Manages conversation history and tool outputs

**Agent Loop (`loop.py`)**
- Core sampling loop that orchestrates API calls and tool execution
- Handles different API providers with unified interface
- Manages prompt caching and image optimization
- Processes tool results and maintains conversation state

**Tool System (`tools/`)**
- Modular tool architecture with versioned implementations
- Three main tool categories:
  - **Computer Tools**: Screenshot, mouse, keyboard control
  - **Bash Tools**: Command line execution
  - **Edit Tools**: File editing and manipulation
- Tool collections organized by API version compatibility

### Key Patterns

**Tool Versioning**
- Tools are versioned to match API capabilities (`20241022`, `20250124`, `20250429`)
- Different tool versions provide different feature sets
- Tool groups bundle compatible tools for specific API versions

**API Provider Abstraction**
- Unified interface supports Anthropic, AWS Bedrock, and Google Vertex
- Provider-specific authentication and configuration
- Consistent tool parameter formatting across providers

**Result Handling**
- `ToolResult` dataclass for structured tool outputs
- Support for text output, errors, images, and system messages
- Tool results can be combined and rendered in the UI

## Configuration Files

- `requirements.txt` - Python package dependencies
- `.venv/pyvenv.cfg` - Virtual environment configuration
- `~/.anthropic/api_key` - Stored API key (optional)
- `~/.anthropic/system_prompt` - Custom system prompt suffix (optional)

## Important Development Notes

### Security Considerations
- API keys are stored securely with proper file permissions (600)
- Security warning displayed in UI by default
- Never commit API keys or sensitive credentials

### Tool Implementation
- All tools inherit from `BaseAnthropicTool`
- Tools must implement `__call__` and `to_params` methods
- Use `ToolResult` for consistent output formatting
- Handle errors gracefully with appropriate error messages

### Testing
- **No formal testing framework is currently set up**
- Manual testing through the Streamlit interface
- To add testing, consider `pytest` and `pytest-asyncio`

### Performance Considerations
- Prompt caching enabled for Anthropic API to reduce costs
- Image truncation available to manage token usage
- Async execution for responsive UI during long operations

## Model Support

The application supports various Claude models with different capabilities:
- **Claude 3.5 Sonnet**: Standard computer use features
- **Claude 3.7 Sonnet**: Enhanced capabilities with thinking mode
- **Claude 4**: Latest model with advanced reasoning and thinking

Each model has specific token limits and feature sets configured in `app.py`.