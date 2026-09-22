# အဖြေများ — Python & uv Project Workflow

## လေ့ကျင့်ခန်း ၁ — uv project အသစ် တည်ဆောက်ခြင်း

```python
# init_project.py - not a script; these are shell commands to run manually
# Step 1: create a new uv project
#   $ uv init my-first-ai-app
# Step 2: change into the project directory
#   $ cd my-first-ai-app
# Step 3: list the generated files
#   $ ls -la
# Step 4: show the generated pyproject.toml
#   $ cat pyproject.toml
#
# Typical pyproject.toml content after uv init:
#   [project]
#   name = "my-first-ai-app"
#   version = "0.1.0"
#   requires-python = ">=3.12"
#   dependencies = []
print("Follow the shell commands in the comments above.")
```

**အဓိကအယူအဆ** — `uv init` က `pyproject.toml` နှင့် အခြေခံ ဖိုင်များကို အလိုအလျောက် ဖန်တီးပေးသဖြင့် project စတင်ရန် အဆင်သင့် ဖြစ်စေသည်။

## လေ့ကျင့်ခန်း ၂ — Dependency ထည့်သွင်းခြင်း

```python
# add_dep.py - shell commands plus a small verification
# Step 1: add the httpx dependency
#   $ uv add httpx
# Step 2: check the changes in pyproject.toml
#   $ cat pyproject.toml
# Step 3: check the lockfile for the exact pinned version
#   $ grep httpx uv.lock
#
# Verify the dependency was recorded:
import tomllib

with open("pyproject.toml", "rb") as f:
    data = tomllib.load(f)

deps = data["project"]["dependencies"]
print("Recorded dependencies:", deps)
assert any("httpx" in d for d in deps), "httpx not found in dependencies"

# Expected output:
# Recorded dependencies: ['httpx>=0.27.0']
```

**အဓိကအယူအဆ** — `uv add` က `pyproject.toml` ထဲ requirement ကို ပေါင်းပြီး၊ `uv.lock` ထဲ version အတိအကျကို မှတ်တမ်းတင်ပေးသည်။

## လေ့ကျင့်ခန်း ၃ — uv run ဖြင့် script run ခြင်း

```python
# show_httpx_version.py - print the installed httpx version
import importlib.metadata

# fetch the installed version of httpx
version = importlib.metadata.version("httpx")
print("httpx version:", version)

# Expected output (run with: uv run show_httpx_version.py):
# httpx version: 0.27.2
```

Run command:

```bash
# no activation needed - uv run handles the environment automatically
uv run show_httpx_version.py
```

**အဓိကအယူအဆ** — `uv run` က environment ကို အလိုအလျောက် ပြင်ဆင်ပေးသောကြောင့် activate လုပ်ရန် မလိုအပ်တော့ပါ။

## လေ့ကျင့်ခန်း ၄ — pyproject.toml ဖတ်ခြင်း (Python)

```python
import tomllib
from pathlib import Path

# Create a sample pyproject.toml file for demonstration
sample_toml = """[project]
name = "my-awesome-app"
version = "1.2.0"
dependencies = [
    "requests>=2.31.0",
    "numpy>=1.26.0",
    "rich>=13.0.0",
]
"""

file_path = Path("pyproject.toml")
file_path.write_text(sample_toml, encoding="utf-8")

# Open the TOML file in binary mode ("rb") as required by tomllib
with open(file_path, "rb") as f:
    data = tomllib.load(f)

# Extract project metadata from the parsed dictionary
project = data.get("project", {})
name = project.get("name", "unknown")
version = project.get("version", "unknown")
dependencies = project.get("dependencies", [])

# Print the results clearly
print(f"Project Name: {name}")
print(f"Version: {version}")
print("Dependencies:")
for dep in dependencies:
    print(f"  - {dep}")

# Clean up the sample file
file_path.unlink()
```

**အဓိကအယူအဆ** — `tomllib.load()` သည် binary mode (`"rb"`) ဖြင့် ဖွင့်ထားသော file object ကို လက်ခံပြီး Python dictionary အဖြစ် ပြောင်းပေးသောကြောင့် project name၊ version နှင့် dependencies တို့ကို key များဖြင့် လွယ်ကူစွာ ရယူနိုင်သည်။

## လေ့ကျင့်ခန်း ၅ — Installed version စစ်ဆေးခြင်း

```python
from importlib.metadata import version, PackageNotFoundError

try:
    # Try to read the installed version of the httpx package
    httpx_version = version("httpx")
    print(f"httpx installed version: {httpx_version}")
except PackageNotFoundError:
    # Raise a clear error if the package is not installed in the environment
    raise RuntimeError(
        "httpx package is not installed in this environment. "
        "Run the script with 'uv run' so the venv includes httpx."
    )
```

**အဓိကအယူအဆ** — `importlib.metadata.version()` ဖြင့် environment အတွင်း installed ဖြစ်နေသော package version ကို စစ်ဆေးနိုင်ပြီး `PackageNotFoundError` ကို catch လုပ်ခြင်းဖြင့် ရှင်းလင်းသော error message ထုတ်ပေးနိုင်သည်။

## လေ့ကျင့်ခန်း ၆ — Environment တူညီမှု စစ်ဆေးခြင်း

```python
import sys

# sys.executable contains the full path of the current Python interpreter.
# We check whether that path contains a ".venv" directory segment, which
# indicates the script is running inside the project's virtual environment.
path = sys.executable

# Normalize separators so the check works on Windows too (where both
# "/" and "\\" may appear in paths).
normalized = path.replace("\\", "/")

# A robust check: look for a path segment that is exactly ".venv"
in_venv = any(part == ".venv" or "venv" in part for part in normalized.split("/"))

if in_venv:
    print("OK — ဤ script သည် project ၏ .venv environment အတွင်းမှ Python interpreter ဖြင့် run နေပါသည်။")
    print(f"Interpreter: {sys.executable}")
else:
    print("WARNING: လက်ရှိ Python interpreter သည် project ၏ .venv အတွင်းတွင် မရှိပါ။")
    print(f"Interpreter: {sys.executable}")
    print("Environment တူညီမှုရှိစေရန် အောက်ပါ command ဖြင့် ပြန်လည် run ပါ:")
    print("    uv run check_env.py")
```

**အဓိကအယူအဆ** — `sys.executable` ထဲတွင် `venv` path ပါဝင်မှုကို စစ်ဆေးခြင်းအားဖြင့် script တစ်ခုသည် project ၏ `.venv` environment အတွင်းမှ run နေခြင်းရှိ မရှိ အသိအမှတ်ပြုနိုင်ပြီး၊ မဟုတ်ပါက `uv run` ဖြင့် ပြန်လည် run ရန် အသုံးပြုသူအား ညွှန်ကြားနိုင်သည်။
