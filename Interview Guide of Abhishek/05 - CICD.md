# CI/CD Interview Guide

> Source: [iam-veeramalla/devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide) — `05-cicd/` folder

---

# Table of Contents

1. [Jenkins Shared Libraries](#1-jenkins-shared-libraries)
2. [Popular Maven Build Targets](#2-popular-maven-build-targets)
3. [Artifact Repository](#3-artifact-repository)
4. [Configure JFrog in Maven](#4-configure-jfrog-in-maven)
5. [Build Fails in CI but Not Locally](#5-build-fails-in-ci-but-not-locally)
6. [CI Passed but App Broken in Prod](#6-ci-passed-but-app-broken-in-prod)
7. [CI Pipeline Slows Down](#7-ci-pipeline-slows-down)
8. [CI Does Not Trigger on Feature Branch](#8-ci-does-not-trigger-on-feature-branch)
9. [CI Cannot Download Dependencies](#9-ci-cannot-download-dependencies)
10. [Python Build Fails in CI](#10-python-build-fails-in-ci)
11. [Python Build Process](#11-python-build-process)
12. [Static Code Analysis Advantages](#12-static-code-analysis-advantages)
13. [Static Code Analysis Slows Down CI](#13-static-code-analysis-slows-down-ci)
14. [ArgoCD App Out of Sync](#14-argocd-app-out-of-sync)
15. [Jenkins Send Notification on Build Failure](#15-jenkins-send-notification-on-build-failure)

---

# 1. Jenkins Shared Libraries

## Question
What are Jenkins Shared Libraries and how do they work?

### 📝 Short Explanation
Jenkins Shared Libraries allow you to **centralize reusable pipeline code** across multiple pipelines. They promote **code reuse**, **maintainability**, and **consistency**.

## ✅ Answer

A **Shared Library** is a Git repository containing reusable Groovy code you can include in Jenkins pipelines using the `@Library` annotation.

Structure:
```
(root)
├── vars/
│   └── sayHello.groovy
├── src/
│   └── org/example/MyClass.groovy
├── resources/
│   └── templates/config.xml
└── README.md
```

### ⚙️ How They Work:
1. **Create a Git repo** with `vars/`, `src/`, etc.
2. Configure in Jenkins: **Manage Jenkins → Global Pipeline Libraries**.
3. In Jenkinsfile:
   ```groovy
   @Library('my-shared-library') _
   ```
4. Use global functions from `vars/` or `src/`.

### 🔍 Example

**vars/sayHello.groovy**
```groovy
def call(String name = 'world') {
    echo "Hello, ${name}!"
}
```

**Jenkinsfile**
```groovy
@Library('my-shared-library') _

pipeline {
    agent any
    stages {
        stage('Greet') {
            steps {
                sayHello('Abhishek')
            }
        }
    }
}
```

### ✅ Benefits
- DRY (Don't Repeat Yourself)
- Cleaner Jenkinsfiles
- Version-controlled and auditable
- Easier team collaboration

> Summary:
> Jenkins Shared Libraries allow you to **modularize and reuse pipeline logic** across projects.

---

# 2. Popular Maven Build Targets

## Question
Talk about 5 build targets that you use on a day-to-day basis in Maven.

## ✅ Answer

### 1. `mvn clean`
- **Purpose:** Deletes the `target/` directory to ensure a clean slate.
- **When:** Before any fresh build.

### 2. `mvn compile`
- **Purpose:** Compiles the source code in `src/main/java`.
- **When:** To verify no compilation issues before packaging.

### 3. `mvn test`
- **Purpose:** Runs unit tests using JUnit or TestNG.
- **When:** Regularly during development or in CI pipelines.

### 4. `mvn package`
- **Purpose:** Packages compiled code into `.jar` or `.war`.
- **When:** When the build is stable and you need an artifact.

### 5. `mvn install`
- **Purpose:** Installs the built artifact into the **local Maven repository** (`~/.m2/repository`).
- **When:** Developing shared libraries or modules reused by other internal projects.

> Summary:
> Most frequently used Maven targets: `clean`, `compile`, `test`, `package`, and `install`.

---

# 3. Artifact Repository

## Question
Which artifact repository do you use for builds?

## ✅ Answer

We use **JFrog Artifactory** as our primary artifact repository.

### 📘 Why JFrog Artifactory?
- **Supports multiple package formats** (Maven, npm, Docker, PyPI, Helm, etc.)
- **Integration with Jenkins & GitHub Actions**
- **Proxying public repositories** like Maven Central or Docker Hub
- **Access control and security** via RBAC
- **Retention policies** to clean up outdated builds

### 🔄 Typical Workflow
1. Developer commits code to GitHub.
2. Jenkins triggers a build and packages using Maven.
3. Artifact pushed to Artifactory using `mvn deploy`.
4. Later stages pull the artifact from Artifactory.

### 🧠 Alternatives:
- **Sonatype Nexus Repository**
- **AWS CodeArtifact**
- **GitHub Packages**

---

# 4. Configure JFrog in Maven

## Question
How do you configure Artifactory for your application in Maven?

## ✅ Answer

### ⚙️ Step 1: Update `settings.xml`

Located at: `~/.m2/settings.xml`
```xml
<settings>
  <servers>
    <server>
      <id>artifactory</id>
      <username>my-artifactory-user</username>
      <password>my-artifactory-password</password>
    </server>
  </servers>
</settings>
```

### ⚙️ Step 2: Configure `pom.xml`
```xml
<distributionManagement>
  <repository>
    <id>artifactory</id>
    <name>Company Release Repo</name>
    <url>https://artifactory.example.com/artifactory/libs-release-local</url>
  </repository>
  <snapshotRepository>
    <id>artifactory</id>
    <name>Company Snapshot Repo</name>
    <url>https://artifactory.example.com/artifactory/libs-snapshot-local</url>
  </snapshotRepository>
</distributionManagement>
```

### 🚀 Deploy Command
```bash
mvn clean deploy
```

> Summary:
> Configure `settings.xml` with credentials and `pom.xml` with repository URLs for secure, automated artifact publishing.

---

# 5. Build Fails in CI but Not Locally

## Question
Build Passed Locally but Fails in CI — How Will You Troubleshoot?

## ✅ Answer

### 🧭 Step-by-Step:

1. **Check the Error Log in CI** — Identify compilation error, test failure, dependency issue, or permission problem.

2. **Reproduce Locally in CI-Like Environment**
   ```bash
   docker run -it --rm openjdk:17 bash
   ```

3. **Compare Local and CI Environments** — OS version, language versions, env variables, build tool versions, memory/CPU limits.

4. **Verify Secrets and Tokens** — Missing Git credentials, API tokens, `.npmrc` or `.pypirc` config files.

5. **Dependency Issues** — Check if build pulls from a local cache (`~/.m2/repository`) that CI doesn't have.

6. **File/Path Issues** — Case sensitivity, permissions, and missing files in Linux-based CI runners.

### 🧠 Example
Build failed with `error: package javax.annotation does not exist`. Local had Java 8, CI had Java 11. Fix: Added `javax.annotation` as explicit dependency and pinned JDK version.

> Summary:
> Start with CI logs → Reproduce in clean container → Compare environments → Check dependencies/secrets → Fix and make build reproducible.

---

# 6. CI Passed but App Broken in Prod

## Question
CI Pipeline Succeeds but App is Broken in Prod — What Action Will You Take?

## ✅ Answer

### 🔍 Step-by-Step:

1. **Initiate Incident Response** — Notify team. Consider automated rollback.
2. **Check Prod Logs and Monitoring** — HTTP status codes, application logs, metrics.
3. **Compare Staging and Production** — Missing env variables, backend services, feature flags?
4. **Check Secrets and External Integrations** — Misconfigured or rotated tokens.
5. **Check Infrastructure Differences** — Different K8s namespace, LB config, Terraform state.

### ✅ Immediate Actions
- Rollback if feasible (Git tags, Helm chart versions, AMI snapshots).
- Create a postmortem entry and assign RCA.

### 🛠️ Preventive Measures
- Add **automated smoke tests** after deployment.
- Integrate **canary releases** or **blue-green deployments**.
- Enforce staging and production **parity**.
- Enable alerting for anomalies post-deployment.

### 🧠 Example
Staging-only env variable `USE_MOCK_SERVICE=true` was hardcoded and not overridden in production. Fix: Added proper `values-prod.yaml` for Helm + smoke tests.

---

# 7. CI Pipeline Slows Down

## Question
Pipeline Slows Down Over Time — How Will You Fix?

## ✅ Answer

### 🧭 Steps:

1. **Measure Stage Durations** — Use CI metrics or Prometheus/Grafana.
2. **Check Dependency Management** — Use `mvn dependency:analyze`, `npm prune`. Cache dependencies between runs.
3. **Enable Incremental Builds** — Gradle `--build-cache`, Docker layer caching.
4. **Clean Up Disk and Workspace** — `docker system prune -af`
5. **Use Parallelism and Matrix Builds** — Split test suites across matrix jobs.
6. **Review Runner Resources** — Check CPU, memory, disk I/O. Use autoscaling runners.

### 🧠 Example
Build time increased from 7 to 19 minutes over 6 months. Causes: Increased tests without parallelism + outdated Docker layers. Fixes: Parallel test stages + Dockerfile cache optimization + weekly runner cleanup. **Result: ~70% reduction.**

---

# 8. CI Does Not Trigger on Feature Branch

## Question
A developer pushes a feature branch, but the pipeline doesn't trigger in GitHub Actions. What could be wrong?

## ✅ Answer

### 🧭 Troubleshooting:

1. **Check `on:` Section** — Ensure `push` includes feature branches:
   ```yaml
   on:
     push:
       branches:
         - main
         - 'feature/*'
   ```

2. **Check `paths` Filter** — If `paths:` is used, changes outside listed paths won't trigger.

3. **Workflow File on Default Branch** — Workflows in `.github/workflows/` must exist on `main` to trigger from other branches.

4. **Branch Protection or Permissions** — Check Settings → Actions → General.

5. **Verify YAML Validity** — Run `act` locally or check GitHub Actions logs for syntax errors.

### 🧠 Example
Developer pushed to `feature/payment-refactor`, `on.push.branches` only had `main` and `develop`. Adding `'feature/*'` fixed it.

---

# 9. CI Cannot Download Dependencies

## Question
Build fails because it can't download a dependency from your artifact repository. What will you do?

## ✅ Answer

### 🧭 Steps:

1. **Check Error Message** — `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, or timeout.

2. **Verify Repository Configuration** — Check `pom.xml`, `build.gradle`, or `.npmrc` for correct repo URL.

3. **Check Credentials** — Verify `settings.xml` or environment variables aren't expired.

4. **Test Repo Connectivity**
   ```bash
   curl -I https://artifactory.example.com/artifactory/libs-release/...
   ```

5. **Check If Artifact Exists** — Log into artifact repository UI and confirm.

6. **Try Local Build with Clean Cache**
   ```bash
   rm -rf ~/.m2/repository/<group>/<artifact>
   mvn clean install
   ```

### 🧠 Example
Jenkins `mvn install` failed because **CodeArtifact token had expired**. Fix: Added token refresh step in Jenkinsfile before build.

---

# 10. Python Build Fails in CI

## Question
Python Build Fails on CI But Works Locally — What Can Be the Issue?

## ✅ Answer

### 🧭 Steps:

1. **Check Python Version** — Pin in CI:
   ```yaml
   - uses: actions/setup-python@v4
     with:
       python-version: '3.10'
   ```

2. **Check Dependency Installation** — `pip install -r requirements.txt`

3. **Check Missing Files/Modules** — CI starts clean. Ensure all required files are in Git.

4. **Check for Missing Secrets** — Locally in `.env`, CI needs them as env variables:
   ```yaml
   env:
     API_KEY: ${{ secrets.API_KEY }}
   ```

5. **Run Tests Verbosely** — `pytest -v`

6. **Check C Extension Issues** — Some packages need system-level dependencies:
   ```bash
   sudo apt-get install -y libpq-dev
   pip install psycopg2
   ```

### 🧠 Example
Flask app used `python-dotenv` but `.env` wasn't in CI. Fix: Added secrets to GitHub Settings and injected via `env:`.

---

# 11. Python Build Process

## Question
Explain the Python Application Build Process in Detail.

## ✅ Answer

### 🧭 Steps:

#### 1. 🗂️ Organize the Project
```
myapp/
├── myapp/              # Source code
│   └── __init__.py
├── tests/              # Unit tests
├── pyproject.toml      # Modern metadata
├── requirements.txt    # Dependencies
└── README.md
```

#### 2. 📦 Declare Dependencies
```bash
pip install -r requirements.txt
```

#### 3. 🛠️ Build the Package
```bash
python3 -m venv venv
source venv/bin/activate
pip install build
python -m build
```
Generates: `dist/myapp-0.1.0.tar.gz` (source) and `dist/myapp-0.1.0-py3-none-any.whl` (wheel)

#### 4. 🧪 Run Unit Tests
```bash
pytest tests/
```

#### 5. 🚀 Distribute or Deploy
```bash
pip install twine
twine upload dist/*
```
Or deploy into containers, VMs, or PaaS.

> Summary:
> Python builds involve organizing code, managing dependencies, building wheels/sdists, and optionally publishing to PyPI or packaging into Docker images.

---

# 12. Static Code Analysis Advantages

## Question
Using Static Code Analysis, what kind of problems can you identify?

## ✅ Answer

### 🔍 Types of Problems:

1. **Syntax Errors and Language Misuse** — Invalid syntax, undeclared variables
2. **Coding Standard Violations** — PEP8, PSR-12. Tools: `pylint`, `flake8`, `eslint`
3. **Code Complexity** — Deep nesting, long methods. Tools: `radon`, `sonarqube`
4. **Security Vulnerabilities** — Hardcoded credentials, SQL injection. Tools: `bandit`, `semgrep`
5. **Dead Code and Unused Variables** — Unused imports, unreachable code
6. **Incorrect Type Usage** — Type mismatches. Tools: `mypy`
7. **Common Bugs and Anti-Patterns** — `==` vs `===`, resource leaks
8. **Duplicate Code** — Copy-pasted blocks. Tools: `SonarQube`, `PMD`, `jscpd`

### 🧠 Example
`bandit` detected a hardcoded AWS access key. `flake8` flagged missing docstrings. `SonarQube` highlighted duplicated logic. All caught **before** staging.

> Summary:
> Static analysis identifies bugs, security risks, code smells, and style issues early — without running the application.

---

# 13. Static Code Analysis Slows Down CI

## Question
Static Code Analysis Slows Down CI Pipeline — How Will You Fix It?

## ✅ Answer

### 🧭 Optimization Strategy:

1. **Run Analysis Only on Changed Files**
   ```bash
   git diff --name-only origin/main...HEAD | grep '\.py$' | xargs pylint
   ```

2. **Run Analysis in Parallel**
   ```bash
   flake8 --jobs=4
   ```

3. **Shift Left: Run Pre-CI** — Use `pre-commit` hooks.

4. **Run Heavy Checks on a Schedule**
   ```yaml
   on:
     schedule:
       - cron: '0 2 * * *'
   ```

5. **Tune Rules and Severity** — Focus on high-impact rules in CI.

6. **Cache Tool Dependencies**
   ```yaml
   - uses: actions/cache@v3
     with:
       path: ~/.cache/pip
       key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
   ```

### 🧠 Example
`pylint` across monorepo took 4-5 min/build. Fixes: Limited to `git diff` changes + split linters into separate jobs + pre-commit hooks. **Result: ~70% reduction.**

---

# 14. ArgoCD App Out of Sync

## Question
App in 'OutOfSync' State in Argo CD, But No Git Changes — What Could Be the Reason?

## ✅ Answer

### 🧭 Common Reasons:

1. **Manual Changes in Cluster** — Someone used `kubectl edit` or `kubectl patch`.
   Fix: `argocd app sync <app-name>`

2. **Dynamic Changes Not Tracked** — Replica counts via HPA, annotations.
   Fix: Use `ignoreDifferences`:
   ```yaml
   spec:
     ignoreDifferences:
       - group: apps
         kind: Deployment
         jsonPointers:
           - /spec/replicas
   ```

3. **Secrets Automatically Rotated** — Sealed Secrets, External Secrets, Vault.
   Fix: `ignoreDifferences` for secret fields.

4. **Sync Window Missed** — Time-based restrictions on auto-sync.

5. **CRDs or Hooks Trigger Drift** — Helm post-install hooks modify resources.

### 🧠 Example
Engineer manually increased replica count. Fix: Re-synced from Argo CD + added `ignoreDifferences` for replica count.

> Summary:
> Argo CD marks apps `OutOfSync` when live state differs from Git — not just when Git changes. Manual changes, runtime drift, or auto-mutations can cause this.

---

# 15. Jenkins Send Notification on Build Failure

## Question
When a build fails in Jenkins, how will you send an email?

## ✅ Answer

### 🧭 Steps:

#### 1. 🧩 Install Email Extension Plugin
- **Manage Jenkins → Plugin Manager → Available** → Search `Email Extension Plugin`

#### 2. ⚙️ Global Configuration
- **Manage Jenkins → Configure System**
- SMTP details:
  ```text
  SMTP Server: smtp.gmail.com
  Use SSL: true
  Port: 465
  ```
- Set authentication (username/password or app token)

#### 3. 📤 Configure Project
- **Post-build Actions → Editable Email Notification**
- Recipient List: `dev-team@example.com`
- Triggers: **Failure - Send email on build failure**
- Subject: `$PROJECT_NAME - Build #$BUILD_NUMBER - FAILED!`
- Body:
  ```groovy
  Build failed at $BUILD_URL
  Triggered by: $CAUSE
  ```

#### 4. 🧪 Testing
Trigger a dummy failure and confirm emails are received.

### 🧠 Real-World Example
We used Email Extension Plugin with triggers for `FAILURE`, `UNSTABLE`, and `FIXED`. Custom HTML templates included links to logs and commit diffs. Author-specific alerts using `git commit --author`.

> Summary:
> Install Email Extension Plugin → Set SMTP details → Enable email triggers in job config → Customize templates.

---
