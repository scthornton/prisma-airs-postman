# Security Policy

## 🔒 Reporting Security Issues

If you discover a security vulnerability in this repository, please **do not** use the public issue tracker. Instead:

1. Create a private security advisory on GitHub: **Security tab → Advisories → New draft**
2. Or contact the repository owner directly

Security reports will be acknowledged within 48 hours.

## 🛡️ Security Best Practices

### API Key Safety

**⚠️ CRITICAL: Never commit API keys to this repository**

This repository integrates with multiple services that require API credentials:
- Prisma AI Runtime Security (AIRS)
- OpenAI
- Anthropic Claude
- Google Vertex AI

**Safe credential management:**
- ✅ **Always use environment variables** for all credentials
- ✅ **Clear notebook outputs** before committing: `Cell → All Output → Clear`
- ✅ **Review all files** for hardcoded secrets before pushing
- ✅ **Add `.env` files to `.gitignore`** (already configured)
- ✅ **Use `.env.example` files** to document required variables (without values)

### Before Committing - Security Checklist

Run this checklist before every commit:

```bash
# 1. Clear all Jupyter notebook outputs
jupyter nbconvert --clear-output --inplace **/*.ipynb

# 2. Search for potential API keys and secrets
grep -r "sk-" . | grep -v ".git"                    # OpenAI keys
grep -r "api_key.*=" . | grep -v "os.getenv" | grep -v ".git"
grep -r "PANW" . | grep -v "os.getenv" | grep -v ".git"

# 3. Check for .env files
git status | grep .env  # Should return nothing

# 4. Verify sensitive files are ignored
git check-ignore .env config.json credentials.json
```

### Recommended Environment Setup

#### Option 1: Shell Environment Variables (Recommended)

Add to your shell config (`~/.zshrc` or `~/.bashrc`):

```bash
# Prisma AIRS
export PANW_AI_SEC_API_KEY="your-key"
export PRISMA_AIRS_PROFILE="your-profile"

# LLM Providers
export OPENAI_API_KEY="your-key"
export ANTHROPIC_API_KEY="your-key"

# Google Cloud (if using Vertex AI)
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/credentials.json"
```

#### Option 2: .env File (Good for Development)

Create `.env` in repository root (already in `.gitignore`):

```bash
PANW_AI_SEC_API_KEY=your-key
PRISMA_AIRS_PROFILE=your-profile
OPENAI_API_KEY=your-key
ANTHROPIC_API_KEY=your-key
```

Load with:
```python
from dotenv import load_dotenv
load_dotenv()
```

#### Option 3: Config File (Advanced)

Create `config.json` (already in `.gitignore`):

```json
{
  "panw_api_key": "your-key",
  "security_profile": "your-profile",
  "openai_key": "your-key",
  "anthropic_key": "your-key"
}
```

**IMPORTANT:** Never commit `config.json` - provide `config.example.json` instead.

## 📦 Dependency Security

### Keeping Dependencies Updated

```bash
# Python dependencies
pip list --outdated
pip install --upgrade -r requirements.txt

# Node.js dependencies (for secure-chatbot)
npm outdated
npm audit
npm audit fix
```

### Dependency Vulnerability Scanning

This repository supports automated security tools:

**GitHub Features:**
- **Dependabot** - Automatically creates PRs for dependency updates
- **GitHub Security Advisories** - Alerts for known vulnerabilities
- **Code Scanning** - Detects security issues in code

**Manual scanning:**

```bash
# Python
pip install safety
safety check -r requirements.txt

# Node.js
npm audit
npm audit fix --force  # Use with caution
```

## 🚨 Known Risks & Mitigations

### Risk: Notebook Output May Contain Sensitive Data

**Threat:** Jupyter notebook outputs may contain API responses with PII, test data, or credentials.

**Mitigation:**
- Always clear outputs before sharing: `Cell → All Output → Clear`
- Review API responses for PII or sensitive information
- The `.gitignore` is configured to ignore notebook checkpoints
- Use `jupyter nbconvert --clear-output` in pre-commit hooks

