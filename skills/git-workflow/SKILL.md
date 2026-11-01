---
name: git-workflow
description: >-
  Professional Git workflow skill. Covers Conventional Commits, branch strategy,
  PR/merge workflow, secret scanning before commit, .gitignore best practices,
  Git hooks setup, commit signing, and team collaboration standards.
  Use when setting up a new repo, reviewing git practices, or automating
  commit/push workflows.
  Trigger phrases: "git setup", "git workflow", "conventional commits",
  "branch strategy", "git hooks", "commit", "git push", "PR workflow",
  "gitignore", "git security".
---

# GIT WORKFLOW — PROFESSIONAL SKILL

## SECTION 1: BRANCH STRATEGY

```
main          ← Production only. Protected. No direct push.
  │
  ├── dev     ← Integration branch. All features merged here first.
  │     │
  │     ├── feature/login-auth       ← Feature branches
  │     ├── feature/api-auth-v2
  │     ├── fix/firebase-rules-idor
  │     └── hotfix/security-patch    ← Emergency fix (merges to main + dev)
  │
  └── release/v1.2.0                 ← Release candidate branch
```

### Branch Rules
- `main` → production only. Direct push **disabled**. PR required.
- `dev` → integration. All features go here first.
- `feature/*` → one feature per branch. Delete after merge.
- `fix/*` → bug fixes. Reference issue number.
- `hotfix/*` → critical production fixes. Merge to `main` AND `dev`.
- `release/*` → release prep. No new features after branch created.

---

## SECTION 2: CONVENTIONAL COMMITS (Mandatory Format)

```
<type>(<scope>): <short description>

[optional body]

[optional footer]
```

### Types
| Type | When to Use | Example |
|---|---|---|
| `feat` | New feature | `feat(auth): add Google OAuth login` |
| `fix` | Bug fix | `fix(firestore): resolve IDOR on user documents` |
| `security` | Security patch | `security(moodle): add sesskey to upload form` |
| `chore` | Build/tool changes | `chore: update firebase-admin to 12.0.0` |
| `refactor` | Code refactor | `refactor(api): extract auth middleware` |
| `test` | Tests only | `test(rules): add IDOR test cases for Firestore` |
| `docs` | Documentation | `docs(readme): add deployment instructions` |
| `perf` | Performance | `perf(db): add index on userId field` |
| `ci` | CI/CD changes | `ci: add secret scanning to GitHub Actions` |
| `revert` | Revert commit | `revert: feat(auth) — breaks mobile login` |

### Commit Message Examples
```bash
# ✅ Good commits
git commit -m "feat(moodle): add privacy API for quiz results plugin"
git commit -m "fix(firebase): enforce RLS on user documents — prevent IDOR"
git commit -m "security(supabase): enable RLS on all tables"
git commit -m "feat(n8n): add webhook signature validation"

# ❌ Bad commits (never do this)
git commit -m "fix"
git commit -m "update"
git commit -m "changes"
git commit -m "WIP"
git commit -m "asdfgh"
```

---

## SECTION 3: SECRET SCANNING (Before Every Push)

### Install gitleaks (One-time setup)
```bash
# Windows (Chocolatey)
choco install gitleaks

# Or download from: https://github.com/gitleaks/gitleaks/releases
```

### Scan Before Push
```bash
# Scan current repo for secrets
gitleaks detect --source . --verbose

# Scan specific commit range
gitleaks detect --source . --log-opts="HEAD~5..HEAD"

# Scan staged files (pre-commit)
gitleaks protect --staged
```

### Pre-commit Hook (Auto-scan on every commit)
```bash
# .git/hooks/pre-commit  (make executable: chmod +x)
#!/bin/sh

echo "🔍 Scanning for secrets..."
gitleaks protect --staged --verbose
if [ $? -ne 0 ]; then
  echo "❌ SECRET DETECTED. Commit blocked. Remove secret first."
  exit 1
fi

echo "✅ No secrets found. Proceeding with commit."
```

### Common Patterns to Catch
```bash
# Manual scan for common patterns
grep -r "AIza" . --include="*.js" --include="*.ts" --include="*.php"  # Google API key
grep -r "sk-" . --include="*.js" --include="*.ts"    # OpenAI key
grep -r "firebase" . --include="*.env*"               # Firebase config
grep -rE "password\s*=\s*['\"][^'\"]{4,}" . --include="*.php"  # PHP passwords
```

