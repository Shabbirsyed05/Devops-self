# 🚀 DevOps Fundamentals – Day 1

## 📌 Brief Overview
This note covering DevOps fundamentals was created from the **Day-1 | Fundamentals of DevOps | Free DevOps Course | 45 Days** video.  
It covers definitions, why DevOps matters, core pillars, interview preparation, and a 40-day learning roadmap.

---

## 🔑 Key Points
- What DevOps is and its goal of faster, reliable delivery  
- Traditional delivery pain points and how DevOps solves them  
- The four core pillars: automation, quality, monitoring, testing  
- Day-1 agenda, interview self-introduction framework, and roadmap overview  

---

## 🤖 What is DevOps?
DevOps is a **culture and practice** that organizations adopt to improve their ability to deliver applications (or any software artifact) **quickly, reliably, and with high quality**.

### Key Notes
- DevOps is **not a single tool or technology**
- It is a **way of working**
- The ultimate goal is **faster, repeatable delivery of value to the end user**

---

## 🚀 Why DevOps?
DevOps emerged to address slow, manual, and siloed processes that historically required multiple separate teams such as developers, system administrators, build/release engineers, and testers.

### ❌ Typical Pain Points Before DevOps
- Long lead times (for example, **10 days** to move from code commit to production)
- Manual hand-offs causing errors and delays
- Limited visibility into system health after deployment

### ✅ How DevOps Solves These Problems
1. **Automation** – reduces manual effort  
2. **Quality Assurance** – ensures every change meets standards  
3. **Monitoring & Observability** – detects issues early  
4. **Continuous Testing** – validates changes throughout the pipeline  

---

## 🏭 Traditional Software Delivery (Pre-DevOps)

| Role | Primary Responsibility | Typical Workflow |
|----|----------------------|----------------|
| Developer | Writes code and checks it into a central repository (SVN, CVS, etc.) | Code → Repository |
| System Administrator | Provisions a server (bare-metal, VM, etc.) for the application | Server creation |
| Build & Release Engineer | Deploys the application and promotes builds through environments (staging, pre-prod, prod) | Deploy → Promote |
| Tester | Validates the deployed application on each environment | Test → Feedback |

### Result
- Multiple hand-offs  
- Manual steps  
- Long delivery cycles (days to weeks)  
- **End-to-end time ≈ 10 days**

---

## 📚 Core Pillars of DevOps
1. **Automation** – scripting repeatable tasks such as provisioning, deployments, and rollbacks  
2. **Quality** – code reviews, static analysis, and automated test suites  
3. **Monitoring & Observability** – real-time metrics, logs, and alerts to track system health  
4. **Continuous Testing** – unit, integration, and performance tests run on every change  

> Together, these pillars enable **continuous delivery or continuous deployment**, where releases reach production in hours or minutes instead of days.

---

## ❓ Interview Preparation – Common DevOps Questions

### 1. What is DevOps?
Provide the concise definition explained above.

### 2. Why do we need DevOps?
To shorten delivery cycles, reduce manual errors, and improve reliability.

### 3. Introduce Yourself as a DevOps Engineer
- State your years of DevOps experience (for example, 4–5 years)
- Mention prior roles such as system administrator, build/release engineer, or developer
- Summarize responsibilities: automation, quality enforcement, monitoring, continuous testing

### 4. What does a day-to-day DevOps engineer do?
- Automates infrastructure using IaC tools  
- Maintains CI/CD pipelines  
- Monitors production systems and responds to alerts  
- Writes and maintains automated test suites  
- Collaborates with development and operations teams  

---

## 🧑‍💼 Introducing Yourself as a DevOps Engineer

### Structure
1. **Current role & experience**  
   “I am a DevOps Engineer with X years of experience.”

2. **Background**  
   “Previously, I worked as a system administrator or build-release engineer.”

3. **Key responsibilities**  
   Automation, quality, monitoring, and testing

4. **Tools (optional)**  
   GitHub Actions, Kubernetes, Ansible, Terraform (if asked)

### Tips
- Be truthful about DevOps experience  
- Highlight how past roles add value  
- Keep responses concise and adaptable  

