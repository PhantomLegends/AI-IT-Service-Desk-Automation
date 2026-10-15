# AI IT Service Desk Automation

### Local AI-Powered IT Ticket Classification and Approval Workflow

An end-to-end IT service desk automation project built using **Azure Logic Apps Standard**, **Azure Functions (Python)**, **LM Studio with Qwen3 8B**, and **Azurite Table Storage**.

The solution accepts employee IT requests, uses a locally hosted AI model to classify and prioritize them, generates unique ticket IDs, stores ticket records, and supports manager approval or rejection workflows.

The entire solution runs locally on Windows 11, without requiring an Azure subscription or paid cloud AI API.

## Architecture

```text
Employee IT Request (HTTP POST)
            |
            v
Azure Logic Apps Standard
            |
            v
Azure Functions - AskAI
            |
            v
LM Studio - Qwen3 8B
            |
            v
AI Classification and Priority
            |
            v
Azurite Table Storage
            |
            v
JSON Response to Employee

Manager Review (HTTP POST)
            |
            v
Azure Functions - ReviewTicket
            |
            v
Azurite Ticket Status Update
```

## Key Features

- AI-powered IT request classification using a locally hosted Qwen3 8B model.
- Automatic priority assignment: Low, Medium, High, or Critical.
- Recommended IT support actions based on the request.
- Unique ticket ID generation and UTC timestamps.
- Persistent ticket storage using Azurite Table Storage.
- Manager approval and rejection for requests requiring authorization.
- Duplicate review protection and missing-ticket error handling.
- Rule-based fallback when AI output is unavailable or invalid.
- Local development and testing without cloud deployment.

## Technology Stack

| Component | Technology |
|---|---|
| Workflow orchestration | Azure Logic Apps Standard |
| Backend API | Azure Functions, Python |
| AI inference | LM Studio, Qwen3 8B |
| Ticket database | Azurite Table Storage |
| Development environment | Windows 11, VS Code, PowerShell |
| Version control | Git and GitHub |

## Project Structure

```text
AI-IT-Service-Desk-Automation/
├── Azure/
│   ├── function_app.py
│   ├── requirements.txt
│   ├── host.json
│   └── README.md
├── LogicApps/
│   └── ITAutomationWorkspace/
│       └── ITAutomationApp/
│           ├── host.json
│           ├── connections.json
│           └── ITServiceRequest/
│               └── workflow.json
├── .gitignore
└── README.md
```

## API Endpoints

### AskAI

`POST http://localhost:7071/api/AskAI`

Example request:

```json
{
  "employee": "Example Employee",
  "request_type": "I need VPN access to work remotely"
}
```

The function classifies the request, determines priority and recommended action, generates a ticket ID, and saves the ticket to local storage.

### ReviewTicket

`POST http://localhost:7071/api/ReviewTicket`

Example request:

```json
{
  "ticket_id": "IT-YYYYMMDD-XXXXXXXX",
  "decision": "Approved"
}
```

Supported decisions are `Approved` and `Rejected`. Reviews update eligible pending tickets in Azurite Table Storage.

## Running Locally

### Prerequisites

- Python and pip
- Azure Functions Core Tools
- Azure Logic Apps Standard local development tools
- Azurite
- LM Studio with a compatible local model
- Visual Studio Code

### 1. Start LM Studio

Load the Qwen3 8B model and start its local inference server at:

`http://localhost:1234`

### 2. Start Azurite

Ensure Blob, Queue, and Table services are running on their default local ports:

- Blob: `10000`
- Queue: `10001`
- Table: `10002`

### 3. Start Azure Functions

From the `Azure` directory, create a Python virtual environment, install dependencies, and configure the local settings required by the Functions host.

Start the Functions host on port `7071`.

### 4. Start Logic Apps

From `LogicApps/ITAutomationWorkspace/ITAutomationApp`, start the local Logic Apps host on port `7072`.

### 5. Test the Workflow

Send an HTTP POST request to the Logic Apps `ITServiceRequest` trigger callback URL with an employee name and IT request.

The workflow invokes the AI classification function and returns the ticket details.

**Note:** Local configuration files such as `local.settings.json` are intentionally excluded from Git. They must be created for each development environment.

## Tested Scenarios

- End-to-end Logic Apps to Azure Functions to local AI execution.
- Ticket creation and persistence in Azurite.
- Requests requiring manager approval.
- Successful ticket approval and rejection.
- Duplicate review attempts returning HTTP `409`.
- Nonexistent ticket lookups returning HTTP `404`.

## Security and Limitations

This project is a **local development and portfolio prototype**, not a production-ready IT service management platform.

The manager review endpoint does not currently enforce manager authentication or authorization. Production deployment would require identity verification, role-based access control, secure secrets management, monitoring, and additional operational safeguards.

## Future Improvements

- Microsoft Entra ID authentication and role-based manager approval.
- Email or Microsoft Teams approval notifications.
- IT service management dashboard.
- Automated testing and CI/CD pipelines.
- Optional Azure cloud deployment.

## Project Repository

[GitHub — AI IT Service Desk Automation](https://github.com/PhantomLegends/AI-IT-Service-Desk-Automation)
