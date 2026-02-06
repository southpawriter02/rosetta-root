# v0.1.1 — Environment Setup

> **Task:** Install Python, create virtual environment, and install all dependencies.
> 

---

## Task Overview

---

## Workflow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ 1. Install  │───▶│ 2. Create   │───▶│ 3. Install  │───▶│ 4. Verify   │
│   Python    │    │    venv     │    │    deps     │    │   setup     │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

---

## Step-by-Step Instructions

### Step 1: Verify Python Installation

```bash
# Check Python version (must be 3.11+)
python --version
# or
python3 --version
```

**If Python is not installed:**

- **macOS:** `brew install python@3.11`
- **Windows:** Download from [python.org](http://python.org)
- **Linux:** `sudo apt install python3.11`

### Step 2: Create Project Directory

```bash
# Create and navigate to project folder
mkdir docstratum
cd docstratum

# Initialize git
git init
```

### Step 3: Create Virtual Environment

```bash
# Create venv
python -m venv .venv

# Activate (macOS/Linux)
source .venv/bin/activate

# Activate (Windows)
.venv\Scripts\activate
```

### Step 4: Create requirements.txt

```
# requirements.txt
# Core
pydantic>=2.0.0
pyyaml>=6.0

# LLM Integration
langchain>=0.2.0
langchain-openai>=0.1.0
openai>=1.0.0

# Demo
streamlit>=1.30.0

# Graph (Optional)
neo4j>=5.0.0

# Testing
pytest>=8.0.0
pytest-cov>=4.0.0

# Development
python-dotenv>=1.0.0
```

### Step 5: Install Dependencies

```bash
pip install -r requirements.txt
```

### Step 6: Create .env.example

```bash
# .env.example
OPENAI_API_KEY=sk-your-key-here
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your-password
```

### Step 7: Create .gitignore

```
# .gitignore
.venv/
__pycache__/
*.pyc
.env
.DS_Store
*.egg-info/
dist/
build/
.pytest_cache/
```

---

## Verification Script

```python
# verify_setup.py
import sys

def check_python_version():
    version = sys.version_info
    assert version.major == 3 and version.minor >= 11, f"Python 3.11+ required, got {version.major}.{version.minor}"
    print(f"✅ Python {version.major}.{version.minor}.{version.micro}")

def check_imports():
    imports = [
        'pydantic',
        'yaml',
        'langchain',
        'streamlit',
        'pytest'
    ]
    for module in imports:
        __import__(module)
        print(f"✅ {module}")

if __name__ == '__main__':
    check_python_version()
    check_imports()
    print("\n🎉 Environment setup complete!")
```

Run: `python verify_[setup.py](http://setup.py)`

---

## Acceptance Criteria

- [ ]  `python --version` returns 3.11+
- [ ]  `.venv` directory exists
- [ ]  `pip list` shows all required packages
- [ ]  `python verify_[setup.py](http://setup.py)` passes all checks
- [ ]  `.gitignore` excludes `.venv` and `.env`

---

## Troubleshooting