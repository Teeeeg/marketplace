---
name: secret-scanning
description: Scan local files, content, or recent git changes for secrets such as API keys, passwords, tokens, and credentials using local commands and pattern matching.
---

# Secret Scanning Skill

## Overview

This skill scans local content, files, and git changes for secrets using local pattern matching and context checks. It helps identify sensitive material like API keys, passwords, and credentials that could pose a security risk if exposed.

### What counts as a secret?

In this context, values that grant access, impersonate a user or service, sign requests, or decrypt protected data are generally treated as secrets.

Treat these as high-confidence secret material:

- Access tokens, API keys, and bearer credentials
- Passwords, database DSNs with embedded credentials, and SMTP auth values
- Private keys, signing keys, certificates with private key blocks, and SSH keys
- OAuth client secrets, refresh tokens, and webhook secrets
- Cloud credentials (AWS/GCP/Azure) and CI/CD deployment credentials

Prefer context, not just regex:

- Values near names like `password`, `token`, `secret`, `client_secret`, `private_key`, or `authorization` are higher risk
- Long high-entropy strings in config files, scripts, and test fixtures deserve review even if unlabeled
- Treat uncertain findings as sensitive until verified

Not everything that looks random is a secret. Example placeholders such as `YOUR_API_KEY_HERE`, obvious test stubs, and documented sample values can be false positives.

### Why this is important

This skill scans for secrets that could compromise security if leaked. A committed secret can persist in git history, trigger incident response, and block deployment at push protection checks.

**Important**: Only use this skill when a user explicitly asks to scan content or check for secrets. Do not run secret scanning unprompted or as part of general workflows.

## Common Scenarios

| User goal                              | How to respond       | Scan approach       |
| -------------------------------------- | -------------------- | ------------------ |
| Check a config snippet or code paste   | Scan as content      | Pattern matching + context |
| Check a specific file in the repo      | Inspect file, then scan | Pattern matching + context |
| Check all staged changes before commit | Get staged diff, then scan  | Pattern matching + context |

## Installation

### Prerequisites & Inputs

No external service setup is required. This skill is local-first.

**Required information for scanning**:

- **Content to scan**: A user-provided snippet, file content, or staged diff output

**What NOT to scan**: By default, avoid scanning large generated or vendor content (for example: `node_modules/`, build artifacts, compiled assets, or other machine-generated files) for performance and noise reduction. Files listed in `.gitignore` may still contain secrets (such as `.env` or local config), so only skip them if the user agrees; if the user explicitly asks to scan them, include them in the scan.

### Scan Content

**When to use**: The user provides text or code snippets they want checked for secrets.

**How**: Inspect the snippet directly and run local pattern checks for common secret indicators (for example: `password\s*=`, `api[_-]?key`, `token`, `client_secret`, `private_key`, `authorization:\s*bearer`).

**Example**

```
You: Check if this config snippet has secrets
User provides:
DB_PASSWORD=MyP@ssw0rd123
API_KEY=abc123xyz

Agent: I'll scan that for secrets.

     ⚠️ Found 2 potential secret(s):
     1. DB_PASSWORD value looks like a password
     2. API_KEY value looks like an API key

     Consider replacing hardcoded values with environment variables.
```

### Scan File(s)

**When to use**: The user asks to check specific files in the repository for secrets.

**How**: Inspect file contents and search for suspicious patterns in the file.

**Example (secrets found)**

```
You: Check if config.py has secrets
Agent: I'll read and scan that file.

     ⚠️ Found 1 potential secret:
     - Line 10: database_password value looks like a password

     Suggestion: Move to environment variable or .env file.
```

**Example (no secrets found)**

```
You: Check if utils.js has any secrets
Agent: I'll read and scan that file.

     ✅ No secrets detected in utils.js
```

### Scan Staged Changes

**When to use**: The user wants to check git staged changes for secrets before committing.

**How**: Get staged content from the local repository state, then scan the diff text locally using secret-pattern heuristics.

**Example (clean)**

```
You: Check my staged changes for secrets before I commit
Agent: I'll scan your staged changes.

     ✅ No secrets found in staged changes. Safe to commit.
```

**Example (secrets detected)**

```
You: Scan my staged changes
Agent: I'll check your staged changes.

     ⚠️ Found 1 potential secret in staged changes:
     - database.js line 5: password value looks like a credential

     Suggestion: Unstage that change, move the credential to env vars, and restage.
```

## How the Scan Works

When you run this skill locally:

- **Input**: File content, snippet content, or staged diff text
- **Process**: Pattern matching + context checks (variable names, assignment style, token shape/entropy, and file type)
- **Output**: Potential secret findings with file or line context and confidence

This approach is fast and private to your local workspace, but may produce false positives/negatives compared to provider-grade detectors.

## Scanning Transparency

### How Your Content Is Processed

When you request a scan, content is processed locally in the workspace using local scanning logic only. No external call is required for this skill.

### What to Do With Results

If secrets are found:

- **Obvious hardcoded values**: Move them to environment variables or `.env` files
- **Config files**: Check if `example.env` or documentation exists that shows the expected structure
- **Already committed**: If the secret was already pushed, credential rotation may be needed (outside this skill's scope)

If no secrets are found:

- The scan completed successfully
- Check the output format in the scan result to make sure coverage was complete

## Learn More

For more details on secret hygiene and credential management:

- [Credential Management Best Practices](https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning): Guidance on handling credentials safely
- [GitHub Push Protection](https://docs.github.com/en/code-security/secret-scanning/working-with-push-protection): Preventing secrets from reaching your repository