---

## 📆 Day-to-Day Activities of a DevOps Engineer
- Automation using scripts or IaC templates  
- CI/CD pipeline design and troubleshooting  
- Quality checks via static analysis and automated tests  
- Monitoring dashboards, alerts, and log aggregation  
- Incident response and root-cause analysis  
- Collaboration with developers, QA, and operations  

> Mastering these activities shortens delivery time, enhances reliability, and drives continuous improvement.

---

## 🤝 Self-Introduction Framework for a DevOps Role
A concise self-introduction should highlight experience, background, and the four DevOps pillars.

### Structure
1. Current role and years of experience  
2. Relevant background roles  
3. Core responsibilities (automation, quality, monitoring, testing)  
4. Tools you are comfortable with (optional)

---

## 📅 Learning Path: 40-Day DevOps Course Roadmap

| Day Range | Focus Area | Key Outcome |
|--------|-----------|------------|
| Day 1 | Introduction, four pillars, self-intro template | Understand DevOps fundamentals |
| Day 2 | SDLC & DevOps role | Map DevOps to SDLC phases |
| Day 3–5 | Version control, branching, CI pipelines | Reliable source-code workflows |
| Day 6–10 | Infrastructure as Code (IaC) | Automated environment creation |
| Day 11–15 | Containerization & orchestration | Scalable services |
| Day 16–20 | Continuous testing & quality gates | Automated test integration |
| Day 21–25 | Monitoring, logging, alerting | Observability |
| Day 26–30 | Security & compliance | Shift-left security |
| Day 31–35 | Release strategies (blue-green, canary) | Safe deployments |
| Day 36–40 | Capstone project & career prep | End-to-end DevOps pipeline |

---

## 📚 Homework & Community Interaction
- Research alternative DevOps definitions on Google  
- Post questions or insights in comments  
- Share the course with peers interested in DevOps  
- Selected questions may appear in future Q&A sessions  

---

## 🔄 Upcoming Topic Preview: Day 2 – DevOps in SDLC
Day 2 explains how DevOps fits into each SDLC phase, including planning, coding, testing, deployment, operations, and monitoring.

======================================================

# 📚 DevOps & SDLC – Day 2

## 📌 Brief Overview
This note covers **DevOps & SDLC** and was created from the  
**Day-2 | Improve SDLC with DevOps | Free DevOps Course | 45 Days** YouTube video.

It gives a concise walk-through of:
- The **Software Development Life Cycle (SDLC)**
- How **DevOps practices like CI/CD** fit into each phase
- An illustrative **e-commerce case study**
- A quick look at **Agile, Waterfall, and Iterative** project models

---

## 🔑 Key Points
- Overview of SDLC phases and their objectives  
- Integration of DevOps automation for building, testing, and deployment  
- Real-world example of adding a kids catalog to an e-commerce site  
- Snapshot of common project management methodologies in software delivery  

---

## 📚 Recap of Day 1
DevOps overview – definition, reasons for adopting DevOps, and how to present
oneself in a DevOps interview.

---

## 🔄 Day 2 – Software Development Life Cycle (SDLC)

### 📌 What Is SDLC?
**Software Development Life Cycle (SDLC)** is a standardized process used by the
software industry to design, develop, and test a high-quality product before delivery.

---

### 🎯 Why SDLC Matters for All Engineers
- Provides a common framework across startups, MNCs, and unicorns  
- Ensures consistent quality and customer satisfaction  
- Forms the baseline that DevOps engineers automate to improve delivery speed  

---

## 🧱 Core Phases of the SDLC

