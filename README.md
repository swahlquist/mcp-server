# MCP Server for Gemini CLI Agent Orchestration

This repository hosts a minimal Flask server designed for agent orchestration via Gemini CLI and other AI tools. It provides a clean, secure interface for exposing callable tools, validating agent inputs, and enabling reproducible workflows across contributors.

## 🔧 What It Does

- Exposes tools via HTTP endpoints for Gemini CLI and Manus AI agents
- Hosts a `tool_registry.json` for agent introspection and schema validation
- Supports modular, agent-free testing via `main.py`
- Deploys seamlessly to Render for public access

## 🧠 Why It Exists

This MCP (Modular Command Processor) server is part of a broader effort to make AI agent workflows:
- **Contributor-friendly**: Easy to onboard, test, and extend
- **Modular**: Tools are isolated, auditable, and reusable
- **Secure**: No secrets in Git history; `.env` is excluded and managed locally
- **Agent-ready**: Compatible with Gemini CLI, Claude, Manus, and other orchestration platforms

## 🚀 How to Use It

### For Contributors:
- Clone the repo and run `main.py` locally to simulate agent calls
- Add new tools to `tool_registry.json` and expose them via Flask routes
- Use `requirements.txt` to manage dependencies

### For Agents:
- Gemini CLI can call tools via HTTP once deployed to Render
- Agents can introspect available tools via `tool_registry.json`
- Supports prompt chaining, validation, and debug workflows

## 🌐 Deployment

This server is ready for deployment to [Render](https://render.com). Once live, agents can access it via a public URL and begin orchestrating workflows.

## 📁 Key Files

- `main.py`: Flask server with exposed tools
- `tool_registry.json`: Tool definitions and schemas
- `requirements.txt`: Python dependencies
- `render.yaml`: Render deployment config
- `.gitignore`: Ensures `.env` and other sensitive files are excluded

---

This is the foundation for scalable, agent-driven automation. Whether you're testing locally or deploying to production, this repo gives you the tools to build, validate, and orchestrate AI workflows with confidence.
