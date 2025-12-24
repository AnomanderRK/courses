- Course: https://luxoft.udemy.com/course/llm-engineering-master-ai-and-large-language-models 
- resources: https://edwarddonner.com/2024/11/13/llm-engineering-resources/
- Repo: https://github.com/ed-donner/llm_engineering

· Setup
1. Clone repo https://github.com/ed-donner/llm_engineering
2. Install uv (Mac):
   ```bash
   brew install uv
   ```
   Or using the official installer:
   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh
   ```
   Restart terminal after installation, then verify with `uv --version`
3. Create an OpenAI key at https://platform.openai.com/api-keys
4. Create .env file

· uv Usage Examples

**Install a specific Python version:**
```bash
uv python install 3.11        # Install Python 3.11
uv python install 3.12        # Install Python 3.12
uv python list                # List all available/installed Python versions
```

**Create a virtual environment:**
```bash
uv venv                       # Creates .venv with default Python
uv venv --python 3.11         # Creates .venv with Python 3.11
uv venv myenv --python 3.12   # Creates 'myenv' folder with Python 3.12
```

**Activate virtual environment (Mac):**
```bash
source .venv/bin/activate     # Activate the virtual environment
deactivate                    # Deactivate when done
```

**Install packages:**
```bash
uv pip install requests                    # Install a single package
uv pip install requests pandas numpy       # Install multiple packages
uv pip install -r requirements.txt         # Install from requirements file
uv pip install -e .                        # Install current project in editable mode
```

**Project management (with pyproject.toml):**
```bash
uv init                       # Initialize a new project
uv add requests               # Add a dependency
uv add --dev pytest           # Add a dev dependency
uv sync                       # Sync all dependencies from pyproject.toml
uv run python script.py       # Run a script in the project environment
```