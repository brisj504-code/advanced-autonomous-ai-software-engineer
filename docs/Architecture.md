# System Architecture for Project Aether

## 1. Introduction
This document details the high-level architecture of Project Aether using the [C4 model](https://c4model.com/). The C4 model is used to visualize software architecture at different levels of detail, providing a clear and understandable blueprint for development.

This document covers the first two levels:
- **Level 1: System Context Diagram:** Shows how the Aether system fits into its operating environment, including key users and external system dependencies.
- **Level 2: Container Diagram:** Zooms into the Aether system to show its major internal components, which are the specialized sub-agents and core supporting services.

---

## 2. C4 Model - Level 1: System Context Diagram

The System Context diagram shows the Aether system as a single black box, interacting with its users and the external systems it depends on to function.

### Diagram
```
                               +-----------------------------+
                               |   Large Language Model      |
                               |   (e.g., OpenAI, Anthropic) |
                               +-------------^---------------+
                                             |
                                             | Uses for code generation,
                                             | reasoning, analysis
                                             |
+---------------------+        +-------------v---------------+        +-----------------------------+
| Project Stakeholder |------->|                             |------->|  Version Control System     |
| (Human)             |        |        Aether System        |        |  (e.g., GitHub, GitLab)     |
+---------------------+        |                             |        +-----------------------------+
  Provides high-level          +-------------^---------------+        Manages source code repos
  project requirements                       |
                                             |
                                             | Deploys/manages apps and its
                                             | own infrastructure via IaC
                                             |
                               +-------------v---------------+        +-----------------------------+
                               |                             |------->| Public Documentation / Web  |
                               |      Cloud Provider         |        | (e.g., Docs, Stack Overflow)|
                               |      (e.g., AWS, GCP)       |        +-----------------------------+
                               +-----------------------------+        Parses for learning new tools


```

### Components

- **Aether System (This System):** The autonomous AI software engineering agent being designed. It takes high-level requirements and executes the full software development lifecycle.

- **Project Stakeholder (Actor):** The human user who initiates a project by providing the high-level goals and requirements to the Aether system. After this initial hand-off, the stakeholder's role is primarily to observe, not to intervene.

- **Large Language Model (External System):** A foundational dependency. Aether uses external, state-of-the-art LLMs for its core cognitive tasks: planning, reasoning, code generation, bug analysis, and documentation writing.

- **Version Control System (External System):** Aether interacts with VCS platforms like GitHub or GitLab to check out code, manage branches, and submit pull requests for the projects it works on.

- **Cloud Provider (External System):** Aether uses cloud provider APIs to provision and manage the infrastructure required for the applications it builds, using Infrastructure as Code (IaC) scripts.

- **Public Documentation / Web (External System):** Aether has the capability to browse and parse technical documentation from the web to learn how to use new tools, libraries, and APIs.

---

## 3. C4 Model - Level 2: Container Diagram

The Container diagram zooms inside the Aether system boundary to show the main logical components (containers). In the context of Aether, a "container" is a separately deployable and runnable component, which maps directly to the specialized sub-agents and core services. All these containers run within a Kubernetes cluster.

### Diagram
```
+--------------------------------------------------------------------------------------------------+
| Aether System (Boundary)                                                                         |
|                                                                                                  |
|    +------------------------+      +------------------------+      +--------------------------+    |
|    |     ArchitectAgent     |<---->|      CoderAgent        |<---->|        TestAgent         |    |
|    | (Designs architecture) |      | (Implements features)  |      | (Generates & runs tests) |    |
|    +-----------^------------+      +-----------^------------+      +------------^-------------+    |
|                |                           |                           |                       |
|                |                           |                           |                       |
|    +-----------v------------+      +-----------v------------+      +------------v-------------+    |
|    |      SecurityAgent     |<---->|       DevOpsAgent      |<---->|    DocumentationAgent    |    |
|    | (Scans for vulns)      |      | (Manages CI/CD, IaC)   |      | (Generates docs)         |    |
|    +------------------------+      +------------------------+      +--------------------------+    |
|                ^                           ^                           ^                       |
|                |                           |                           |                       |
|                +---------------------------+---------------------------+                       |
|                                            |                                                   |
|                                            | All agents read/write via gRPC                    |
|                                            v                                                   |
| +----------------------------------------------------------------------------------------------+ |
| |                                     Core Services                                            | |
| |                                                                                              | |
| |  +--------------------------+      +-----------------------------+      +------------------+ | |
| |  | Planning/Reasoning Engine| <--> | Persistent Memory (Postgres)| <--> |  Secrets Vault   | | |
| |  +--------------------------+      +-----------------------------+      +------------------+ | |
| |                                                                                              | |
| +----------------------------------------------------------------------------------------------+ |
+--------------------------------------------------------------------------------------------------+

```

### Containers (Sub-Agents)

- **ArchitectAgent:**
  - **Responsibilities:** Interprets initial project requirements, selects the technology stack, designs the high-level system architecture, and defines the tasks for the CoderAgent.
  - **Technology:** Go, gRPC.

- **CoderAgent:**
  - **Responsibilities:** Receives tasks from the ArchitectAgent and implements them by writing clean, high-quality, idiomatic code. Follows TDD principles by collaborating with the TestAgent.
  - **Technology:** Go, gRPC.

- **TestAgent:**
  - **Responsibilities:** Generates and maintains unit, integration, and E2E tests. Executes tests and reports failures. Works with the CoderAgent in a TDD/BDD cycle.
  - **Technology:** Go, gRPC.

- **SecurityAgent:**
  - **Responsibilities:** A specialized DevSecOps agent. Continuously scans source code, dependencies, and infrastructure for vulnerabilities (SAST, SCA). Can autonomously apply security patches.
  - **Technology:** Go, gRPC, security scanning tools (e.g., CodeQL, Snyk integration).

- **DevOpsAgent:**
  - **Responsibilities:** Manages the entire CI/CD pipeline. Provisions infrastructure using Terraform. Manages the containerized deployment using Kubernetes.
  - **Technology:** Go, gRPC, Terraform, Kubernetes CLI.

- **DocumentationAgent:**
  - **Responsibilities:** Generates and maintains project documentation in real-time. Creates "living documentation" from source code comments (e.g., OpenAPI specs) and architectural records.
  - **Technology:** Go, gRPC.

### Containers (Core Services)

- **Planning/Reasoning Engine:**
  - **Responsibilities:** The central orchestrator. Decomposes goals, explores solution paths, and adapts plans based on feedback from other agents. All agents report to and receive high-level direction from this engine.
  - **Technology:** Go, gRPC.

- **Persistent Memory (PostgreSQL + pgvector):**
  - **Responsibilities:** The long-term memory of the system. Stores and indexes all project artifacts (code, docs, logs, requirements) in a vectorized knowledge base to provide context for all LLM queries (RAG).
  - **Technology:** PostgreSQL with `pgvector` extension.

- **Secrets Vault:**
  - **Responsibilities:** Securely stores and manages all sensitive credentials, such as API keys and database passwords.
  - **Technology:** A dedicated secrets management solution (e.g., HashiCorp Vault or a cloud provider's equivalent).