| Phase | Main Activities | DevOps Automation Focus |
|------|----------------|------------------------|
| **Planning & Requirements** | • Gather stakeholder & customer feedback<br>• Evaluate feasibility of new features (e.g., kids catalog)<br>• Document needs | – |
| **Defining** | • Create **Software Requirement Specification (SRS)** document<br>• Detail functional & non-functional requirements | – |
| **Designing** | • **High-Level Design (HLD)** – architecture, scalability, availability<br>• **Low-Level Design (LLD)** – module functions, database choices, API contracts | – |
| **Building (Development)** | • Write code according to designs<br>• Commit to a source-code repository (e.g., Git) | **Continuous Integration (CI)** – automated builds, code reviews, merge checks |
| **Testing** | • Deploy code to a test environment<br>• Quality Assurance (QA) engineers execute functional, performance, and regression tests | **Continuous Testing** – automated test suites, test result reporting |
| **Deployment** | • Promote verified build to production<br>• Deliver the feature to end users | **Continuous Deployment (CD)** – automated release pipelines, infrastructure provisioning |

---

## 🔄 Circular Nature of SDLC
After a feature is deployed, the cycle restarts for the next feature or enhancement  
(e.g., adding a new catalog section).

---

## 📂 Example Scenario – “example.com” E-Commerce Site

1. **Idea Generation** – Decision to add a kids catalog based on market research.  
2. **Planning & Requirements** – Business analyst collects data (e.g., interest from 6–12-year-old segment).  
3. **Defining** – Write an SRS documenting the collected metrics.  
4. **Designing**  
   - **HLD** – Ensure scalability for seasonal spikes, decide on database replication  
   - **LLD** – Choose specific technologies (e.g., Java/Python services, MySQL)  
5. **Building** – Developers code the feature, push commits to Git.  
6. **Testing** – QA team runs automated and manual tests on a staging server.  
7. **Deployment** – Feature is released to production; customers can now shop for kids’ clothing.

---

## 🤖 DevOps Engineer’s Role
- Automate the building, testing, and deployment phases to accelerate delivery and reduce manual errors  
- Evaluate tools (e.g., Terraform, Ansible) for fit within the organization; recommend or reject based on specific needs  
- Collaborate with product owners, business analysts, and developers to ensure the pipeline aligns with business goals  

---

## 📈 Project Management Models (Brief Mention)
- **Waterfall** – linear, sequential phases  
- **Iterative** – repeated cycles with incremental improvements  
- **Agile** – short sprints; phases are executed in small, fast increments  

Most modern organizations adopt **Agile** to apply SDLC phases continuously.

===============================================

# 📚 DevOps Foundations 🚀

---

## 📝 Brief Overview
> This note covers DevOps fundamentals and was created from the Absolute Prerequisite  
> for Learning DevOps YouTube video. It dives into the essential roles in an organization,  
> walks through the Software Development Life Cycle, explains Scrum practices, and  
> demonstrates how Jira is used to manage work from epics to deployment.

---

## ⭐ Key Points
- ✅ Understand the main roles that collaborate on a DevOps team.
- 🔄 Follow the SDLC phases from planning to maintenance.
- 🏃 Apply Scrum sprint practices and Jira workflows to track progress.
- ⚙️ Grasp the responsibilities of a DevOps engineer in infrastructure and automation.

---

## 🏢 Roles in an Organization

### 🧑‍💼 Business Analyst (BA)
**Definition:** The person who interacts with customers, gathers feedback, and translates  
it into a Business Requirements Document (BRD).
- Collects customer feedback (e.g., 15-minute grocery delivery request).
- Prepares BRD for each requirement.
- Acts as the bridge between customers and internal teams.

---

### 📊 Product Manager (PM)
**Definition:** Sets product vision, prioritizes requirements, and decides which items  
move forward in the roadmap.
- Receives BRDs from BAs.
- Prioritizes items based on market, competition, and organizational capacity.
- Communicates priorities to the Product Owner.

---

### 🧩 Product Owner (PO)
**Definition:** Transforms prioritized requirements into actionable items (epics/features)  
and maintains the product backlog.
- Breaks high-level requirements into epics (or features).
- Writes acceptance criteria / definition of done for each epic.
- Collaborates with Solution Architects to confirm feasibility.

---

### 🏗 Solution / Software Architect
**Definition:** Designs the technical solution, producing High-Level Design (HLD) and  
Low-Level Design (LLD) documents.
- Reviews epics from PO.
- Determines required infrastructure, platforms, and technologies.
- Flags skill gaps or resource constraints back to the PM.

