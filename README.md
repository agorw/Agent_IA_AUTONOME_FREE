# Local Autonomous Agent

A local development agent powered by a language model running through **Ollama**. It can explore a workspace, read and modify text files, run selected project checks, and iterate based on tool results.

The main script is `agent_autonome.py`.

> Despite the terminal banner ("AGENT WEB LOCAL"), the provided script does not implement web browsing. Its actions are limited to the local workspace and the explicitly allowed checks.

## Features

- Browse files and directories in the workspace.
- Read UTF-8 text files.
- Create or replace text files.
- Make targeted text replacements when the original text occurs exactly once.
- Automatically back up existing files before replacing them.
- Run predefined local checks:
  - PHP syntax checks (`php -l`);
  - `composer.json` validation;
  - PHPUnit tests;
  - `npm test` and `npm build` scripts;
  - Git status and change summaries;
  - Python syntax compilation checks.
- Log actions and errors to `.agent_cache/agent.log`.
- Fall back to a JSON-based compatibility mode if native model tool calls fail.

## Requirements

- Python 3.10 or later is recommended.
- [Ollama](https://ollama.com/) installed and running.
- The configured model must be available in Ollama.
- The Python `ollama` package.

Depending on which checks you want to run, you may also need PHP, Composer, PHPUnit, Node.js/npm, or Git.

## Installation

1. Place `agent_autonome.py` in the project directory the agent should manage, or in a directory from which you will specify the workspace.
2. Install the Python dependency:

   ```bash
   python -m pip install ollama
   ```

3. Make sure Ollama is running and that the desired model is available. The default model is:

   ```text
   hf.co/Hob-forge/smeagle-4b:Q4_K_M
   ```

   If needed, pull it with Ollama:

   ```bash
   ollama pull hf.co/Hob-forge/smeagle-4b:Q4_K_M
   ```

   You can select another model that supports tool calling by setting `OLLAMA_MODEL`.

## Configuration

Configuration is handled through environment variables.

| Variable | Default | Description |
|---|---|---|
| `OLLAMA_MODEL` | `hf.co/Hob-forge/smeagle-4b:Q4_K_M` | Ollama model name. |
| `AGENT_WORKSPACE` | Current directory (`Path.cwd()`) | Root directory in which the agent is allowed to work. |

### Examples

Linux / macOS:

```bash
export OLLAMA_MODEL="hf.co/Hob-forge/smeagle-4b:Q4_K_M"
export AGENT_WORKSPACE="/path/to/my-project"
python agent_autonome.py
```

PowerShell:

```powershell
$env:OLLAMA_MODEL = "hf.co/Hob-forge/smeagle-4b:Q4_K_M"
$env:AGENT_WORKSPACE = "C:\path\to\my-project"
python .\agent_autonome.py
```

If `AGENT_WORKSPACE` is not set, the current working directory when the script starts is used.

## Running the Agent

From the directory containing the script, run:

```bash
python agent_autonome.py
```

The agent starts an interactive terminal session. Enter a request in natural language, for example:

```text
Analyze the project structure and explain how it is organized.
```

Other examples:

```text
Find PHP syntax errors in the project.
```

```text
Add server-side validation to the login form.
```

```text
Run the available tests and fix any errors you can reproduce.
```

To stop the agent, enter `quitter`, `exit`, or `quit`, or press `Ctrl+C`.

## Available Tools

| Tool | Purpose |
|---|---|
| `list_files` | Lists the contents of a directory, optionally recursively. |
| `read_text_file` | Reads a text file in the workspace. |
| `write_text_file` | Creates a file or completely replaces its contents. |
| `edit_text_file` | Replaces an exact occurrence that appears only once in a file. |
| `run_check` | Runs one of the predefined checks listed below. |
| `get_project_info` | Displays the workspace, model, Python version, detected executables, and the presence of selected project configuration files. |

### Allowed Checks

The accepted `run_check` values are:

| Check | Action |
|---|---|
| `php_lint_all` | Runs `php -l` on discovered PHP files (up to 150 files). |
| `composer_validate` | Runs `composer validate --no-check-publish` if `composer.json` exists. |
| `phpunit` | Runs `vendor/bin/phpunit` if PHPUnit is installed locally in the project. |
| `npm_test` | Runs `npm run test` if `package.json` and npm are available. |
| `npm_build` | Runs `npm run build` if `package.json` and npm are available. |
| `git_status` | Runs `git status --short`. |
| `git_diff_stat` | Runs `git diff --stat`. |
| `python_compile` | Checks the syntax of discovered Python files (up to 100 files). |

Commands are executed without a shell (`shell=False`), from the workspace, and with time limits. This is not a general-purpose terminal: arbitrary shell commands are not exposed to the agent.

## Security and Limitations

The script enforces several restrictions:

- Resolved paths must remain inside the workspace.
- Access to `.git`, `.venv`, `venv`, `node_modules`, `vendor`, `.idea`, and `.vscode` directories is blocked.
- Access to certain sensitive files is prohibited, including `.env` files, secret files, and private keys.
- The `.pem`, `.pfx`, `.p12`, `.kdbx`, and `.secret` file extensions are blocked.
- Binary files are not read as text.
- Reading is limited to 180,000 characters; writing is limited to 180,000 characters.
- Before an existing file is replaced, a backup is created in `.agent_cache/backups/`.
- Tool output is truncated to keep the conversation history manageable.

**Please note:**

- These safeguards reduce risk, but they do not replace project backups or code review.
- The model may produce incorrect changes. Review modifications before integrating or deploying them.
- Destructive operations and actions involving external systems require user approval under the system prompt instructions.
- The agent does not guarantee that tests exist or provide adequate coverage.
- The script does not configure a database or provide a dedicated SQL connection.
- The supplied version does not include a web-browsing tool.

## Files Created at Runtime

The script automatically creates the following directory in the workspace:

```text
.agent_cache/
├── agent.log
└── backups/
    └── ... backups of replaced files
```

The log contains timestamps, log levels, and the names of tools called. Backups are created before an existing file is replaced.

## Troubleshooting

### Ollama is not responding

Make sure Ollama is installed and running, then check the available models with:

```bash
ollama list
```

### Model not found

Check the value of `OLLAMA_MODEL` and make sure the corresponding model is available in Ollama.

### Invalid tool calls

The script automatically attempts to switch to a JSON-based compatibility mode. If the model still cannot produce valid tool calls, try a model with better tool-calling support.

### A check is unavailable

Make sure the required executable is installed and available in your `PATH`. Some checks also require project files, such as `composer.json`, `package.json`, or `vendor/bin/phpunit`.

---

## License

No license is specified in the provided source file. Add a `LICENSE` file if you intend to distribute the project under a specific license.
