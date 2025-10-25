# Python Project Setup Guide

This guide provides general setup instructions for Python projects in this repository.

## Prerequisites

- Python 3.8 or higher
- pip (Python package manager)
- git
- A text editor or IDE (VS Code, PyCharm, etc.)

## Initial Setup

### 1. Virtual Environment

Always use a virtual environment for each project to isolate dependencies:

```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Linux/Mac:
source venv/bin/activate
# On Windows:
venv\Scripts\activate
```

### 2. Install Dependencies

```bash
# Install from requirements.txt
pip install -r requirements.txt

# Or install packages individually
pip install package_name
```

### 3. Environment Variables

1. Copy `.env.example` to `.env`:
   ```bash
   cp .env.example .env
   ```

2. Edit `.env` with your actual values

3. Never commit `.env` to version control (already in .gitignore)

## Development Workflow

### Creating a New Project

1. Choose the appropriate directory:
   - `telegram_bots/` for Telegram bots
   - `django_websites/` for Django websites

2. Create project directory:
   ```bash
   cd telegram_bots  # or django_websites
   mkdir my_project
   cd my_project
   ```

3. Set up virtual environment and install dependencies

4. Create project structure following the guidelines in respective README files

### Managing Dependencies

```bash
# Install a new package
pip install package_name

# Update requirements.txt
pip freeze > requirements.txt

# Install all dependencies from requirements.txt
pip install -r requirements.txt
```

## Code Quality

### Formatting

Consider using code formatters:
```bash
pip install black isort
black .
isort .
```

### Linting

Consider using linters:
```bash
pip install flake8 pylint
flake8 .
pylint your_module/
```

### Type Checking

Consider using type checkers:
```bash
pip install mypy
mypy .
```

## Version Control

### Git Workflow

1. Create a feature branch:
   ```bash
   git checkout -b feature/my-feature
   ```

2. Make your changes and commit:
   ```bash
   git add .
   git commit -m "Description of changes"
   ```

3. Push to remote:
   ```bash
   git push origin feature/my-feature
   ```

### Commit Messages

Write clear commit messages:
- Use present tense ("Add feature" not "Added feature")
- Be concise but descriptive
- Reference issue numbers when applicable

## Testing

### Running Tests

```bash
# For Django projects
python manage.py test

# For other Python projects
pytest
# or
python -m unittest discover
```

### Writing Tests

- Write tests for new features
- Maintain test coverage
- Use descriptive test names

## Troubleshooting

### Common Issues

1. **ModuleNotFoundError**: Make sure virtual environment is activated and dependencies are installed
2. **Permission Errors**: Check file permissions, may need to use `sudo` on Linux/Mac
3. **Port Already in Use**: Kill the process using the port or use a different port

### Getting Help

- Check project-specific README files
- Review Django/Telegram Bot documentation
- Search for error messages online
- Ask for help in team channels

## Resources

- [Python Virtual Environments Guide](https://docs.python.org/3/tutorial/venv.html)
- [pip Documentation](https://pip.pypa.io/)
- [Git Documentation](https://git-scm.com/doc)
- [Python Best Practices](https://docs.python-guide.org/)
