
# Local Setup Guide

## AI IT Service Desk Automation

This guide explains how to run the AI-powered IT Service Desk automation locally on Windows 11.

The project uses Azure Logic Apps Standard, Azure Functions (Python), LM Studio with Qwen3 8B, and Azurite storage.

No Azure subscription or cloud AI API key is required.

## 1. Prerequisites

Install the following:

- Python
- Visual Studio Code
- Azure Functions Core Tools v4
- Azure Functions VS Code extension
- Azure Logic Apps (Standard) VS Code extension and its local development prerequisites
- Azurite
- LM Studio
- Git

A computer capable of running a local language model is recommended. The project was developed using an NVIDIA RTX 3060 with 12 GB VRAM and 64 GB system RAM.

## 2. Clone the Repository

Open PowerShell:

```powershell
git clone https://github.com/PhantomLegends/AI-IT-Service-Desk-Automation.git
cd AI-IT-Service-Desk-Automation
```

## 3. Configure LM Studio

1. Open LM Studio.
2. Download and load a compatible Qwen3 8B model.
3. Start the local API server.
4. Confirm the server is listening at:

   http://localhost:1234

The project was tested using the `qwen/qwen3-8b` model.

Ensure the model identifier and local API configuration match those expected by `Azure/function_app.py`.

## 4. Start Azurite

Start the Azurite Blob, Queue, and Table services using VS Code or your preferred local Azurite setup.

Default ports:

| Service | Port |
|---|---|
| Blob Storage | 10000 |
| Queue Storage | 10001 |
| Table Storage | 10002 |

The application stores ticket records in the `ITServiceTickets` table.

Azurite data files are excluded from Git because they are generated locally.

## 5. Configure Azure Functions

Open PowerShell in the repository's `Azure` directory:

```powershell
cd Azure
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Create a file named `local.settings.json` inside the `Azure` directory.

Example local development configuration:

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "python"
  }
}
```

Check `function_app.py` for any additional environment variables or model configuration required by your version of the project.

**Security:** Never commit `local.settings.json` to GitHub.

Start the Azure Functions host:

```powershell
$env:languageWorkers__python__defaultExecutablePath = (Resolve-Path ".\.venv\Scripts\python.exe").Path
$env:PYTHONPATH = (Resolve-Path ".\.venv\Lib\site-packages").Path
func host start --port 7071
```

Expected HTTP endpoints:

- `POST http://localhost:7071/api/AskAI`
- `POST http://localhost:7071/api/ReviewTicket`

Keep this terminal running.

## 6. Configure Logic Apps Standard

Open a separate PowerShell terminal and navigate to:

```powershell
cd LogicApps\ITAutomationWorkspace\ITAutomationApp
```

Create the local `local.settings.json` required by the Logic Apps Standard runtime and its local storage configuration.

Use the Azure Logic Apps (Standard) VS Code extension to initialize local settings if necessary.

Example `local.settings.json` configuration:

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_INPROC_NET8_ENABLED": "1",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet",
    "APP_KIND": "workflowapp",
    "ProjectDirectoryPath": "REPLACE_WITH_ABSOLUTE_PATH_TO_ITAutomationApp",
    "languageWorkers__node__defaultExecutablePath": "REPLACE_WITH_ABSOLUTE_PATH_TO_NODE_EXE"
  }
}
```

Replace the two placeholder paths with the paths on your own computer.

Check that the workflow's HTTP action targets:

```text
http://localhost:7071/api/AskAI
```

Start the Logic Apps host:

```powershell
func host start --port 7072
```

Keep this terminal running.

## 7. Test AI Ticket Creation

Open another PowerShell terminal and send a test request:

```powershell
Invoke-RestMethod -Uri "http://localhost:7071/api/AskAI" -Method POST -ContentType "application/json" -Body '{"employee":"Demo Employee","request_type":"I need VPN access to work remotely"}' | ConvertTo-Json -Depth 10
```

A successful response should contain fields such as:

- `ticket_id`
- `created_at`
- `employee`
- `request_type`
- `category`
- `priority`
- `recommended_action`
- `status`
- `processed_by`

Requests requiring manager approval receive the status `Pending Approval`.

## 8. Test Manager Approval

Use the `ticket_id` returned by a pending-approval ticket.

Replace `YOUR_TICKET_ID` in this command:

```powershell
Invoke-RestMethod -Uri "http://localhost:7071/api/ReviewTicket" -Method POST -ContentType "application/json" -Body '{"ticket_id":"YOUR_TICKET_ID","decision":"Approved"}' | ConvertTo-Json -Depth 10
```

The response should confirm that the ticket status has changed to `Approved`.

You can also submit `Rejected` for a different pending ticket.

The application rejects duplicate reviews and handles nonexistent ticket IDs.

## 9. Test the Logic Apps Workflow

With both local hosts running, obtain the HTTP trigger callback URL for the `ITServiceRequest` workflow:

```powershell
$callback = Invoke-RestMethod -Uri "http://localhost:7072/runtime/webhooks/workflow/api/management/workflows/ITServiceRequest/triggers/When_an_HTTP_request_is_received/listCallbackUrl?api-version=2019-10-01" -Method POST
```

Submit a test request:

```powershell
Invoke-RestMethod -Uri $callback.value -Method POST -ContentType "application/json" -Body '{"employee":"Demo Employee","request_type":"I need VPN access to work remotely"}'
```

The Logic Apps workflow should call the `AskAI` function and return the ticket information.

**Important:** The callback URL may contain a signature. Treat it as sensitive and do not publish it.

## 10. Troubleshooting

### Azure Functions cannot import Python packages

Verify that dependencies are installed inside `Azure/.venv` and that the Functions host uses that environment.

### AI model is unavailable

Check that LM Studio is running, the model is loaded, and the local API server is listening on port `1234`.

### Ticket storage fails

Check that Azurite Blob, Queue, and Table services are running and that the local storage configuration is correct.

### Logic Apps cannot reach Azure Functions

Check that Azure Functions is running on port `7071` and the workflow points to the correct local endpoint.

## 11. Security and Production Limitations

This is a local portfolio prototype.

The `ReviewTicket` endpoint does not currently authenticate managers or enforce role-based permissions.

Production use would require authentication, authorization, secrets management, monitoring, and additional security controls.