**Automation:**
```bash
# Add to .git/hooks/pre-commit
jupyter nbconvert --clear-output --inplace $(git diff --cached --name-only --diff-filter=ACM | grep '\.ipynb$')
```

### Risk: Postman Environment Variables May Leak

**Threat:** Postman environment files contain API keys and tokens.

**Mitigation:**
- `.gitignore` is configured to ignore `*.postman_environment.json`
- Use collection variables with `{{x-pan-token}}` placeholders
- Never export Postman environments with actual credentials
- Provide `postman_environment.example.json` with placeholder values

### Risk: Test Prompts May Trigger API Rate Limits

**Threat:** Batch testing may exceed API rate limits or cost quotas.

**Mitigation:**
- Use synchronous notebook for individual tests
- Add delays between batch requests in async notebook
- Monitor your Prisma AIRS API usage dashboard
- Set up billing alerts in your cloud provider

### Risk: Hardcoded Test Data in Code

**Threat:** Test data might accidentally include real PII or credentials.

**Mitigation:**
- Example prompts use fake/sanitized data only
- Do not use real SSNs, credit cards, or production data in tests
- Use EICAR test strings for malware detection (standard test pattern)
- Replace example data before sharing notebooks or collections

### Risk: Local .env Files in Development

**Threat:** `.env` files contain secrets and may be accidentally committed.

**Mitigation:**
- `.env` is in `.gitignore` (multiple patterns)
- Provide `.env.example` files without actual credentials
- Use `git status` to verify `.env` is ignored
- Review `.gitignore` patterns regularly

## 🔐 Component-Specific Security

### Jupyter Notebooks (`prisma-airs-jupyter/`)

**Security Features:**
- ✅ Environment variable usage by default
- ✅ No hardcoded credentials in shared versions
- ✅ Clear documentation on credential management
- ✅ Error handling for API failures
- ✅ Timeout configurations to prevent hanging requests

**Security Checklist:**
- [ ] Clear all outputs before committing
- [ ] Verify no hardcoded API keys
- [ ] Check for accidental PII in test prompts
- [ ] Review API responses for sensitive data

### Postman Collections (`postman/`)

**Security Features:**
- ✅ Collection-level authentication with variables
- ✅ No hardcoded API keys in requests
- ✅ Environment variables for sensitive values
- ✅ Organized test cases for threat detection

**Security Checklist:**
- [ ] Use collection variables (`{{x-pan-token}}`)
- [ ] Never export environments with real credentials
- [ ] Provide example environment files
- [ ] Document required variables in README

### Secure Chatbot (`secure-chatbot/`)

**Security Features:**
- ✅ Environment-based configuration
- ✅ AIRS security scanning on all prompts
- ✅ Separate backend/frontend architecture
- ✅ CORS configuration
- ✅ Input validation

**Security Checklist:**
- [ ] Review CORS settings for production
- [ ] Validate all user inputs
- [ ] Implement rate limiting
- [ ] Add authentication for production use
- [ ] Configure secure HTTPS in production
- [ ] Review environment variable exposure

## 📋 Supported Versions

This is an active development/testing repository. Security updates apply to:

| Version | Supported          |
| ------- | ------------------ |
| Latest commit (main) | ✅ Supported |
| Feature branches | ⚠️ May be experimental |
| Older commits | ❌ Not supported - please update |

## 🔍 Security Features Overview

### Built-in Security Practices

- ✅ Comprehensive `.gitignore` for all secret patterns
- ✅ Environment variable usage across all components
- ✅ No hardcoded credentials in shared code
- ✅ Clear documentation on credential management
- ✅ Error handling for API failures
- ✅ Timeout configurations
- ✅ Input validation examples
- ✅ Secure by default configurations

### Automated Security

- ✅ Dependabot enabled for dependency updates
- ✅ GitHub Security Advisories for vulnerabilities
- ✅ `.gitignore` patterns for common secret files
- ✅ Pre-commit hook examples provided

## 📚 Additional Resources

### Official Documentation

