# EnvLock Prototype Project Structure

```
envlock/
├── README.md                    # Project overview
├── LICENSE                      # License file
├── pyproject.toml              # Modern Python project config
├── requirements.in              # Development dependencies (for pip-tools)
├── requirements.txt             # Pinned development dependencies
├── .envlock-version            # Tool version file
│
├── src/
│   └── envlock/
│       ├── __init__.py
│       ├── __main__.py          # CLI entry point
│       ├── cli.py               # CLI command definitions
│       ├── manager.py           # EnvLockManager class
│       ├── config.py            # Configuration loading/validation
│       ├── resolver.py          # PySMT dependency resolver
│       ├── binary_mapper.py     # Binary package mappings
│       ├── enforcer.py          # Channel and rule enforcement
│       ├── packager.py          # Packaging and offline utilities
│       ├── reproducibility.py   # Spec generation and reproduction
│       └── utils/
│           ├── __init__.py
│           ├── conda_utils.py   # Conda wrapper utilities
│           ├── pip_utils.py     # Pip wrapper utilities
│           ├── system_utils.py  # System checks and utilities
│           └── validation.py    # Input validation
│
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── test_manager.py
│   ├── test_resolver.py
│   ├── test_packager.py
│   ├── test_cli.py
│   └── fixtures/
│       ├── sample_config.yaml
│       └── sample_requirements.txt
│
├── examples/
│   ├── basic-usage/
│   │   ├── envlock.yaml
│   │   ├── requirements.txt
│   │   └── wheels/
│   │       └── README.md
│   ├── full-project/
│   │   ├── envlock.yaml
│   │   └── reproduce-script.sh
│   └── offline-packs/
│       └── README.md
│
├── templates/
│   ├── envlock.yaml.template    # Default configuration template
│   └── spec.meta.json.template  # Spec metadata template
│
├── schemas/
│   ├── config_schema.json       # JSON schema for envlock.yaml
│   └── spec_schema.json         # JSON schema for spec files
│
├── docs/
│   ├── index.md
│   ├── getting-started.md
│   ├── configuration.md
│   ├── commands.md
│   ├── binary-support.md
│   ├── offline-usage.md
│   └── advanced.md
│
├── scripts/
│   ├── install-miniconda.sh     # Miniconda bootstrap script
│   ├── bootstrap.py             # Initial setup script
│   ├── generate-schema.py       # Schema generation
│   └── release.py               # Release packaging script
│
└── .github/
    ├── workflows/
    │   ├── test.yml
    │   ├── release.yml
    │   └── publish.yml
    └── ISSUE_TEMPLATE/
        ├── bug_report.md
        └── feature_request.md
```

## Key File Details:

### **pyproject.toml**
```toml
[project]
name = "envlock"
version = "0.1.0"
description = "Layered environment manager with strict reproducibility"
readme = "README.md"
requires-python = ">=3.8"
dependencies = [
    "click>=8.0.0",
    "pyyaml>=6.0",
    "pysmt>=0.9.0",
    "requests>=2.28.0",
    "rich>=13.0.0",
    "checksumdir>=1.2.0",
    "filelock>=3.0.0",
]

[project.scripts]
envlock = "envlock.cli:main"

[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[tool.black]
line-length = 88
target-version = ['py38']

[tool.mypy]
python_version = "3.8"
warn_return_any = true
warn_unused_configs = true
```

### **src/envlock/__main__.py**
```python
#!/usr/bin/env python3
"""EnvLock CLI entry point."""

from envlock.cli import main

if __name__ == "__main__":
    main()
```