---

### 👨‍💻 Development Lead & Developers
- Receive HLD/LLD and break work into stories.
- Identify needed infrastructure (e.g., Kubernetes clusters, Docker images, Git repos).
- Implement code and coordinate with DevOps for environment provisioning.

---

### ⚙️ DevOps Engineer
**Definition:** Provides the infrastructure and automation required for development,  
testing, and deployment.
- Sets up Kubernetes clusters, Dockerfiles, Git repositories, CI/CD pipelines.
- Integrates security scans and test automation into pipelines.
- Works with developers to ensure required environments are available.
- Looks for process improvements to shorten cycle time (e.g., automating manual QA steps).

---

### 🧪 QA Engineer
- Writes manual or automated tests.
- Works with DevOps to embed tests in CI/CD pipelines.
- Validates that acceptance criteria are met.

---

### 🗄 Database Administrator (DBA)
- Provisions and manages databases required by the solution.
- May collaborate with DevOps for infrastructure as code.

---

### 🚨 Site Reliability Engineer (SRE)
**Definition:** Ensures reliability and availability of the released product.
- Creates dashboards, metrics, and alerting.
- Monitors production health and notifies stakeholders of incidents.

---

### ✍️ Technical Writer
**Definition:** Documents new features, architecture, and user guides.
- Produces documentation for internal teams and external users.
- May serve multiple scrum teams simultaneously.

---

## 📈 Software Development Life Cycle (SDLC) Phases

| Phase | Primary Activities | Key Participants |
|------|------------------|----------------|
| Planning | Gather requirements (BRD) | Business Analyst, PM |
| Analysis | Evaluate feasibility, prioritize | PM, PO |
| Design | Create HLD & LLD | Solution Architect |
| Implementation | Develop code, provision infrastructure | Developers, DevOps, DBA |
| Testing & Integration | Execute test suites, integrate components | QA, DevOps (CI/CD) |
| Deployment & Release | Deploy to production, monitor | DevOps, SRE |
| Maintenance | Incident handling, updates | SRE, DevOps, Developers |

---

## 🔄 Requirement Flow Overview
1. Customer → Business Analyst – feedback captured.
2. BA → Product Manager – BRD handed over.
3. PM → Product Owner – prioritized items.
4. PO → Solution Architect – epics broken into actionable items, designs produced.
5. Solution Architect → Development Lead – HLD/LLD delivered.
6. Development Team – identifies infrastructure needs → DevOps Engineer.
7. DevOps – builds environments, CI/CD pipelines, automates testing.
8. QA & SRE – test, monitor, and maintain the product.

> All parties collaborate within a scrum team to deliver the feature.

---

## 🏃 Scrum Framework & Sprint
- **Scrum Team composition:** Developers, DevOps, QA, DBA, (optional) Technical Writer.
- **Sprint:** Fixed-length iteration (2–3 weeks) where a set of stories is completed.

### Sprint Activities
1. Sprint Planning – select stories from the backlog.
2. Daily Stand-up – quick status updates.
3. Sprint Review – demonstrate completed work.
4. Sprint Retrospective – discuss improvements for the next sprint.

Progress tracked via Jira boards  
🟦 To-Do → 🟨 In Progress → 🟪 Review → 🟩 Done

---

## 📋 Jira Project Management Tool

### Creating an Account & Organization
- Sign up on Atlassian using a Google account.
- Choose a work email (can be personal for learning).
- Create an organization (e.g., AmazonDemoVala).

### Setting Up a Project
- Within the organization, create a project (e.g., AmazonFresh).
- Jira automatically provisions a board for the project.

---

## 📊 Jira Workflow for DevOps Engineers
- Open the specific story in Jira.
- Add daily comments describing work completed.
- Update status (To-Do → In Progress → Review → Done).

> All team members and stakeholders can see real-time updates, facilitating transparency and quick issue escalation.

---

## 📘 Learning Resources
- Atlassian’s **“Learn Jira in 5 minutes”** tutorial (available on the Atlassian website).


===============================================================



