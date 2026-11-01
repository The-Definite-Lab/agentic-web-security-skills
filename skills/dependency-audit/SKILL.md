---
name: dependency-audit
description: >-
  Professional dependency security audit skill. Covers npm audit, composer audit,
  pip-audit, OWASP Dependency-Check, SCA (Software Composition Analysis),
  CVE scanning, automated dependency updates (Dependabot/Renovate), and
  lock file security. Use when installing packages, before deploying, or
  auditing existing projects for vulnerable dependencies.
  Trigger phrases: "dependency audit", "npm audit", "composer audit",
  "vulnerable packages", "CVE", "security scan", "SCA", "outdated packages",
  "dependency check", "package vulnerabilities".
---

# DEPENDENCY AUDIT — PROFESSIONAL SKILL

## SECTION 1: QUICK AUDIT COMMANDS (Run These First)

### JavaScript / Node.js / React
```bash
# Basic audit
npm audit

# Detailed with fix suggestions
npm audit --audit-level=moderate

# Auto-fix (safe fixes only — review before running)
npm audit fix

# Force fix (may break things — test after)
npm audit fix --force

# Check for outdated packages
npm outdated

# Interactive update tool
npx npm-check-updates   # shows what can be updated
npx npm-check-updates -u  # updates package.json (then npm install)
```

### PHP / Composer (Web & APIs)
```bash
# Audit composer packages
composer audit

# Check for outdated
composer outdated

# Update dependencies
composer update --dry-run    # preview changes first
composer update              # apply updates

# Check specific package
composer audit --format=json | jq '.advisories'
```

### Python (Scripts / n8n / AI)
```bash
# Install pip-audit
pip install pip-audit

# Run audit
pip-audit

# With requirements file
pip-audit -r requirements.txt

# Export report
pip-audit --format=json > security-report.json
```

---

## SECTION 2: FULL SCA AUDIT (OWASP Dependency-Check)

```bash
# Install (Java required)
# Download: https://owasp.org/www-project-dependency-check/

# Windows
dependency-check.bat --project "MyProject" --scan . --format HTML --out reports/

# Scan Node project
dependency-check.bat --project "MyApp" \
  --scan ./package.json \
  --scan ./package-lock.json \
  --format HTML --format JSON \
  --out ./security-reports/

# View report
start reports/dependency-check-report.html
```

---

## SECTION 3: AUTOMATED MONITORING (Set & Forget)

### GitHub Dependabot (Recommended — Free)
```yaml
# .github/dependabot.yml — add to every repo
version: 2
updates:
  # JavaScript / Node.js
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"   # or "daily" for high-security projects
    open-pull-requests-limit: 10
    reviewers:
      - "your-github-username"
    labels:
      - "dependencies"
      - "security"

  # PHP / Composer (for Moodle)
  - package-ecosystem: "composer"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 5

  # GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

### Renovate Bot (More powerful than Dependabot)
```json
// renovate.json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:base"],
  "automerge": false,
  "prCreation": "immediate",
  "vulnerabilityAlerts": {
    "enabled": true,
    "labels": ["security-critical"]
  },
  "packageRules": [
    {
      "matchUpdateTypes": ["patch"],
      "automerge": true    // auto-merge safe patch updates
    }
  ]
}
```

---

## SECTION 4: SEVERITY LEVELS & ACTION MATRIX

| Severity | CVSS Score | Action Required | Timeline |
|---|---|---|---|
| **Critical** | 9.0–10.0 | Update immediately. Block deploy if unfixed. | Same day |
| **High** | 7.0–8.9 | Update in current sprint. Don't ship with known High. | 7 days |
| **Moderate** | 4.0–6.9 | Schedule update. Document if deferring. | 30 days |
| **Low** | 0.1–3.9 | Update at next dependency maintenance cycle. | Next release |

---

## SECTION 5: FIREBASE / SUPABASE SDK MONITORING

```bash
# Check firebase versions
npm list firebase
npm list firebase-admin
npm list firebase-functions

# Firebase SDK CVE check
npm audit --workspace=. 2>/dev/null | grep -i firebase

# Supabase SDK
npm list @supabase/supabase-js
npm audit | grep supabase

# Update Firebase
npm install firebase@latest
npm install firebase-admin@latest
npm install firebase-functions@latest

# Update Supabase
npm install @supabase/supabase-js@latest
```

---

## SECTION 6: LICENSE COMPLIANCE CHECK

```bash
# Check licenses of all dependencies (legal compliance)
npx license-checker --summary

# Check for problematic licenses
npx license-checker --failOn "GPL-2.0;GPL-3.0;AGPL-3.0"

# Export full license report
npx license-checker --csv > licenses.csv
```

---

## SECTION 7: PRE-DEPLOY AUDIT CHECKLIST

Run before every production deployment:

```bash
# JavaScript projects
npm audit --audit-level=high   # fail if any High or Critical
echo "Exit: $?"                # must be 0

# PHP/Moodle projects
composer audit 2>&1
echo "Exit: $?"

# Check for known-bad packages (manually verify these)
cat package-lock.json | python -c "
import json, sys
lock = json.load(sys.stdin)
packages = lock.get('packages', {})
for name, info in packages.items():
    resolved = info.get('resolved', '')
    if 'github.com' in resolved and 'raw.' in resolved:
        print(f'WARNING: Direct GitHub source: {name} -> {resolved}')
"
```

---

## SECTION 8: LOCK FILE SECURITY

```bash
# ALWAYS commit lock files
# package-lock.json  → commit (Node)
# composer.lock      → commit (PHP)
# requirements.txt   → commit (Python — pin exact versions)
# Gemfile.lock       → commit (Ruby)

# Verify lock file integrity
npm ci          # instead of npm install (uses exact lock file versions)
composer install --no-dev  # use lock file in production

# Detect tampering (CI pipeline check)
git diff package-lock.json  # should only change when package.json changes

# ❌ NEVER delete lock files
# ❌ NEVER run npm install --legacy-peer-deps in production
# ❌ NEVER ignore lock file conflicts — resolve them properly
```

---

## SECTION 9: MOODLE-SPECIFIC DEPENDENCY AUDIT

```bash
# Moodle plugin composer packages
cd /path/to/moodle/plugin
composer audit

# Check Moodle core version for known CVEs
# Reference: https://moodletracker.org/issues/?jql=project+%3D+MDL+AND+labels+%3D+security

# Moodle Security Announcements RSS:
# https://moodle.org/security/

# Check installed Moodle version
php admin/cli/cfg.php --name=release

# List all installed plugins and versions
php admin/cli/maintenance.php --list
```

---

## SECTION 10: AUTOMATED CI/CD INTEGRATION

```yaml
# GitHub Actions — security-audit.yml
name: Security Audit

on:
  push:
    branches: [main, dev]
  pull_request:
  schedule:
    - cron: '0 9 * * 1'  # Every Monday 9am

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run security audit
        run: npm audit --audit-level=high
        
      - name: Check for secrets
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          
      - name: License check
        run: npx license-checker --failOn "GPL-2.0;GPL-3.0"
```
