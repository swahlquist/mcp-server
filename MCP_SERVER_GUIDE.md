# MCP Server Developer Guide

## Table of Contents
1. [What is This Server?](#what-is-this-server)
2. [What is MCP?](#what-is-mcp)
3. [Architecture Overview](#architecture-overview)
4. [Current Implementation](#current-implementation)
5. [How It Works](#how-it-works)
6. [Getting Started](#getting-started)
7. [Development Workflow](#development-workflow)
8. [Deployment](#deployment)
9. [Security Considerations](#security-considerations)
10. [Troubleshooting](#troubleshooting)
11. [FAQ](#faq)
12. [Resources](#resources)

---

## What is This Server?

This is an **MCP (Model Context Protocol) server** designed for AI agent orchestration. It provides a standardized interface for AI agents (like Claude, Gemini CLI, and other LLM-based tools) to interact with your systems, data, and services.

### Purpose

**For AI Agents:**
- Discover available tools and capabilities via the protocol
- Execute actions on external systems securely
- Access data and resources through standardized interfaces

**For Developers:**
- Build tools that AI agents can use
- Connect AI systems to proprietary data and services
- Create reproducible, testable AI workflows

**For the Organization:**
- Enable AI-driven automation at scale
- Maintain security and audit trails
- Provide controlled access to systems for AI agents

---

## What is MCP?

**Model Context Protocol (MCP)** is an open-source standard created by Anthropic that enables AI applications to securely connect to external systems and data sources.

### Key Concepts

#### 1. **MCP Servers** (This Project)
Expose capabilities (tools, resources, prompts) that AI agents can use. Think of it as an API specifically designed for AI agents to consume.

#### 2. **MCP Clients**
AI applications (Claude Desktop, Gemini CLI, custom agents) that connect to MCP servers to access capabilities.

#### 3. **Tools**
Discrete operations that agents can invoke to accomplish tasks. Unlike traditional APIs, tools are designed around **what users want to accomplish**, not just technical operations.

**Example:**
- ❌ Bad Tool: `POST /api/database/query` (too technical)
- ✅ Good Tool: `analyze_sales_trends` (user goal-oriented)

#### 4. **Resources**
Data or content that agents can read to get context (documents, database records, file contents).

#### 5. **Prompts**
Pre-defined workflows or templates that help guide agent behavior.

### Why MCP?

**Before MCP:**
- Every AI integration was custom-built
- No standardization across tools
- Security and authentication varied wildly
- Hard to maintain and scale

**With MCP:**
- ✅ Standardized protocol for all AI integrations
- ✅ Built-in security and authentication patterns
- ✅ Reusable tools across different AI agents
- ✅ Easier testing and debugging
- ✅ Open-source ecosystem with community contributions

---

## Architecture Overview

### High-Level Architecture

```
┌─────────────────┐
│   AI Agents     │
│  (Claude, etc)  │
└────────┬────────┘
         │ MCP Protocol
         │ (JSON-RPC)
         ▼
┌─────────────────┐
│   MCP Server    │◄──── This Project
│   (Flask App)   │
└────────┬────────┘
         │
         ├─── Tool Registry
         ├─── Authentication
         ├─── Tool Implementations
         └─── External Services
                  │
                  ├─── Databases
                  ├─── APIs
                  ├─── File Systems
                  └─── Other Services
```

### Components

#### 1. **Flask Application** (`main.py`)
- Web server that handles HTTP requests
- Entry point for all MCP communication
- Routes requests to appropriate handlers

#### 2. **Tool Registry** (`tool_registry.json`)
- JSON file listing all available tools
- Includes tool schemas, descriptions, parameters
- Used by agents for capability discovery

#### 3. **Transport Layer**
MCP supports two transport mechanisms:

**stdio (Standard Input/Output):**
- For local, single-user scenarios
- Process-to-process communication
- Best for development and testing

**Streamable HTTP:**
- For networked, multi-user scenarios
- Supports horizontal scaling
- Production-ready
- Replaces deprecated SSE transport

#### 4. **Tool Implementations**
- Python functions that execute when tools are called
- Handle business logic
- Interact with external services

---

## Current Implementation

### File Structure

```
mcp-server/
├── main.py                 # Flask app entry point
├── requirements.txt        # Python dependencies
├── tool_registry.json      # Tool definitions (currently empty)
├── render.yaml            # Render deployment config
├── .gitignore             # Git ignore rules
├── README.md              # Project overview
├── MCP_IMPROVEMENT_PLAN.md    # Improvement roadmap
└── MCP_SERVER_GUIDE.md        # This document
```

### Current State

**✅ What's Working:**
- Basic Flask server running
- Compatible dependency versions (Flask 3.0.0, Werkzeug 3.0.0)
- Render deployment configured
- Git version control set up

**⚠️ What's Missing:**
- Actual MCP protocol implementation
- Tool implementations
- Authentication/authorization
- Error handling
- Logging and monitoring
- Tests
- Production security measures

**🚨 Critical Issues:**
- Debug mode enabled (security risk)
- No authentication
- Empty tool registry
- No input validation

---

## How It Works

### Request Flow

```
1. Agent Discovery Phase:
   Agent → GET /tools → Server
   Server → Returns tool_registry.json
   Agent understands available capabilities

2. Tool Invocation Phase:
   Agent → POST /tools/execute
           {
             "tool": "analyze_data",
             "parameters": {...},
             "request_id": "req_123"
           }
   Server → Validates auth
   Server → Validates input
   Server → Executes tool
   Server → Returns result or error

3. Result Processing:
   Agent receives structured response
   Agent decides next action based on result
```

### MCP Protocol Basics

MCP uses **JSON-RPC 2.0** for communication:

**Example Tool Call:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "analyze_sales_trends",
    "arguments": {
      "start_date": "2024-01-01",
      "end_date": "2024-12-31",
      "region": "north_america",
      "request_id": "req_abc123"
    }
  }
}
```

**Example Success Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Sales increased 23% YoY in North America..."
      }
    ],
    "isError": false
  }
}
```

**Example Error Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": "INVALID_PARAMETERS",
    "message": "Invalid date range: end_date must be after start_date",
    "data": {
      "suggested_action": "Check date parameters and retry",
      "correlation_id": "req_abc123"
    }
  }
}
```

---

## Getting Started

### Prerequisites

- Python 3.11+
- pip (Python package manager)
- Git
- Text editor or IDE
- (Optional) Render account for deployment

### Local Setup

#### 1. Clone the Repository

```bash
git clone <repository-url>
cd mcp-server
```

#### 2. Create Virtual Environment (Recommended)

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

#### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

#### 4. Set Up Environment Variables

Create a `.env` file (never commit this!):

```bash
# .env
ENV=development
PORT=8080
SECRET_KEY=your-secret-key-here
API_KEY=your-api-key-here

# Add other service credentials as needed
DATABASE_URL=postgresql://...
REDIS_URL=redis://...
```

#### 5. Run the Server

```bash
python main.py
```

Server will start at `http://localhost:8080`

#### 6. Test the Server

```bash
curl http://localhost:8080/
# Should return: "Hello, World!"
```

---

## Development Workflow

### Adding a New Tool

#### Step 1: Define the Tool Schema

Edit `tool_registry.json`:

```json
[
  {
    "name": "get_weather",
    "description": "Get current weather for a location. Use this when the user asks about weather conditions.",
    "inputSchema": {
      "type": "object",
      "properties": {
        "location": {
          "type": "string",
          "description": "City name or zip code"
        },
        "units": {
          "type": "string",
          "enum": ["celsius", "fahrenheit"],
          "description": "Temperature units"
        },
        "request_id": {
          "type": "string",
          "description": "Client-generated unique ID for idempotency"
        }
      },
      "required": ["location"]
    }
  }
]
```

**Key Points:**
- `description` should tell the agent **when** to use the tool
- Define all parameters with types and descriptions
- Mark required fields
- Always include `request_id` for idempotency

#### Step 2: Implement the Tool

Create a new file or add to `main.py`:

```python
from flask import request, jsonify
import requests

@app.route("/tools/get_weather", methods=["POST"])
def get_weather():
    """Get current weather for a location"""

    # 1. Extract and validate input
    data = request.get_json()
    location = data.get("location")
    units = data.get("units", "celsius")
    request_id = data.get("request_id")

    if not location:
        return jsonify({
            "error_code": "MISSING_PARAMETER",
            "message": "Parameter 'location' is required",
            "suggested_action": "Provide a location parameter and retry"
        }), 400

    # 2. Check cache for idempotency (if request_id seen before)
    # cached_result = check_cache(request_id)
    # if cached_result:
    #     return cached_result

    # 3. Execute business logic
    try:
        # Call external weather API
        weather_data = fetch_weather_from_api(location, units)

        result = {
            "location": location,
            "temperature": weather_data["temp"],
            "conditions": weather_data["conditions"],
            "units": units
        }

        # 4. Cache result
        # cache_result(request_id, result)

        # 5. Return result
        return jsonify(result), 200

    except Exception as e:
        # 6. Handle errors helpfully
        return jsonify({
            "error_code": "WEATHER_API_ERROR",
            "message": f"Failed to fetch weather data: {str(e)}",
            "suggested_action": "Check location spelling or try again later",
            "correlation_id": request_id
        }), 500

def fetch_weather_from_api(location, units):
    """Helper function to call weather API"""
    api_key = os.environ.get("WEATHER_API_KEY")
    response = requests.get(
        f"https://api.weather.com/v1/current",
        params={"location": location, "units": units, "key": api_key}
    )
    response.raise_for_status()
    return response.json()
```

#### Step 3: Test the Tool

```bash
# Test locally
curl -X POST http://localhost:8080/tools/get_weather \
  -H "Content-Type: application/json" \
  -d '{"location": "San Francisco", "units": "celsius", "request_id": "req_123"}'
```

#### Step 4: Write Tests

```python
# tests/test_weather.py
def test_get_weather_success():
    response = client.post('/tools/get_weather', json={
        'location': 'San Francisco',
        'units': 'celsius',
        'request_id': 'test_123'
    })
    assert response.status_code == 200
    assert 'temperature' in response.json

def test_get_weather_missing_location():
    response = client.post('/tools/get_weather', json={
        'request_id': 'test_124'
    })
    assert response.status_code == 400
    assert response.json['error_code'] == 'MISSING_PARAMETER'
```

### Best Practices for Tool Development

#### 1. **Design for Agents, Not Humans**
- Think about what the agent needs to accomplish
- Provide clear, actionable descriptions
- Return structured, parseable data

#### 2. **Make Tools Idempotent**
- Accept `request_id` parameter
- Cache results by `request_id`
- Return same result if called multiple times with same ID
- Safe to retry without side effects

#### 3. **Error Handling**
```python
# ❌ Bad: Unhelpful error
return {"error": "Failed"}

# ✅ Good: Agent-helpful error
return {
    "error_code": "INVALID_DATE",
    "message": "Date must be in YYYY-MM-DD format",
    "suggested_action": "Use format: 2024-12-31",
    "examples": ["2024-01-01", "2024-12-31"]
}
```

#### 4. **Input Validation**
```python
from pydantic import BaseModel, Field

class WeatherInput(BaseModel):
    location: str = Field(..., min_length=1, max_length=100)
    units: str = Field(default="celsius", pattern="^(celsius|fahrenheit)$")
    request_id: str = Field(..., min_length=1)

# Use in endpoint
input_data = WeatherInput(**request.get_json())
```

#### 5. **Logging**
```python
import logging

logger = logging.getLogger(__name__)

logger.info(f"Weather request for {location}", extra={
    "request_id": request_id,
    "location": location,
    "user_id": current_user.id
})
```

---

## Deployment

### Deploying to Render

This server is configured for deployment on [Render](https://render.com).

#### 1. Push to Git

```bash
git add .
git commit -m "Add weather tool"
git push origin main
```

#### 2. Connect to Render

1. Log in to Render dashboard
2. Click "New +" → "Web Service"
3. Connect your Git repository
4. Render will auto-detect `render.yaml`

#### 3. Configure Environment Variables

In Render dashboard:
- Go to your service → Environment
- Add all variables from `.env`:
  - `ENV=production`
  - `SECRET_KEY=...`
  - `API_KEY=...`
  - etc.

#### 4. Deploy

Render will automatically:
- Install dependencies from `requirements.txt`
- Run `python main.py`
- Provide a public URL

#### 5. Verify Deployment

```bash
curl https://your-app.onrender.com/health
```

### Production Checklist

Before deploying to production:

- [ ] Disable debug mode (`debug=False`)
- [ ] Set strong `SECRET_KEY`
- [ ] Enable authentication
- [ ] Set up logging/monitoring
- [ ] Configure rate limiting
- [ ] Enable HTTPS (Render does this automatically)
- [ ] Test all tools
- [ ] Document all environment variables
- [ ] Set up error alerting (Sentry, etc.)
- [ ] Configure backups (if using database)

---

## Security Considerations

### Current Security Risks

🚨 **Critical Issues to Fix:**

1. **Debug Mode Enabled**
   - Exposes stack traces and internal info
   - Can leak sensitive data
   - **Fix:** Set `debug=False` in production

2. **No Authentication**
   - Anyone can call tools
   - No user context or audit trail
   - **Fix:** Implement OAuth 2.1 or API key auth

3. **No Input Validation**
   - Vulnerable to injection attacks
   - Can cause crashes or unexpected behavior
   - **Fix:** Use Pydantic for validation

4. **No Rate Limiting**
   - Vulnerable to DoS attacks
   - Can rack up API costs
   - **Fix:** Implement rate limiting per user/IP

### Security Best Practices

#### 1. **Authentication**

Implement OAuth 2.1 flow:

```python
from authlib.integrations.flask_oauth2 import ResourceProtector
from authlib.oauth2.rfc6749 import grants

# Set up OAuth provider
require_oauth = ResourceProtector()

@app.route("/tools/get_weather", methods=["POST"])
@require_oauth("tools:read")
def get_weather():
    current_user = request.oauth.user
    # Tool implementation...
```

#### 2. **Input Sanitization**

```python
import bleach
from html import escape

def sanitize_input(user_input):
    # Remove HTML tags
    clean = bleach.clean(user_input, tags=[], strip=True)
    # Escape special characters
    return escape(clean)
```

#### 3. **Environment Variables**

Never commit secrets:

```python
# ❌ Bad
API_KEY = "sk_live_123456"

# ✅ Good
API_KEY = os.environ.get("API_KEY")
if not API_KEY:
    raise ValueError("API_KEY environment variable required")
```

#### 4. **HTTPS Only**

```python
@app.before_request
def force_https():
    if request.headers.get('X-Forwarded-Proto') == 'http':
        return redirect(request.url.replace('http://', 'https://'))
```

#### 5. **Rate Limiting**

```python
from flask_limiter import Limiter

limiter = Limiter(
    app,
    key_func=lambda: request.headers.get("X-API-Key"),
    default_limits=["100 per hour"]
)

@app.route("/tools/get_weather")
@limiter.limit("10 per minute")
def get_weather():
    # Tool implementation...
```

---

## Troubleshooting

### Common Issues

#### Issue: Server Won't Start

**Symptoms:**
```
ModuleNotFoundError: No module named 'flask'
```

**Solution:**
```bash
pip install -r requirements.txt
```

---

#### Issue: Import Error with Werkzeug

**Symptoms:**
```
ImportError: cannot import name 'url_quote' from 'werkzeug.urls'
```

**Solution:**
Update `requirements.txt`:
```
Flask==3.0.0
Werkzeug==3.0.0
```

Then reinstall:
```bash
pip install -r requirements.txt --force-reinstall
```

---

#### Issue: Tools Not Showing Up for Agent

**Symptoms:**
Agent says "No tools available" or doesn't see your tools.

**Solution:**
1. Check `tool_registry.json` is valid JSON
2. Ensure tool schemas are complete
3. Verify agent is configured to use your server URL
4. Check authentication is working

---

#### Issue: "Debug mode should not be used in production"

**Symptoms:**
Warning message in logs.

**Solution:**
In `main.py`:
```python
if __name__ == "__main__":
    app.run(
        debug=os.environ.get("ENV") != "production",
        host="0.0.0.0",
        port=int(os.environ.get("PORT", 8080))
    )
```

---

#### Issue: Agent Gets Errors Calling Tools

**Symptoms:**
Agent reports errors or unexpected responses.

**Debug Steps:**
1. Check server logs for errors
2. Test tool manually with curl
3. Verify input validation
4. Check authentication headers
5. Ensure error responses are structured properly

---

## FAQ

### Q: What's the difference between MCP and REST APIs?

**REST API:**
- Designed for human developers
- CRUD operations (Create, Read, Update, Delete)
- Technical, resource-oriented

**MCP:**
- Designed for AI agents
- Goal-oriented operations
- Standardized protocol
- Built-in discovery and introspection

### Q: Can I use this with Claude Desktop?

Yes! Claude Desktop supports MCP servers. Configure it to connect to your server URL in Claude Desktop settings.

### Q: Should I use stdio or HTTP transport?

- **stdio**: Local development, single-user, command-line tools
- **Streamable HTTP**: Production, multi-user, web-accessible, scalable

### Q: How do I know if my tool design is good?

Test it with an actual AI agent! If the agent:
- ✅ Can understand when to use it
- ✅ Gets useful results
- ✅ Can handle errors gracefully
- ✅ Accomplishes user goals efficiently

Then your tool is well-designed.

### Q: What's idempotency and why does it matter?

**Idempotency:** Calling the same operation multiple times produces the same result.

**Why it matters:** AI agents may retry failed requests or parallelize operations. If your tool isn't idempotent, this could cause:
- Duplicate charges
- Inconsistent state
- Data corruption

**Solution:** Use `request_id` to detect and handle duplicate requests.

### Q: How do I handle long-running operations?

For operations that take >30 seconds:

1. Return immediately with a job ID
2. Provide a separate tool to check status
3. Consider using webhooks to notify when complete

```json
{
  "status": "processing",
  "job_id": "job_789",
  "check_status_with": "check_job_status",
  "estimated_time": "2-3 minutes"
}
```

---

## Resources

### Official Documentation
- [Anthropic MCP Documentation](https://docs.anthropic.com/en/docs/mcp)
- [MCP GitHub Organization](https://github.com/modelcontextprotocol)
- [MCP Specification](https://modelcontextprotocol.io/specification/draft/basic/security_best_practices)
- [Anthropic Training Course](https://anthropic.skilljar.com/introduction-to-model-context-protocol)

### Best Practices & Guides
- [MCP Best Practices: Architecture & Implementation](https://modelcontextprotocol.info/docs/best-practices/)
- [15 Best Practices for Production](https://thenewstack.io/15-best-practices-for-building-mcp-servers-in-production/)
- [MCP Development Guide](https://github.com/cyanheads/model-context-protocol-resources/blob/main/guides/mcp-server-development-guide.md)
- [Docker MCP Best Practices](https://www.docker.com/blog/mcp-server-best-practices/)

### Tutorials
- [DigitalOcean MCP Tutorial](https://www.digitalocean.com/community/tutorials/model-context-protocol)
- [Beginner's Guide to MCP](https://dev.to/hussain101/a-beginners-guide-to-anthropics-model-context-protocol-mcp-1p86)
- [MCP Complete Tutorial](https://medium.com/@nimritakoul01/the-model-context-protocol-mcp-a-complete-tutorial-a3abe8a7f4ef)

### Security
- [MCP Security Survival Guide](https://towardsdatascience.com/the-mcp-security-survival-guide-best-practices-pitfalls-and-real-world-lessons/)
- [MCP Security Risks & Controls](https://www.redhat.com/en/blog/model-context-protocol-mcp-understanding-security-risks-and-controls)

### Community
- GitHub Discussions: https://github.com/modelcontextprotocol/discussions
- Check for community Slack/Discord channels
- Follow Anthropic's announcements for updates

---

## Next Steps

Now that you understand how the MCP server works:

1. **Review the [Improvement Plan](./MCP_IMPROVEMENT_PLAN.md)** for detailed roadmap
2. **Set up your local environment** following the Getting Started guide
3. **Implement your first tool** using the development workflow
4. **Test with an AI agent** (Claude Desktop, Gemini CLI, etc.)
5. **Deploy to Render** when ready
6. **Monitor and iterate** based on agent usage patterns

---

**Questions or Issues?**
- Check the [Troubleshooting](#troubleshooting) section
- Review the [FAQ](#faq)
- Consult official Anthropic documentation
- Open an issue in the GitHub repository

**Document Version:** 1.0
**Last Updated:** 2025-12-04
**Maintained By:** Development Team