### **src/envlock/cli.py**
```python
"""EnvLock CLI interface."""

import click
from envlock.manager import EnvLockManager
from envlock.config import load_config


@click.group()
@click.version_option()
def main():
    """EnvLock: Reproducible environment manager."""
    pass


@main.command()
@click.argument("name")
@click.option("--python", default="3.11", help="Python version")
@click.option("--config", type=click.Path(exists=True), help="Config file")
def create(name, python, config):
    """Create a new environment."""
    manager = EnvLockManager()
    if config:
        cfg = load_config(config)
    else:
        cfg = {"environment": name, "python_version": python}
    
    manager.create_env(name, cfg)
    click.echo(f"Environment '{name}' created successfully.")


@main.command()
@click.argument("packages", nargs=-1)
@click.option("--env", help="Environment name")
def install(packages, env):
    """Install Python packages (pip-only)."""
    manager = EnvLockManager()
    manager.install_python(env, packages)


@main.command()
@click.argument("tool")
@click.option("--env", help="Environment name")
def install_binary(tool, env):
    """Install a binary tool."""
    manager = EnvLockManager()
    manager.install_binary(env, tool)


@main.command()
@click.option("--env", help="Environment name")
def lock(env):
    """Generate lock file."""
    manager = EnvLockManager()
    manager.generate_lock(env)


@main.command()
@click.option("--env", help="Environment name")
@click.option("--output", type=click.Path(), help="Output file")
def pack(env, output):
    """Pack environment for offline use."""
    manager = EnvLockManager()
    manager.pack_environment(env, output)


@main.command()
@click.argument("spec", type=click.Path(exists=True))
def reproduce(spec):
    """Reproduce environment from spec."""
    manager = EnvLockManager()
    manager.reproduce_from_spec(spec)


if __name__ == "__main__":
    main()
```

### **templates/envlock.yaml.template**
```yaml
# EnvLock Configuration Template
environment: "myenv"
python_version: "3.11"

# Binary tools to install (from conda-forge only)
binaries:
  required:
    # - java
    # - nodejs
    # - gcc
  optional:
    # - ollama
    # - rustc

# Python packages configuration
python:
  pip_only: true  # Enforced
  wheels_dir: "./wheels"  # Directory for user-provided wheels
  requirements_files:
    - "./requirements.txt"
  # Additional pip install options
  pip_options:
    - "--no-deps"  # Recommended for reproducible builds

# Lock settings
lock:
  freeze_file: "requirements.freeze.txt"
  pysmt_resolution: true
  generate_hash: true

# Network settings (optional)
network:
  proxy: null
  timeout: 30
  retries: 3

# Advanced settings (usually not needed)
advanced:
  miniconda_path: null  # Auto-detect
  conda_channels:
    - conda-forge  # Fixed for binaries only
  env_prefix: null  # Auto-generated
```

## Installation and Setup Scripts:

### **scripts/bootstrap.py**
```python
#!/usr/bin/env python3
"""Bootstrap EnvLock installation."""

import os
import subprocess
import sys
from pathlib import Path


def install_miniconda():
    """Install Miniconda if not present."""
    home = Path.home()
    miniconda_path = home / ".envlock" / "miniconda"
    
    if miniconda_path.exists():
        print("✓ Miniconda already installed")
        return str(miniconda_path)
    
    print("Installing Miniconda...")
    miniconda_path.parent.mkdir(parents=True, exist_ok=True)
    
    # Platform-specific download and install
    # ... implementation details
    
    return str(miniconda_path)


def setup_shell_integration():
    """Set up shell completion."""
    # Add envlock to PATH
    # Set up bash/zsh/fish completion
    pass


def main():
    print("🚀 Bootstrapping EnvLock...")
    
    # 1. Check Python version
    if sys.version_info < (3, 8):
        print("Error: Python 3.8+ required")
        sys.exit(1)
    
    # 2. Install Miniconda
    conda_path = install_miniconda()
    
    # 3. Install EnvLock in development mode
    subprocess.run([sys.executable, "-m", "pip", "install", "-e", "."])
    
    # 4. Set up shell integration
    setup_shell_integration()
    
    print("✅ EnvLock bootstrap complete!")
    print("\nNext steps:")
    print("  envlock create myenv --python=3.11")
    print("  envlock install-binary nodejs")
    print("  envlock install numpy pandas")


if __name__ == "__main__":
    main()
```

## Initial Development Setup:

```bash
# Clone the repository
git clone https://github.com/yourusername/envlock.git
cd envlock

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install development dependencies
pip install -r requirements.txt

# Install in development mode
pip install -e .

# Run tests
pytest tests/

# Run the CLI
envlock --help
```

This prototype structure provides:
1. **Clean separation** of concerns with modular components
2. **Full test suite** for reliability
3. **Comprehensive documentation** structure
4. **Example configurations** for users
5. **Development tooling** ready for CI/CD
6. **Extensible architecture** for future features

The implementation focuses on the core requirements while maintaining the strict enforcement rules and reproducibility guarantees specified in the design.
