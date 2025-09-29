# Development Guide

## Local Setup

### Prerequisites

- Python 3.13+
- PostgreSQL 17+
- Redis 7.0+

### Installation

1. Clone the repository:

    ```bash
    git clone https://github.com/yourusername/project-nexus.git
    cd project-nexus
    ```

2. Set up Python environment:

    ```bash
    python -m venv .venv
    source .venv/bin/activate  # On Windows: .venv\Scripts\activate
    pip install -r requirements/development.txt
    ```

3. Set up environment variables:

    ```bash
    cp .env.example .env

    # Edit .env with your configuration
    ```

4. Set up pre-commit hooks:

    ```bash
    pre-commit install # Installs the git hook scripts

    # I was able to set it up very little time to learn it.
    ```

## Development Workflow

### Branching Strategy

I did not use Gitflow or Github Flow. I used a simple branching strategy. I planned to use it after the ALX programme.

- `main`: Production-ready code
- `develop`: Integration branch
- `feature/*`: New features
- `bugfix/*`: Bug fixes
- `hotfix/*`: Critical production fixes

### Code Style

- Follow PEP 8
- Use Black for code formatting
- Use isort for import sorting
- Maximum line length: 88 characters

### Testing

I set up pytest for testing but did not write tests due to time constraints.

Run tests:

```bash
pytest
```

Run with coverage:

```bash
pytest --cov=.
```

### Code Review Process

1. Create a feature branch from develop
2. Write tests for new features
3. Submit a Pull Request (PR)
4. Address review comments
5. Get at least one approval before merging

## Debugging

### Django Debug Toolbar

The Django Debug Toolbar is enabled in development. Access it at the top of any page.

### Logging

Logs are written to `logs/` directory. For development, logs are also printed to console.

### Documentation

Build documentation:

```bash
cd docs
make html
```

View documentation:

```bash
open _build/html/index.html
```
