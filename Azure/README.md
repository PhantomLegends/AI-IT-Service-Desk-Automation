# Azure Functions Creation

A simple Python Azure Function with an HTTP endpoint called `AskAI`. It accepts a name and returns a personalized greeting.

## Run locally

Requires Python, Azure Functions Core Tools, and Azurite if your local settings use development storage.

From the project folder, run in PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
func start
```

Open `http://localhost:7071/api/AskAI?name=World` to try it:

```text
Hello, World. This HTTP triggered function executed successfully.
```

You can also send a JSON request body containing `{"name": "World"}`.

## Project files

- `function_app.py` — HTTP function and greeting logic.
- `host.json` — Azure Functions host configuration.
- `requirements.txt` — Python dependencies.

Local configuration belongs in `local.settings.json`, which is excluded from Git along with the virtual environment and local storage files.