- [Prisma AIRS Security Best Practices](https://pan.dev/prisma-airs/)
- [Prisma AIRS API Authentication](https://pan.dev/prisma-airs/api/airuntimesecurity/scan/)

### Security Standards

- [Jupyter Security Best Practices](https://jupyter-notebook.readthedocs.io/en/stable/security.html)
- [OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [CWE Top 25 Software Weaknesses](https://cwe.mitre.org/top25/)

### Tools

- [git-secrets](https://github.com/awslabs/git-secrets) - Prevent committing secrets
- [detect-secrets](https://github.com/Yelp/detect-secrets) - Find secrets in code
- [truffleHog](https://github.com/trufflesecurity/trufflehog) - Scan git history for secrets

## ✅ Complete Security Checklist for Contributors

Before submitting any changes:

### Code Security
- [ ] No API keys, tokens, or secrets in code
- [ ] Environment variables used for all credentials
- [ ] No real PII or production data in examples
- [ ] Input validation implemented
- [ ] Error handling in place

### Jupyter Notebooks
- [ ] All notebook outputs cleared
- [ ] No hardcoded credentials in cells
- [ ] Test prompts use sanitized/fake data only
- [ ] API responses reviewed for sensitive data

### Git & Version Control
- [ ] `.gitignore` includes all sensitive file patterns
- [ ] `.env` files are ignored
- [ ] No credential files in git history
- [ ] Commit messages don't contain secrets

### Dependencies
- [ ] Dependencies are up to date
- [ ] No known vulnerabilities (`npm audit`, `safety check`)
- [ ] Only trusted packages used
- [ ] Package lock files updated

### Documentation
- [ ] Security best practices documented
- [ ] Example configuration files provided (`.example` versions)
- [ ] Required environment variables documented
- [ ] Credentials management explained

### Testing
- [ ] Security scanning integrated (AIRS)
- [ ] Error cases handled gracefully
- [ ] Rate limiting tested
- [ ] Authentication tested

## 🚀 Secure Development Workflow

### 1. Initial Setup

```bash
# Clone repository
git clone https://github.com/yourusername/prisma-airs.git
cd prisma-airs

# Set up environment
cp .env.example .env
# Edit .env with your credentials (DO NOT COMMIT)

# Install dependencies
pip install -r requirements.txt
```

### 2. Development

```bash
# Work in feature branch
git checkout -b feature/my-feature

# Make changes...

# Before committing, run security checks
jupyter nbconvert --clear-output --inplace **/*.ipynb
grep -r "api_key.*=" . | grep -v "os.getenv"
git status | grep .env  # Should be empty
```

### 3. Pre-Commit (Recommended Hook)

Create `.git/hooks/pre-commit`:

```bash
#!/bin/bash

# Clear notebook outputs
jupyter nbconvert --clear-output --inplace $(git diff --cached --name-only | grep '\.ipynb$')

# Check for potential secrets
if git diff --cached | grep -E "(api_key|token|password|secret)" | grep -v "os.getenv"; then
    echo "❌ ERROR: Potential secret found in staged changes"
    exit 1
fi

# Check for .env files
if git diff --cached --name-only | grep "\.env$"; then
    echo "❌ ERROR: .env file in staged changes"
    exit 1
fi

echo "✅ Pre-commit checks passed"
```

Make it executable:
```bash
chmod +x .git/hooks/pre-commit
```

### 4. Pull Request Checklist

- [ ] Security checklist completed
- [ ] No secrets in code or commit history
- [ ] Tests pass
- [ ] Documentation updated
- [ ] Code reviewed by at least one other person

## 🔔 Security Update Notifications

To receive security updates:

1. **Watch this repository** on GitHub (Watch → All Activity)
2. **Enable Security Alerts** in your GitHub settings
3. **Review Dependabot PRs** regularly
4. **Check GitHub Security Advisories** tab

## 📞 Contact

For security concerns:
- Create a private security advisory on GitHub
- Or contact the repository owner directly

**Please do not disclose security vulnerabilities publicly until they have been addressed.**

---

**Last Updated:** October 2025

**Security Policy Version:** 1.0
