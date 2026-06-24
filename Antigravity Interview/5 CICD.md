# ⚙️ CI/CD Interview Guide — Complete Q&A

> 📚 Source: [iam-veeramalla/devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide) — `05-cicd/`
> 🎯 Purpose: Master Jenkins, Maven, GitHub Actions, ArgoCD & CI/CD troubleshooting for interviews

---

## 📋 Table of Contents

| # | Topic |
|---|-------|
| [1](#-1-jenkins-shared-libraries) | Jenkins Shared Libraries |
| [2](#-2-popular-maven-build-targets) | Maven Build Targets (Top 5) |
| [3](#-3-artifact-repository--jfrog) | Artifact Repository — JFrog |
| [4](#-4-configure-jfrog-in-maven) | Configure JFrog in Maven |
| [5](#-5-build-fails-in-ci-but-not-locally) | Build Fails in CI but Not Locally |
| [6](#-6-ci-passed-but-app-broken-in-prod) | CI Passed but App Broken in Prod |
| [7](#-7-ci-pipeline-slows-down) | CI Pipeline Slows Down |
| [8](#-8-ci-does-not-trigger-on-feature-branch) | CI Doesn't Trigger on Feature Branch |
| [9](#-9-ci-cannot-download-dependencies) | CI Cannot Download Dependencies |
| [10](#-10-python-build-fails-in-ci) | Python Build Fails in CI |
| [11](#-11-python-build-process) | Python Build Process |
| [12](#-12-static-code-analysis-advantages) | Static Code Analysis Advantages |
| [13](#-13-static-analysis-slows-down-ci) | Static Analysis Slows Down CI |
| [14](#-14-argocd-app-out-of-sync) | ArgoCD App Out of Sync |
| [15](#-15-jenkins-email-notification-on-failure) | Jenkins Email on Build Failure |
| [Cheatsheet](#-master-cheatsheet) | Master Cheatsheet |

---

---

# 📚 1. Jenkins Shared Libraries

**Q: What are Jenkins Shared Libraries and how do they work?**

> **Problem:** 50 Jenkinsfiles all repeating the same Docker build + push + deploy logic.
> **Solution:** Extract into a Shared Library — one place, reused everywhere.

### Repository Structure

```
my-shared-library/          ← a Git repository
├── vars/
│   └── buildAndPush.groovy ← global functions (auto-loaded by name)
├── src/
│   └── org/company/Utils.groovy  ← classes for complex logic
├── resources/
│   └── templates/config.xml      ← static resources
└── README.md
```

### Setup Steps

```
1. Create the Git repo with vars/ structure

2. Register in Jenkins:
   Manage Jenkins → System → Global Pipeline Libraries
   → Name: my-shared-library
   → Source: Git → URL of your repo
   → Default version: main

3. Use in Jenkinsfile:
```

```groovy
@Library('my-shared-library') _

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                buildAndPush('my-app:latest')   // calls vars/buildAndPush.groovy
            }
        }
    }
}
```

### Example: `vars/buildAndPush.groovy`

```groovy
def call(String imageName) {
    sh "docker build -t ${imageName} ."
    sh "docker push ${imageName}"
    echo "Built and pushed: ${imageName}"
}
```

### Benefits

| Benefit | Detail |
|---|---|
| **DRY** | Write once, use in 50 pipelines |
| **Consistency** | All teams follow the same build standards |
| **Versioned** | Tag library versions — pipelines can pin to `v1.2.0` |
| **Auditable** | Changes to shared logic are reviewable via PR |

### 🎙️ Answer
> *"Jenkins Shared Libraries centralize reusable Groovy pipeline code in a separate Git repo. The `vars/` directory holds global functions — any `.groovy` file there becomes callable by its filename. I register the library in Manage Jenkins → Global Pipeline Libraries, then import it in Jenkinsfiles with `@Library('my-shared-library')`. When all teams use the same library, updating Docker build logic in one place propagates to every pipeline — reducing duplication and drift."*

---

---

# 🏗️ 2. Popular Maven Build Targets

**Q: Talk about 5 Maven build targets you use daily.**

```
Maven Lifecycle (in order):
validate → compile → test → package → verify → install → deploy
```

| Command | What It Does | When to Use |
|---|---|---|
| `mvn clean` | Deletes `target/` directory | Before every fresh build |
| `mvn compile` | Compiles `src/main/java` → `.class` files | Verify no compilation errors |
| `mvn test` | Runs unit tests (JUnit/TestNG) | Every commit, in CI |
| `mvn package` | Creates `.jar` or `.war` artifact | When ready to ship |
| `mvn install` | Installs artifact to local `~/.m2/repository` | Local multi-module projects |

```bash
# Most common CI pipeline command:
mvn clean package -DskipTests       # skip tests for speed (only if separate test stage)

# Include tests:
mvn clean test package

# Full publish to artifact repo:
mvn clean deploy                    # runs compile → test → package → push to repo
```

**Bonus targets:**

```bash
mvn dependency:analyze              # find unused/undeclared dependencies
mvn dependency:tree                 # visualize full dependency graph
mvn versions:display-dependency-updates   # check for newer versions
```

### 🎙️ Answer
> *"The five I use most: `clean` to wipe the target directory before building, `compile` to check for compilation errors, `test` to run unit tests, `package` to produce the JAR or WAR artifact, and `install` to publish it to the local Maven cache for use by other modules. In CI pipelines I typically run `mvn clean package` or `mvn clean deploy` depending on whether I need to publish to Artifactory."*

---

---

# 📦 3. Artifact Repository — JFrog

**Q: Which artifact repository do you use and why?**

> We use **JFrog Artifactory** as the primary artifact repository.

### Why JFrog?

| Feature | Detail |
|---|---|
| **Multi-format** | Maven, npm, Docker, PyPI, Helm, Go — one tool |
| **Proxy public repos** | Caches Maven Central, Docker Hub — faster + resilient |
| **RBAC** | Fine-grained access control per team/repo |
| **Retention policies** | Auto-delete old snapshots — saves storage |
| **CI/CD integration** | Native plugins for Jenkins, GitHub Actions |

### Typical Workflow

```
Developer commits → Jenkins runs mvn clean deploy
                          ↓
                   Artifact pushed to JFrog
                    (libs-release-local)
                          ↓
             Downstream stages / other teams
             pull artifact from JFrog ✅
```

### Alternatives

| Tool | Best For |
|---|---|
| **Sonatype Nexus** | Open-source teams, self-hosted |
| **AWS CodeArtifact** | Pure AWS environments |
| **GitHub Packages** | Small teams already on GitHub |

---

---

# 🔧 4. Configure JFrog in Maven

**Q: How do you configure Artifactory in Maven?**

### Step 1: Add Credentials to `~/.m2/settings.xml`

```xml
<settings>
  <servers>
    <server>
      <id>artifactory</id>
      <username>my-username</username>
      <password>my-api-token</password>   <!-- use API token, not password -->
    </server>
  </servers>
</settings>
```

> ⚠️ Never hardcode credentials in `pom.xml` — always use `settings.xml` or environment variables.

### Step 2: Configure Repository URLs in `pom.xml`

```xml
<distributionManagement>
  <repository>
    <id>artifactory</id>
    <name>Release Repository</name>
    <url>https://artifactory.example.com/artifactory/libs-release-local</url>
  </repository>
  <snapshotRepository>
    <id>artifactory</id>
    <name>Snapshot Repository</name>
    <url>https://artifactory.example.com/artifactory/libs-snapshot-local</url>
  </snapshotRepository>
</distributionManagement>
```

### Step 3: Deploy

```bash
mvn clean deploy
# Runs: compile → test → package → uploads .jar + .pom to Artifactory
```

**In Jenkins (using env variables for security):**
```groovy
withCredentials([usernamePassword(
    credentialsId: 'artifactory-creds',
    usernameVariable: 'ARTIFACTORY_USER',
    passwordVariable: 'ARTIFACTORY_PASS'
)]) {
    sh 'mvn clean deploy -Dartifactory.user=$ARTIFACTORY_USER -Dartifactory.password=$ARTIFACTORY_PASS'
}
```

---

---

# 🔬 5. Build Fails in CI but Not Locally

**Q: Build passes locally but fails in CI. How do you troubleshoot?**

> **Root cause is almost always:** environment differences — not code differences.

### Investigation Steps

```bash
# Step 1: Read the CI error log carefully
# Common errors: compilation failure, test failure, dependency 404, permission denied

# Step 2: Reproduce in a clean CI-like container
docker run -it --rm openjdk:17-slim bash
# Then run the exact same build commands inside

# Step 3: Compare environments
# Check: OS, JDK version, Node version, Python version, env variables
java -version         # locally vs CI
node --version        # locally vs CI
echo $JAVA_HOME

# Step 4: Check missing secrets/env vars
# CI doesn't have your local .env file or ~/.m2/settings.xml
printenv | grep -i "token\|secret\|key\|password"
# In GitHub Actions: check secrets are wired in env: section

# Step 5: Check file/path issues
# Linux CI is case-sensitive — Import.java vs import.java = different files
# Also check: .gitignore excluding needed files from CI
```

### Common Mismatches Table

| Cause | Local | CI | Fix |
|---|---|---|---|
| Java version | Java 8 | Java 17 | Pin JDK in CI config |
| Dependency cache | `~/.m2` populated | Empty | Add cache step |
| Secrets | In `.env` | Missing | Add to CI secrets |
| File paths | Windows backslash | Linux slash | Normalize paths |
| Permissions | root user | restricted user | `chmod +x script.sh` |

**Real example:**
```
Error: package javax.annotation does not exist
Cause: Local = Java 8 (included javax.annotation), CI = Java 11 (removed it)
Fix: Added explicit dependency in pom.xml + pinned JDK in GitHub Actions
```

### 🎙️ Answer
> *"I start by reading the CI error log to identify the failure type. Then I reproduce it in a clean Docker container matching the CI environment. The most common causes are: different Java/Python/Node versions, missing credentials that exist locally in .env but not in CI secrets, dependency cache being empty in CI, and Linux case-sensitivity for file paths. I fix it by pinning tool versions in CI config, adding cache steps for dependencies, and ensuring all secrets are injected through CI secret management."*

---

---

# 🚨 6. CI Passed but App Broken in Prod

**Q: CI pipeline succeeds but the app is broken in production. What do you do?**

### Immediate Response

```
1. Notify the team (incident channel, PagerDuty)
2. Assess impact — all users or subset? All features or one?
3. Decision: rollback NOW or try to fix forward?
```

### Rollback Options

```bash
# Kubernetes rollback
kubectl rollout undo deployment/myapp
kubectl rollout status deployment/myapp

# Helm rollback
helm rollback myapp 2       # rollback to revision 2

# Git tag-based rollback (redeploy previous version via CI)
git checkout v1.4.2
# trigger deploy pipeline
```

### Root Cause Investigation

```bash
# Step 1: Check prod logs
kubectl logs -l app=myapp --since=30m
journalctl -u myapp --since "30 minutes ago"

# Step 2: Check metrics (Prometheus/Grafana/Datadog)
# Look for: error rate spike, latency increase, memory/CPU anomaly

# Step 3: Compare staging vs production
# Common diff causes:
# - Missing env variables (prod has different config)
# - Feature flags not configured
# - Database schema mismatch (migration didn't run)
# - Secrets rotated in prod but not updated

# Step 4: Check recent infrastructure changes
# Terraform state, Helm chart values, K8s config changes
```

### Prevention (Long-Term Fixes)

```
1. Add smoke tests that run immediately after deployment
2. Use canary releases (5% traffic first → check metrics → expand)
3. Use blue-green deployments (switch only when health checks pass)
4. Enforce staging/prod environment parity (same configs, same secrets structure)
5. Post-deploy monitoring alerts (error rate > 1% → page on-call)
```

**Real example:**
```
Staging: USE_MOCK_SERVICE=true (hardcoded)
Production: USE_MOCK_SERVICE not set (defaulted to true)
Fix: Added proper values-prod.yaml + smoke test that verifies real service connection
```

### 🎙️ Answer
> *"First I assess severity and decide on rollback vs fix-forward — if users are impacted, I rollback immediately with `kubectl rollout undo` or Helm. Then I investigate: check prod logs for errors, compare staging vs prod environment variables and secrets, look at what changed recently (infrastructure, config, dependencies). Common cause is environment drift — something that works in staging fails in prod due to different secrets, feature flags, or config values. Long-term fix: add post-deploy smoke tests and canary releases."*

---

---

# 🐌 7. CI Pipeline Slows Down

**Q: Your CI pipeline is getting slower over time. How do you fix it?**

### Step 1: Measure — Find the Bottleneck

```bash
# GitHub Actions: check step durations in job summary
# Jenkins: Pipeline Stage View plugin shows per-stage timing
# Prometheus: track build duration over time
```

### Step 2: Cache Dependencies

```yaml
# GitHub Actions — cache Maven dependencies
- name: Cache Maven
  uses: actions/cache@v3
  with:
    path: ~/.m2/repository
    key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}

# Cache npm
- uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-npm-${{ hashFiles('package-lock.json') }}
```

### Step 3: Parallelize

```yaml
# GitHub Actions matrix — run tests across multiple versions/OS simultaneously
strategy:
  matrix:
    java: [11, 17]
    os: [ubuntu-latest, windows-latest]
```

```groovy
// Jenkins parallel stages
parallel {
    stage('Unit Tests') { steps { sh 'mvn test -Dtest=Unit*' } }
    stage('Integration Tests') { steps { sh 'mvn test -Dtest=IT*' } }
    stage('Lint') { steps { sh 'pylint src/' } }
}
```

### Step 4: Docker Layer Caching

```dockerfile
# Bad — any change rebuilds from apt install
COPY . .
RUN apt-get install -y python3

# Good — dependencies cached unless requirements.txt changes
COPY requirements.txt .
RUN pip install -r requirements.txt   # ← cached if requirements.txt unchanged
COPY . .                              # ← only code layer rebuilds on code change
```

### Step 5: Runner Maintenance

```bash
# Clean up disk space on runners
docker system prune -af
# Remove old workspaces (Jenkins)
# Scale runners with autoscaling (GitHub Actions self-hosted + KEDA)
```

**Real example:**
```
Pipeline grew from 7 → 19 minutes over 6 months
Root causes: no dependency caching, 600 tests running sequentially, stale Docker layers
Fixes: added Maven cache + parallel test split + Dockerfile optimization
Result: reduced to ~6 minutes (70% reduction)
```

---

---

# 🔕 8. CI Doesn't Trigger on Feature Branch

**Q: Developer pushes to a feature branch — GitHub Actions doesn't trigger. Why?**

### Cause 1: `on:` section doesn't include feature branches

```yaml
# ❌ Wrong — only triggers on main
on:
  push:
    branches:
      - main

# ✅ Fix — include feature branches
on:
  push:
    branches:
      - main
      - 'feature/*'
      - 'fix/*'
      - 'release/*'
```

### Cause 2: `paths:` filter excludes the changed files

```yaml
on:
  push:
    paths:
      - 'src/**'       # Only triggers if files in src/ changed
      # If developer only changed README.md → no trigger
```

### Cause 3: Workflow file doesn't exist on the default branch

> GitHub Actions requires workflow files in `.github/workflows/` on the **default branch** (`main`).
> If the workflow was added only on the feature branch → may not trigger correctly.

```bash
# Fix: merge workflow file to main first, then use from feature branches
```

### Cause 4: Actions permissions disabled

```
GitHub → Repo Settings → Actions → General
→ Allow all actions → Save
```

### Cause 5: YAML syntax error

```bash
# Test locally with 'act'
brew install act
act push

# Or check GitHub Actions → failed workflow → "This workflow has a problem"
```

**Real example:**
```
Developer pushed feature/payment-refactor
on.push.branches: [main, develop]  ← feature/* not included
Fix: added 'feature/*' to branches list
```

### 🎙️ Answer
> *"The first place I check is the `on:` trigger section — does it include the branch pattern? Feature branches often need `'feature/*'` with glob syntax. I also check if a `paths:` filter is excluding the changed files, verify the workflow file exists on the default branch (GitHub requires this), and confirm Actions permissions are enabled in repo settings. If the YAML looks correct, I use `act` locally or GitHub's workflow syntax checker to find errors."*

---

---

# 📥 9. CI Cannot Download Dependencies

**Q: CI build fails because it can't download a dependency. What do you do?**

### Diagnose by Error Code

| Error | Meaning | Action |
|---|---|---|
| `401 Unauthorized` | Wrong/expired credentials | Refresh token or secret |
| `403 Forbidden` | No permission to this repo | Check RBAC in Artifactory |
| `404 Not Found` | Artifact doesn't exist | Check name/version/URL |
| `Connection timeout` | Network/firewall issue | Test with curl |

```bash
# Step 1: Test connectivity to artifact repository
curl -I https://artifactory.example.com/artifactory/libs-release/com/example/app/1.0.0/app-1.0.0.jar
# 200 = accessible | 401 = auth issue | 404 = doesn't exist

# Step 2: Check pom.xml / build.gradle for correct URL
cat pom.xml | grep -A5 "<repository>"

# Step 3: Verify credentials in CI
# Jenkins: credentials store → check token expiry
# GitHub Actions: Settings → Secrets → check secret name matches workflow

# Step 4: Try with clean local cache
rm -rf ~/.m2/repository/com/example/app
mvn clean install -U           # -U forces update check

# Step 5: Check if artifact actually exists in Artifactory UI
# Log into JFrog → browse the path manually
```

**Real example:**
```
Jenkins mvn install fails with 401
Root cause: AWS CodeArtifact token expired (12-hour TTL)
Fix: Added token refresh step at start of Jenkinsfile:
```

```groovy
stage('Auth') {
    steps {
        sh '''
        export CODEARTIFACT_TOKEN=$(aws codeartifact get-authorization-token \
          --domain my-domain --query authorizationToken --output text)
        echo "Refreshed CodeArtifact token"
        '''
    }
}
```

---

---

# 🐍 10. Python Build Fails in CI

**Q: Python build works locally but fails in CI. What could be the issue?**

### Diagnostic Checklist

```yaml
# Step 1: Pin Python version explicitly in CI
- uses: actions/setup-python@v4
  with:
    python-version: '3.10'   # ← never leave this ambiguous

# Step 2: Verify dependencies installed
- run: pip install -r requirements.txt

# Step 3: Run tests verbosely
- run: pytest -v --tb=long    # -v = verbose | --tb=long = full traceback
```

```bash
# Step 4: Check for missing system-level dependencies (C extensions)
# psycopg2, numpy, cryptography often need OS packages:
sudo apt-get install -y libpq-dev libssl-dev python3-dev
pip install psycopg2           # now works

# Step 5: Check secrets — locally in .env, CI needs explicit injection
```

```yaml
# Wrong: local .env not in CI
# Fix: inject secrets via GitHub Actions env:
env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}
  API_KEY: ${{ secrets.API_KEY }}
```

**Real example:**
```
Flask app uses python-dotenv, .env has DATABASE_URL
CI has no .env → database connection fails with None
Fix: Added secrets to GitHub Settings → Secrets + injected via env: in workflow
```

---

---

# 🐍 11. Python Build Process

**Q: Explain the Python application build process.**

### Project Structure

```
myapp/
├── myapp/              # Source code package
│   ├── __init__.py
│   └── app.py
├── tests/              # Unit tests
│   └── test_app.py
├── pyproject.toml      # Modern metadata (PEP 517/518)
├── requirements.txt    # Runtime dependencies
├── requirements-dev.txt # Dev/test dependencies (pytest, flake8)
└── Dockerfile          # Container packaging
```

### Build Steps

```bash
# Step 1: Create virtual environment (isolation)
python3 -m venv venv
source venv/bin/activate     # Linux/Mac
venv\Scripts\activate        # Windows

# Step 2: Install dependencies
pip install -r requirements.txt
pip install -r requirements-dev.txt   # for testing tools

# Step 3: Lint (static analysis)
flake8 myapp/                # style checks
pylint myapp/                # deeper analysis

# Step 4: Run tests
pytest tests/ -v --cov=myapp --cov-report=xml

# Step 5: Build distributable package
pip install build
python -m build
# Output:
# dist/myapp-0.1.0.tar.gz          ← source distribution
# dist/myapp-0.1.0-py3-none-any.whl ← wheel (faster to install)

# Step 6a: Publish to PyPI (public)
pip install twine
twine upload dist/*

# Step 6b: Or package as Docker image (DevOps path)
docker build -t myapp:latest .
docker push registry.example.com/myapp:latest
```

**What is a `.whl` file?**
> A **wheel** is a pre-built, ready-to-install Python package. Installing a wheel is faster than installing from source because compilation is pre-done.

---

---

# 🔍 12. Static Code Analysis Advantages

**Q: Using static code analysis, what kinds of problems can you identify?**

| Category | What's Caught | Tool |
|---|---|---|
| **Syntax errors** | Invalid syntax, undeclared variables | Compiler, `pyflakes` |
| **Style violations** | PEP8, indentation, naming | `flake8`, `pylint`, `eslint` |
| **Complexity** | Deep nesting, methods too long | `radon`, SonarQube |
| **Security vulnerabilities** | Hardcoded credentials, SQL injection, eval() | `bandit`, `semgrep` |
| **Dead code** | Unused imports, unreachable blocks | `pylint`, `vulture` |
| **Type errors** | Wrong type passed to function | `mypy` (Python), TypeScript |
| **Anti-patterns** | `==` vs `is`, resource leaks | `pylint`, PMD |
| **Duplicated code** | Copy-pasted blocks | SonarQube, `jscpd`, PMD |

**Key benefit:** Catches issues **before** the code runs — no runtime required.

```bash
# Run bandit (Python security scanner)
bandit -r myapp/ -f json -o bandit-report.json
# Detects: hardcoded passwords, use of MD5, shell injection, etc.

# Run SonarQube analysis
mvn sonar:sonar \
  -Dsonar.projectKey=myapp \
  -Dsonar.host.url=http://sonarqube:9000 \
  -Dsonar.login=$SONAR_TOKEN
```

**Real example:**
```
bandit → detected hardcoded AWS access key in config.py
flake8 → flagged 3 unused imports
SonarQube → found duplicated validation logic in 4 files
All caught before merging to main ✅
```

### 🎙️ Answer
> *"Static analysis catches bugs without running the code. It identifies syntax errors, style violations, security issues like hardcoded credentials or SQL injection (using bandit/semgrep), dead code, type mismatches, and copy-pasted duplication. The key value in CI is shift-left — finding these at commit time costs 10x less than finding them in production. I use flake8/pylint for style, bandit for security, mypy for types, and SonarQube for comprehensive quality gates."*

---

---

# ⏱️ 13. Static Analysis Slows Down CI

**Q: Static code analysis is making CI pipelines slow. How do you optimize?**

### Optimization Strategies

#### Strategy 1: Analyze only changed files (fastest)

```bash
# Get only changed Python files vs main branch
git diff --name-only origin/main...HEAD | grep '\.py$' | xargs pylint
# Instead of: pylint entire codebase (thousands of files)
```

#### Strategy 2: Run in parallel

```bash
# flake8 with multiple workers
flake8 --jobs=4 myapp/

# GitHub Actions: separate job runs in parallel with build job
```

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps: [...]

  lint:           # runs simultaneously with build
    runs-on: ubuntu-latest
    steps:
      - run: flake8 --jobs=4 myapp/
```

#### Strategy 3: Shift left — pre-commit hooks

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pycqa/flake8
    rev: 6.0.0
    hooks:
      - id: flake8
  - repo: https://github.com/pycqa/bandit
    rev: 1.7.4
    hooks:
      - id: bandit
```

```bash
pip install pre-commit
pre-commit install    # runs linters before every git commit
# Issues caught locally → don't even reach CI
```

#### Strategy 4: Move heavy checks to scheduled jobs

```yaml
# Heavy analysis (SonarQube full scan) runs nightly, not on every PR
on:
  schedule:
    - cron: '0 2 * * *'    # 2 AM daily
```

#### Strategy 5: Cache tool installations

```yaml
- uses: actions/cache@v3
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements-dev.txt') }}
```

**Real example:**
```
pylint across monorepo: 4-5 minutes per build
Fixes:
  1. Changed to git diff (changed files only)
  2. Split into parallel jobs
  3. Added pre-commit hooks
Result: CI lint time → 45 seconds (70% reduction)
```

---

---

# 🔄 14. ArgoCD App Out of Sync

**Q: ArgoCD shows app as OutOfSync, but no Git changes were made. Why?**

> **Key concept:** ArgoCD compares Git (desired state) vs live cluster (actual state). If they differ for **any reason** — not just Git changes — the app shows OutOfSync.

### Root Causes

| Cause | Explanation | Fix |
|---|---|---|
| **Manual kubectl edit** | Someone ran `kubectl edit deployment` or `kubectl scale` | `argocd app sync <app>` + don't use kubectl on ArgoCD-managed resources |
| **HPA scaling replicas** | HPA changed replica count, Git still says `replicas: 2` | Add `ignoreDifferences` for replicas |
| **Secret rotation** | Vault/External Secrets auto-updated a secret | `ignoreDifferences` for secret data |
| **Helm hooks modified resources** | Post-install hooks changed annotations | `ignoreDifferences` for those fields |
| **Sync window expired** | Time-based sync restriction blocked auto-sync | Check `spec.syncPolicy.syncWindows` |

### Fix: ignoreDifferences

```yaml
# application.yaml — tell ArgoCD to ignore replica count drift
spec:
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas         # HPA manages this — don't flag as OutOfSync

    - kind: Secret
      jsonPointers:
        - /data                  # rotated secrets — don't flag as OutOfSync
```

### Manual Sync

```bash
# Check why it's out of sync
argocd app diff myapp

# Force sync from Git
argocd app sync myapp

# Hard refresh (re-fetch from cluster)
argocd app get myapp --hard-refresh
```

### 🎙️ Answer
> *"ArgoCD shows OutOfSync when the live cluster state doesn't match Git — this can happen without Git changes. Common causes: someone ran `kubectl edit` manually, HPA changed the replica count that Git shows as fixed, or a secret was auto-rotated. The fix for manual changes is to sync back from ArgoCD. For legitimate runtime drift like HPA scaling, I add `ignoreDifferences` in the Application spec to tell ArgoCD to stop tracking those specific fields."*

---

---

# 📧 15. Jenkins Email Notification on Failure

**Q: How do you configure Jenkins to send an email when a build fails?**

### Step 1: Install Email Extension Plugin

```
Manage Jenkins → Plugin Manager → Available
→ Search: "Email Extension Plugin" → Install
```

### Step 2: Configure SMTP (Manage Jenkins → Configure System)

```
SMTP Server: smtp.gmail.com
Default user email suffix: @example.com
Use SMTP Authentication: ✅
Username: jenkins@example.com
Password: app-specific-password
Use SSL: ✅
Port: 465
```

### Step 3: Add to Jenkinsfile (Declarative Pipeline)

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
    post {
        failure {
            emailext(
                to: 'dev-team@example.com',
                subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} FAILED",
                body: """
                    Build failed!
                    Job: ${env.JOB_NAME}
                    Build: ${env.BUILD_NUMBER}
                    URL: ${env.BUILD_URL}
                    
                    Check the console output for details.
                """,
                attachLog: true          // attach build log to email
            )
        }
        fixed {
            emailext(
                to: 'dev-team@example.com',
                subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} FIXED ✅",
                body: "Build is back to normal: ${env.BUILD_URL}"
            )
        }
    }
}
```

### Step 4: Trigger Types

| Trigger | When Email Sent |
|---|---|
| `failure` | Build fails |
| `fixed` | Build recovers after failure |
| `unstable` | Tests failed but build succeeded |
| `always` | Every build regardless of result |

**Real-world enhancement — author-specific alerts:**
```groovy
emailext(
    to: '$DEFAULT_RECIPIENTS, $CHANGED_COMMITTER',   // include commit author
    recipientProviders: [[$class: 'RequesterRecipientProvider'],
                         [$class: 'DevelopersRecipientProvider']]
)
```

### 🎙️ Answer
> *"I install the Email Extension Plugin, configure SMTP settings in Manage Jenkins with an app-specific password, then use the `post { failure {} }` block in the Jenkinsfile to send emails. I configure separate triggers for failure, fixed (recovery), and unstable states. In production, I also use `$CHANGED_COMMITTER` to automatically email the developer who broke the build, and attach the log file so they can debug without opening Jenkins."*

---

---

# 📌 Master Cheatsheet

```
╔══════════════════════════════════════════════════════════════════════╗
║            CI/CD INTERVIEW MASTER CHEATSHEET                         ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  JENKINS SHARED LIBRARIES:                                           ║
║  Git repo → vars/ (global functions) + src/ (classes)               ║
║  Register in Manage Jenkins → System → Global Pipeline Libraries     ║
║  Use: @Library('my-lib') _ in Jenkinsfile                           ║
║  Benefit: DRY, versioned, consistent across all pipelines            ║
║                                                                      ║
║  MAVEN LIFECYCLE (order matters):                                    ║
║  validate → compile → test → package → verify → install → deploy    ║
║  clean = delete target/ | package = create JAR/WAR                  ║
║  install = push to local ~/.m2 | deploy = push to artifact repo     ║
║  mvn clean package -DskipTests (CI fast build)                      ║
║                                                                      ║
║  ARTIFACT REPO:                                                      ║
║  JFrog Artifactory = multi-format (Maven/Docker/npm/Helm)           ║
║  settings.xml = credentials | pom.xml = repo URLs                   ║
║  mvn clean deploy = build + upload to Artifactory                   ║
║                                                                      ║
║  BUILD FAILS IN CI NOT LOCALLY:                                      ║
║  docker run -it openjdk:17 bash → reproduce in clean env            ║
║  Compare: Java/Python version, env vars, secrets, file paths        ║
║  CI is case-sensitive Linux | local may be Windows/Mac              ║
║                                                                      ║
║  CI PASSED BUT APP BROKEN IN PROD:                                   ║
║  Rollback: kubectl rollout undo | helm rollback                      ║
║  Root cause: env diff (secrets, config, feature flags, migrations)  ║
║  Prevention: smoke tests + canary releases + env parity             ║
║                                                                      ║
║  CI SLOWNESS:                                                        ║
║  Cache: ~/.m2, ~/.npm, ~/.cache/pip with hashFiles() key            ║
║  Parallel jobs: matrix strategy or Jenkins parallel{}               ║
║  Dockerfile: dependencies layer before source code layer            ║
║  Pre-commit hooks: shift left → catch issues before CI              ║
║                                                                      ║
║  GITHUB ACTIONS NOT TRIGGERING:                                      ║
║  on.push.branches: must include 'feature/*' with glob               ║
║  paths: filter may exclude changed files                             ║
║  Workflow file must exist on default branch (main)                  ║
║                                                                      ║
║  STATIC CODE ANALYSIS:                                               ║
║  bandit=security | flake8=style | pylint=deep | mypy=types          ║
║  SonarQube=comprehensive | semgrep=custom rules                     ║
║  Speed fix: git diff changed files only + parallel jobs             ║
║                                                                      ║
║  ARGOCD OUT OF SYNC:                                                 ║
║  Cause: live state ≠ Git (kubectl edit, HPA scaling, secret rotation)
║  Fix drift: argocd app sync | Fix permanently: ignoreDifferences    ║
║  ignoreDifferences → /spec/replicas for HPA-managed deployments    ║
║                                                                      ║
║  JENKINS EMAIL:                                                      ║
║  Email Extension Plugin → SMTP config → post{failure{emailext()}}  ║
║  Triggers: failure | fixed | unstable | always                      ║
║  $CHANGED_COMMITTER → emails the developer who broke the build      ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 🗣️ Universal Interview Tips

| ✅ DO | ❌ DON'T |
|---|---|
| For "build fails CI not local" → say **"reproduce in Docker first"** | Just say "check the logs" |
| For "CI slow" → mention **caching + parallelism + pre-commit** | Just say "upgrade the runner" |
| For ArgoCD OutOfSync → explain **ignoreDifferences** | Just say "sync it" |
| For email → mention **$CHANGED_COMMITTER** to email the author | Forget the "fixed" recovery trigger |
| For Maven → recite `validate → compile → test → package → install → deploy` in order | List random commands |

---

## 📚 Resources

- 🔗 [Jenkins Shared Libraries Docs](https://www.jenkins.io/doc/book/pipeline/shared-libraries/)
- 🔗 [Maven Lifecycle Reference](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)
- 🔗 [GitHub Actions `on` trigger docs](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs)
- 🔗 [ArgoCD ignoreDifferences](https://argo-cd.readthedocs.io/en/stable/user-guide/diffing/)
- 🔗 [devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide)

---

> ⭐ **Star this repo** if it helped you prepare for your DevOps interview!
> 🔔 Paste the next topic's notes — I'll overwrite with only those!
