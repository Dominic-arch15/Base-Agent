# Base Agent — Installation Guide

A lightweight, local-first AI agent with a browser-based chat interface, local memory, tools, optional skills, scheduled tasks, and optional Ollama model support.

## Contents

- [Requirements](#requirements)
- [1. Download and extract](#1-download-and-extract)
- [2. Install and run with Python](#2-install-and-run-with-python)
- [3. Enable a local AI model with Ollama (optional)](#3-enable-a-local-ai-model-with-ollama-optional)
- [4. Optional features](#4-optional-features)
- [5. Configuration](#5-configuration)
- [6. Run with Docker (optional)](#6-run-with-docker-optional)
- [Troubleshooting](#troubleshooting)
- [Security notes](#security-notes)

## Requirements

**Basic mode**
- Python 3.10 or newer recommended
- A terminal / command prompt
- A modern browser

The core agent uses Python's standard library, so no third-party Python packages are required for basic operation.

**Optional**
- [Ollama](https://ollama.com/) for responses from a local language model
- `ddgs` for web search
- Additional packages for standalone desktop voice control

## 1. Download and extract

1. Download `base-agent.zip` from the project's GitHub repository.
2. Extract the ZIP archive.
3. Open a terminal in the extracted `base-agent` directory — the directory containing `server.py`, `config.json`, and `requirements.txt`.

For example, on Linux:

```bash
unzip base-agent.zip
cd base-agent
```

If you downloaded the source using Git instead, run:

```bash
git clone <YOUR-REPOSITORY-URL>
cd base-agent
```

Replace `<YOUR-REPOSITORY-URL>` with the actual repository URL.

## 2. Install and run with Python

### Linux / macOS

Check your Python version:

```bash
python3 --version
```

Create and activate a virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Start the agent:

```bash
python3 server.py
```

If you want to enable optional packages, install them inside the activated environment:

```bash
python -m pip install -r requirements.txt
```

### Windows

Check your Python version:

```powershell
py --version
```

Create and activate a virtual environment:

```powershell
py -m venv .venv
.venv\Scripts\Activate.ps1
```

Start the agent:

```powershell
py server.py
```

To install optional packages:

```powershell
py -m pip install -r requirements.txt
```

If PowerShell blocks activation, you can run the environment's Python directly instead:

```powershell
.venv\Scripts\python.exe server.py
```

### Open the web interface

When the terminal reports that the server is running, open:

**http://localhost:8000**

Keep the terminal open while using the agent. Press `Ctrl+C` in that terminal to stop the server.

> **Note:** `requirements.txt` contains optional integrations. The basic web agent can run without installing it.

## 3. Enable a local AI model with Ollama (optional)

Without a model, the agent uses its built-in rule-based mode. That mode can route certain requests to tools, but it is not a full conversational reasoning model.

To use a local language model:

1. Install Ollama from [ollama.com](https://ollama.com/).
2. Start Ollama if it is not already running.
3. Download a model. For example:

   ```bash
   ollama pull llama3.1
   ```

4. Confirm the model is available:

   ```bash
   ollama list
   ```

5. In `config.json`, leave `"brain": "auto"` to use Ollama when available, or set it to `"ollama"` to request Ollama mode.
6. Start (or restart) the agent:

   ```bash
   python server.py
   ```

The default Ollama address is `http://localhost:11434`, and the configured model is `llama3.1`. If you download a different model, update `ollama_model` in `config.json` to match its name.

The first model download requires an internet connection. After it has been downloaded, local inference can run without sending prompts to a hosted model service. Hardware needs depend on the model you choose.

## 4. Optional features

### Web search

Install the optional dependencies:

```bash
python -m pip install -r requirements.txt
```

The `ddgs` package enables the web-search tool. Web search requires an internet connection.

### Standalone desktop voice assistant

The separate desktop voice loop requires additional packages:

```bash
python -m pip install pyttsx3 SpeechRecognition pyaudio
python -m agent.voice
```

Microphone support can require operating-system audio libraries. PyAudio installation can be especially difficult on some Windows setups. The browser-based voice controls, where supported, use browser speech features and do not require this standalone voice loop.

### Skills and knowledge

- Put a skill folder containing a `SKILL.md` file in `skills/`. A skill may also include a `run.py` file.
- Put supported `.md` or `.txt` reference documents in `knowledge/`.
- Restart the server after changing skills or adding files so the agent reloads them.

Only install skills and code you trust. Skill code may execute as part of the agent.

## 5. Configuration

Edit `config.json` in the project root. The default configuration is:

```json
{
  "brain": "auto",
  "ollama_url": "http://localhost:11434",
  "ollama_model": "llama3.1",
  "history_turns": 12,
  "network_tools_enabled": true,
  "automation_enabled": true
}
```

| Setting | Purpose |
|---|---|
| `brain` | `auto` selects Ollama when available and otherwise uses rule-based mode; `ollama` selects Ollama mode; `rule` forces rule-based mode. |
| `ollama_url` | Address of the Ollama service. |
| `ollama_model` | Model name already downloaded in Ollama. |
| `history_turns` | Number of previous conversation turns used for context. |
| `network_tools_enabled` | Enables or disables the agent's network-diagnostic tool group. |
| `automation_enabled` | Enables or disables scheduled tasks. |

Restart the server after editing the configuration.

## 6. Run with Docker (optional)

Install Docker, then run these commands from the project root:

```bash
docker build -t base-agent .
docker run --rm -p 8000:8000 base-agent
```

Open **http://localhost:8000** in your browser.

The container's filesystem is separate from your host. If you want local memory and scheduled-job data to persist across container recreation, mount a host directory to `/app/data`, for example:

```bash
docker run --rm -p 8000:8000 -v "$(pwd)/data:/app/data" base-agent
```

On Windows, adjust the volume path to match your shell and filesystem.

**Ollama in Docker:** `localhost` inside the container refers to the container itself, not your host. To use Ollama running on the host, configure `ollama_url` to an address reachable from the container and configure the host/network securely.

## Troubleshooting

### `python` command not found
Try `python3` on Linux/macOS or `py` on Windows. Install Python from [python.org](https://www.python.org/downloads/) if needed.

### Port 8000 is already in use
Choose another port when starting the server.

Linux / macOS:

```bash
PORT=8001 python server.py
```

Windows PowerShell:

```powershell
$env:PORT = "8001"
py server.py
```

Then visit **http://localhost:8001**.

### Ollama is not detected
- Make sure Ollama is installed and running.
- Run `ollama list` and verify that the configured model is downloaded.
- Check `ollama_url` and `ollama_model` in `config.json`.
- Try `ollama run llama3.1` to test the model independently.

### Web search does not work
Install `ddgs` in the same Python environment used to run the agent, and check your internet connection.

### Reset local memory
Stop the agent and remove `data/memory.db` if you intentionally want to clear its stored conversation memory. Back up the file first if you need to keep it.

## Security notes

- This project is designed primarily for local use. The server has **no built-in login or rate limiting**.
- The server defaults to binding to `0.0.0.0`, which can make it reachable by other devices on your network depending on firewall and network settings. For strictly local-only access, start it with a loopback host:

  Linux / macOS:
  ```bash
  HOST=127.0.0.1 python server.py
  ```

  Windows PowerShell:
  ```powershell
  $env:HOST = "127.0.0.1"
  py server.py
  ```

- Do not expose this server directly to the public internet without adding authentication, access controls, rate limiting, HTTPS, and appropriate isolation.
- The `run_code` tool executes code with the operating-system permissions of the server process. Run the agent only in an environment where you trust the code and its users.
- Review third-party skills before installing them. This project does not provide its own comprehensive moderation or sandboxing layer.

## Project structure

```text
base-agent/
├── agent/          # Agent logic, tools, memory, and automation
├── data/           # Local runtime data
├── knowledge/      # Documents used as reference knowledge
├── skills/         # Optional skill extensions
├── web/            # Browser-based user interface
├── config.json     # Agent configuration
├── requirements.txt
├── server.py       # Web server entry point
└── Dockerfile
```

## License

Add your project's license here before publishing if you intend others to reuse or distribute the code.
