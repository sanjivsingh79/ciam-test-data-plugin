# CIAM Test Data — Claude Code Plugin

A demonstration of Claude Code using a specialist agent, a Skill, an MCP server, and a local synthetic-data service to retrieve safe test identities for software testing.

## What this demonstrates

1. A developer asks Claude Code to generate a focused test.
2. The `test-data-agent` specialist selects test-data requirements.
3. The agent uses the `test-data-selection` Skill.
4. The `find_test_customer` MCP tool queries the local test-data service.
5. Claude uses the returned synthetic identity when generating a test.
6. The developer reviews and executes the generated test.

The plugin retrieves data; it does not itself generate Java tests or execute Gradle. Claude Code performs those development tasks.

## Architecture

```text
Developer
   |
   v
Claude Code
   |
   v
ciam-test-data plugin
   |-- Agent: test-data-agent
   |-- Skill: test-data-selection
   |-- MCP configuration (.mcp.json)
   |
   v
MCP server (127.0.0.1:9000)
   |
   v
Synthetic-data HTTP service (127.0.0.1:8090)
   |
   v
mock-test-data/test-data.json
```

## Repository layout

```text
.
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── .mcp.json
├── agents/
│   └── test-data-agent.md
├── skills/
│   └── test-data-selection/
│       └── SKILL.md
└── mock-test-data/
    ├── requirements.txt
    ├── test-data.json
    ├── mcp-server/
    │   ├── server.py
    │   └── test_client.py
    └── service/
        └── server.py
```

## Prerequisites

- Claude Code with plugin support
- Git
- Python 3.10 or newer
- Permission to bind localhost ports 8090 and 9000
- Network access to GitHub for marketplace installation

Review all plugin code before installing or running it. This is a demonstration project, not a production CIAM data service.

## 1. Clone the repository

```powershell
git clone https://github.com/sanjivsingh79/ciam-test-data-plugin.git
cd ciam-test-data-plugin
```

Replace the example repository URL with the actual published GitHub URL.

## 2. Set up Python

Create and activate a virtual environment:

```powershell
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r .\mock-test-data\requirements.txt
```

If PowerShell blocks activation, you can invoke the environment's Python directly:

```powershell
.\.venv\Scripts\python.exe -m pip install -r .\mock-test-data\requirements.txt
```

## 3. Start the synthetic-data service

Open a dedicated PowerShell terminal at the repository root:

```powershell
.\.venv\Scripts\python.exe .\mock-test-data\service\server.py
```

Expected startup message:

```text
Mock Test Data Service running on http://127.0.0.1:8090
Press Ctrl+C to stop.
```

Leave this terminal running.

## 4. Start the MCP server

Open a second PowerShell terminal at the same repository root:

```powershell
.\.venv\Scripts\python.exe .\mock-test-data\mcp-server\server.py
```

The MCP server listens on:

```text
http://127.0.0.1:9000/mcp
```

Leave this terminal running too.

The plugin's `.mcp.json` configures Claude Code to connect to this endpoint. Both local services must be running before the MCP tool can retrieve data.

## 5. Install the plugin for local development

Open a third terminal at the repository root and launch:

```powershell
claude --plugin-dir .
```

This loads the plugin from the checked-out directory for development and testing.

Inspect the plugin and verify the MCP connection using the available plugin/MCP commands in your Claude Code version. If a connection or permission approval is requested, review it and approve only if appropriate.

## 6. Install it through the GitHub marketplace

After this repository has been published publicly, start Claude Code and run:

```text
/plugin marketplace add sanjivsingh79/ciam-test-data-plugin
/plugin install ciam-test-data@ciam-marketplace
```

Replace `sanjivsingh79` with the actual GitHub owner.

Follow the installation prompts and choose the desired installation scope. If Claude Code asks you to reload plugins, run:

```text
/reload-plugins
```

The marketplace install loads the plugin configuration, agent, and Skill. The Python service and MCP server are separate local processes; start them using steps 3 and 4.

## 7. Verify the synthetic-data service

With the service running, use another PowerShell terminal:

```powershell
Invoke-RestMethod `
  "http://127.0.0.1:8090/test-customers?scenario=mfa_enrolment&status=ACTIVE&mfaEnabled=false"
```

The response should contain the matching synthetic identity, including:

- Customer ID: `TEST-10002`
- Scenario: `mfa_enrolment`
- Status: `ACTIVE`
- MFA enabled: `false`
- Email: `mfa-enrolment-01@example.test`

These are fictional test values. The service reads the repository's `test-data.json`; it does not create real application records.

## 8. Invoke the specialist agent

In Claude Code, ask for a test requiring the synthetic identity, for example:

```text
Use the test-data-agent to find an ACTIVE synthetic CIAM test identity
for scenario mfa_enrolment with MFA disabled.

Use the test-data-selection Skill and the find_test_customer MCP tool.
Return only the required synthetic identity fields and explain why
the identity matches the constraints.
```

The expected workflow is for the agent to call `find_test_customer`, validate the returned constraints, and report the result.

For test generation in a Java application, make a separate request to Claude Code that identifies the target class and desired behaviour. Review the proposed diff before executing tests.

## 9. Run the MCP client smoke test (optional)

The repository includes `mock-test-data/mcp-server/test_client.py`. Inspect it before running it, and check that it uses the current MCP endpoint and client API.

```powershell
Get-Content .\mock-test-data\mcp-server\test_client.py
```

If it is configured for the running server, execute it using the virtual environment:

```powershell
.\.venv\Scripts\python.exe .\mock-test-data\mcp-server\test_client.py
```

## Safety and limitations

- Use synthetic test identities only; never use production customer data.
- The service binds to localhost and is intended for local demonstration.
- The MCP tool returns data; it does not write to the application database.
- The `mfa_enrolment` scenario describes the synthetic identity, not implemented MFA enrolment functionality.
- Claude-generated tests must be reviewed before commit or execution.
- Do not place credentials, tokens, production data, or local environment files in this repository.
- This is a proof of concept, not a hardened production service.

## Troubleshooting

**MCP connection fails**
- Confirm the synthetic-data service is running on port 8090.
- Confirm the MCP server is running on port 9000.
- Check `.mcp.json` and verify that its URL matches the MCP server.
- Review the MCP server terminal for connection or Python errors.

**No matching identity is returned**
- Check the scenario spelling and the requested status/MFA constraints.
- Inspect `mock-test-data/test-data.json`.
- Do not invent an identity to compensate for missing test data.

**The Python package fails to install**
- Confirm that Python 3.10 or newer is being used.
- Run `python --version` and check that the virtual environment is active.
- Review the installation error before changing package versions.

## Cleanup

Stop both Python servers with `Ctrl+C`.

To uninstall the plugin, use the plugin management interface or the supported plugin uninstall command for your Claude Code version. Remove the marketplace registration only if you no longer need it.

## Related project

The Java application demonstrating MCP-assisted registration tests is maintained separately in the `claude-ciam-sdlc` repository.