---

## SECTION 4: .GITIGNORE — MANDATORY PATTERNS

```gitignore
# === ENVIRONMENT & SECRETS ===
.env
.env.*
.env.local
.env.development
.env.production
.env.staging
*.secret
secrets/
credentials/

# === FIREBASE ===
.firebaserc          # contains project IDs — okay to commit if no secrets
firebase-debug.log
firebase-debug.*.log
.firebase/
serviceAccountKey.json    # NEVER commit — Firebase Admin SDK key

# === NODE ===
node_modules/
dist/
build/
.next/
.nuxt/

# === PHP / MOODLE ===
vendor/
config.php           # Moodle config — contains DB password
*.log
/moodledata/

# === ANDROID ===
*.jks               # Keystore — NEVER commit
*.keystore
local.properties    # Contains SDK path — okay but personal
google-services.json  # Contains Firebase config — review carefully

# === GENERAL ===
.DS_Store
Thumbs.db
*.swp
*.swo
__pycache__/
*.pyc
.idea/
.vscode/             # Optional — team preference
```

---

## SECTION 5: GIT HOOKS SETUP

```bash
# Option 1: Husky (Node projects) — recommended
npm install --save-dev husky lint-staged
npx husky init

# .husky/pre-commit
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

# Secret scan
gitleaks protect --staged --verbose || exit 1

# Lint + format
npx lint-staged || exit 1

# package.json — lint-staged config
{
  "lint-staged": {
    "*.{js,ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{css,scss}": ["prettier --write"],
    "*.php": ["phpcs --standard=moodle"]
  }
}

# .husky/commit-msg
#!/bin/sh
# Enforce conventional commits
npx --no -- commitlint --edit "$1"
```

---

## SECTION 6: GITHUB / GITLAB SECURITY SETTINGS

```
Repository Settings Checklist:
├── Branch Protection (main):
│   ├── [ ] Require PR before merging
│   ├── [ ] Require at least 1 review
│   ├── [ ] Require status checks to pass (CI)
│   ├── [ ] No direct push to main
│   └── [ ] No force push
│
├── Secret Scanning:
│   ├── [ ] GitHub Secret Scanning enabled
│   ├── [ ] Push protection enabled (blocks push with secrets)
│   └── [ ] Dependabot alerts enabled
│
├── Actions / CI:
│   ├── [ ] Secret stored in GitHub Secrets (never in code)
│   ├── [ ] Third-party actions pinned to specific commit SHA
│   └── [ ] Workflow permissions: minimum required
│
└── Access:
    ├── [ ] No unnecessary collaborators
    ├── [ ] 2FA required for all team members
    └── [ ] Deploy keys: read-only where possible
```

---

## SECTION 7: DAILY GIT WORKFLOW (Easiest Reference)

```bash
# Start of day — sync
git checkout dev
git pull origin dev

# Start new feature
git checkout -b feature/your-feature-name

# During development — commit often
git add -A
git status                    # always check before commit
git diff --staged             # review changes
git commit -m "feat(scope): description"

# Before pushing
gitleaks protect --staged     # secret scan
git push origin feature/your-feature-name

# Create PR → dev (not main)
# After PR merged → delete feature branch

# Release to production
git checkout main
git merge dev
git tag -a v1.2.0 -m "Release v1.2.0"
git push origin main --tags
```

---

## SECTION 8: EMERGENCY — IF SECRET WAS COMMITTED

```bash
# IMMEDIATE ACTION — do this within minutes
# 1. REVOKE THE SECRET FIRST (don't wait)
# Go to Firebase Console / GitHub Settings / OpenAI / etc. and revoke

# 2. Remove from git history
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch path/to/secret-file' \
  --prune-empty --tag-name-filter cat -- --all

# Or use BFG Repo Cleaner (faster)
java -jar bfg.jar --delete-files secret-file.env
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push origin --force --all

# 3. Notify team to re-clone the repo
# 4. Rotate ALL secrets that were in that file
# 5. Audit logs for any usage of leaked credentials
```
