# Agent Automation and Development Guidelines

## Overview

This document provides instructions for AI agents on when and how to execute
FluidAttacks security scanners. FluidAttacks provides free, open-source SAST
(Static Application Security Testing) and SCA (Software Composition Analysis)
scanners that can be executed locally using Docker.

## When to Execute Scanners

### Execute SCA Scanner When:
- New dependencies are added to the project
- Dependencies are updated to new versions
- Lock files are modified (e.g.,`package-lock.json`, `uv.lock`)
- User explicitly requests a dependency security scan
- Setting up a new project for the first time
- Before deploying to production

### Execute SAST Scanner When:
- Source code changes are made to application files
- New features or modules are added
- Security-sensitive code is modified (authentication, authorization)
- User explicitly requests a code security scan
- Before committing significant code changes
- During code reviews
- Before deploying to production

### Execute Both Scanners When:
- Complete security audit is needed
- Major project updates involving both code and dependencies
- Pre-deployment security check
- User requests a full security scan

## Prerequisites

- Docker installed on the system
- No Dockerfile creation needed - only download the Docker images
- Write access to the project directory for configuration files and results


## SCA Scanner (Software Composition Analysis)

### Purpose
Scans dependencies and third-party packages for known vulnerabilities.

### Step-by-Step Instructions

#### 1. Pull the Docker Image
```bash
docker pull fluidattacks/sca:latest
```

#### 2. Create/Update Configuration File
Create `fluidattacks_scanners_config.yaml` in the project root with appropriate
settings.

**Example for Python Project:**
```yaml
strict: true # exit code different from 0 if any vulnerability is found
output:
  file_path: ./Fluid-Attacks-Results.csv
  format: CSV
working_dir: .
language: EN
sca:
  include:
    - pyproject.toml
    - uv.lock
  exclude:
    - venv/
  recursion-limit: 1000
```

#### 3. Path Specification Options

You can specify paths in two ways:

**Relative paths:**
```yaml
sca:
  include:
    - src/main/package.json
    - frontend/package-lock.json
```

**Unix-style globs:**
```yaml
sca:
  include:
    - glob(*)                    # All files
    - glob(**.json)             # All JSON files
  exclude:
    - glob(**/test/)            # All test directories
    - glob(src/**/test*.py)     # Test files in src
```

#### 4. Run the Scanner
```bash
docker run --rm -v /path/to/project:/src fluidattacks/sca:latest sca scan \
  fluidattacks_sca_config.yaml
```

#### 5. Add the output file to .gitignore
```
Fluid-Attacks-Results.csv
```

#### 7 Remediate vulnerabilities
- Review the `Fluid-Attacks-Results.csv` file
- If there are vulnerabilities, remediate them

## SAST Scanner (Static Application Security Testing)

### Purpose
Analyzes source code for security vulnerabilities, coding errors, and security
anti-patterns.

### Step-by-Step Instructions

#### 1. Pull the Docker Image
```bash
docker pull fluidattacks/sast:latest
```

#### 2. Create/Update Configuration File
Create `fluidattacks_scanners_config.yaml` in the project root.


**Example for Node.js/Express Project:**
```yaml
strict: true
output:
  file_path: ./Fluid-Attacks-Results.csv
  format: CSV
working_dir: .
language: EN
sast:
  include:
    - src/
    - routes/
    - controllers/
  exclude:
    - node_modules/
    - dist/
    - test/
  recursion-limit: 1000
```

#### 3. Path Specification Options

Same as SCA scanner - use relative paths or Unix-style globs.

#### 4. Run the Scanner
```bash
docker run --rm -v /path/to/project:/src fluidattacks/sast:latest sast scan \
  fluidattacks_sast_config.yaml
```

#### 5. Add the output file to .gitignore
```
Fluid-Attacks-Results.csv
```

#### 6 Remediate vulnerabilities
- Review the `Fluid-Attacks-Results.csv` file
- If there are vulnerabilities, remediate them


## Running Both Scanners (SAST + SCA)

### Purpose
Comprehensive security analysis covering both code vulnerabilities and
dependency vulnerabilities.

### Step-by-Step Instructions

#### 1. Pull the Docker Images
```bash
docker pull fluidattacks/sast:latest
docker pull fluidattacks/sca:latest
```

#### 2. Create/Update Configuration File
Create `fluidattacks_scanners_config.yaml` in the project root with both
scanner configurations.

**Example for Django Project:**
```yaml
strict: true # exit code different from 0 if any vulnerability is found
output:
  file_path: ./Fluid-Attacks-Results.csv
  format: CSV
working_dir: .
language: EN

sast:
  include:
    - project/
    - static/js/
    - templates/
  exclude:
    - venv/
    - __pycache__/
    - migrations/
    - static/images/
  recursion-limit: 1000

sca:
  include:
    - Pipfile
    - Pipfile.lock
    - requirements.txt
  exclude:
    - venv/
  recursion-limit: 1000
```

#### 3. Run Both Scanners
```bash
# Run SAST scanner
docker run --rm -v /path/to/project:/src fluidattacks/sast:latest sast scan \
 fluidattacks_scanners_config.yaml

# Run SCA scanner
docker run --rm -v /path/to/project:/src fluidattacks/sca:latest sca scan \
 fluidattacks_scanners_config.yaml
```


#### 4. Add the output file to .gitignore
```
Fluid-Attacks-Results.csv
```

#### 5 Remediate vulnerabilities
- Review the `Fluid-Attacks-Results.csv` file
- If there are vulnerabilities, remediate them


## Best Practices for Agents

### 1. Configuration File Management
- Always verify the correct paths for include/exclude before running
- Adjust configuration based on project structure
- Use `.gitignore` as a reference for exclude patterns
- Store configuration files in the project root and add them to .gitignore
- Add the output file (Fluid-Attacks-Results.csv) to .gitignore


## When to Run What

| Scenario | Scanner | Priority |
|----------|---------|----------|
| New dependency added | SCA | High |
| Code changes in auth/security | SAST | Critical |
| Weekly security audit | Both | Medium |
| Pre-deployment check | Both | Critical |
| Dependency version update | SCA | High |
| New feature development | SAST | Medium |
| Third-party library added | SCA | High |
| API endpoint changes | SAST | High |


## Integration with Development Workflow
- On Code Changes: Run SAST if source files modified
- On Dependency Changes: Run SCA if dependency files modified
- On User Request: Run appropriate scanner(s)
- Help with remediation: Always create/update security reports
- Re-scan: After fixes to verify remediation
