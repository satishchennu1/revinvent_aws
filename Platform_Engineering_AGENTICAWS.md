#  AWS re:Invent 2025 - Building agentic AI platform engineering solutions with open source (OPN303)

        https://www.youtube.com/watch?v=xwzInf90iUc


        
        Here's a full synopsis and deeper breakdown of the session:

---

                              ## OPN303 — Full Synopsis & Deep Dive
                              
                              ### 🎤 Speakers
                              - **Niall Thomson** — Container Specialist Solutions Architect, AWS (EKS/Kubernetes focus)
                              - **Hasith Kalpage** — Head of Platform Engineering & Security, Cisco Outshift (incubation unit)
                              
                              ---
                              
                              ## Part 1: The Platform Engineering Problem (Niall)
                              
                              ### Why Platform Engineering Struggles Today
                              
                              The session opens with a frank assessment: 76% of orgs have a dedicated platform team (DORA 2025), but most struggle. The core issue is that platform teams end up as **human support desks** instead of building new capabilities. The failure modes are:
                              - Abstractions not thought through end-to-end
                              - Documentation that was last updated in 2022
                              - Constant Slack/JIRA handholding eating up engineering time
                              
                              ### Where AI Fits In — Beyond Just Coding
                              
                              A key stat Niall highlights: **the median developer spends less than one hour a day writing code**. The rest of their day is CI/CD fixes, vulnerability patching, PR reviews, incident response, and cost optimization. So if platform teams only think about AI for code generation, they're solving a small fraction of the problem.
                              
                              The better framing: **meet developers where they already are** — their IDE, CLI, GitHub, Backstage, Slack, incident tools — and inject AI assistance in all those channels.
                              
                              ---
                              
                              ## Part 2: The Live Demo Walkthrough
                              
                              The demo shows a developer named John whose pipeline has failed. The scenario:
                              - Platform is built on **Kubernetes + ArgoCD + CodePipeline**
                              - John uses **Kiro CLI** (formerly Amazon Q Developer CLI) with a custom MCP tool called `Query`
                              - He types in plain English: *"troubleshoot why the last CI/CD pipeline execution failed for my payment API workload"*
                              
                              ### What Happens Under the Hood
                              
                              1. Kiro sends the query via **MCP to a central platform agent**
                              2. The platform agent consults an **agent card registry** to discover specialist agents
                              3. It calls a **catalog agent** → hits Backstage MCP server → resolves "payment API" to an exact workload ID
                              4. It calls a **CI/CD agent** → hits AWS API MCP server → pulls CodePipeline status + CodeBuild logs
                              5. The same CI/CD agent hits **ArgoCD MCP** → sees the sync failure and Kubernetes events showing memory request > memory limit
                              6. It also hits **GitHub MCP** → pulls the specific commit that introduced the bad change
                              7. Everything flows back up through the agents → Kiro gets a full diagnosis + remediation options
                              8. Kiro locally edits the YAML file → developer reviews → pushes fix → pipeline passes
                              
                              ### The Three Trigger Modes Demonstrated
                              1. **Manual via CLI** — developer asks Kiro
                              2. **Manual via Slack** — same central agent, same answer, different channel
                              3. **Autonomous/event-driven** — pipeline failure fires an event → agent auto-raises a pull request for human review
                              
                              The key point: **the central AI is one platform capability reused across all channels**, not built separately for each one.
                              
                              ---
                              
                              ## Part 3: The Technical Architecture
                              
                              ### Building Blocks
                              
                              | Layer | What It Is |
                              |---|---|
                              | **Agent Framework** | Strands Agents (AWS, open source) — but LangChain, LangGraph, CrewAI etc. all valid |
                              | **Context/Tools** | MCP servers — Backstage, ArgoCD, GitHub, AWS API, Kubernetes |
                              | **Agent Comms** | A2A (Agent2Agent protocol, Google → Linux Foundation) |
                              | **Deployment** | Standard EKS deployments — agents are just APIs/containers |
                              | **Observability** | OpenTelemetry (OTEL) |
                              
                              ### Why Multi-Agent Instead of One Big Agent
                              
                              Niall references a LangChain study showing that as you add more tools to a single agent, its ability to pick the right tool degrades. The solution: **specialist agents** each with their own tools and prompts — a CI/CD agent, a catalog agent, a GitHub agent — orchestrated by a supervisor/platform agent.
                              
                              ### MCP vs A2A — The Distinction
                              
                              | Protocol | Purpose | Analogy |
                              |---|---|---|
                              | **MCP** | Agent ↔ Tool/API communication | USB-C: standard plug for any tool |
                              | **A2A** | Agent ↔ Agent communication | Microservices API contracts, but for agents |
                              
                              A2A introduces **Agent Cards** — a well-known endpoint (like OIDC's `.well-known`) where an agent advertises what it can do, example prompts, its URL and version. This lets agents dynamically discover each other and delegate tasks without hardcoded wiring.
                              
                              ---
                              
                              ## Part 4: Cisco Outshift's Real-World Implementation (Hasith)
                              
                              ### Context
                              Hasith joined Cisco's incubation unit in Jan 2024 to find a burnt-out SRE team drowning in support requests. They started a **grassroots, bottom-up** agentic AI project — not a top-down mandate.
                              
                              ### The Outshift Platform Stack
                              - **Cloud**: Single AWS provider (dev/staging/prod)
                              - **Interfaces**: WebEx (messaging), Backstage (dev portal), JIRA, CLI/IDE
                              - **Observability**: Splunk
                              - **Agent System**: Named **JARVIS** internally, now open-sourced as **CAIPE**
                              
                              ### Three Demo Use Cases Shown
                              
                              **1. LLM API Key Provisioning**
                              - Used to take half a day with multiple back-and-forth emails
                              - Now: developer asks in Backstage chat → JARVIS asks clarifying questions (which model? which project?) → key provisioned **in under 2 minutes** with full audit trail via an LLM gateway
                              
                              **2. Dev Machine Provisioning via JIRA**
                              - Developer creates a vague JIRA ticket asking for an EC2 instance
                              - SRE assigns it to JARVIS
                              - JARVIS asks clarifying questions, presents options (EC2 vs EKS)
                              - Human-in-the-loop GitOps approval via PR
                              - Dev gets instance details + private key over secure channel
                              - Entire flow: end-to-end automated with audit trail
                              
                              **3. Post-Outage Triage ("Deep Agent" Pattern)**
                              - Morning after a big outage, SRE on-call is exhausted
                              - Developer asks: *"what JIRA tickets are open from last night?"*
                              - Supervisor agent (a "deep agent") plans tasks: query PagerDuty, query JIRA, correlate findings
                              - Spawns **two parallelized sub-agents** to do it simultaneously
                              - Returns: on-call SRE contact, PagerDuty schedule link, 4 open issues with JIRA links
                              - Completed in **40 seconds**
                              
                              ### Impact Metrics at Outshift
                              - **Eliminated a dedicated 3-engineer support desk** — that capacity redirected to engineering work
                              - **20+ Slack "ask rooms"** generating unanswered questions → now intercepted by AI with high-confidence answers
                              - Tasks like LLM key requests or dev machine provisioning: **days/hours → minutes**
                              
                              ---
                              
                              ## Part 5: CAIPE — The Open Source Project
                              
                              Cisco open-sourced their internal system as **CAIPE (Cloud Native AI Platform Engineering)**, now a special interest group under **CNOE (Cloud Native Operational Excellence)**.
                              
                              ### What CAIPE Includes
                              - Built-in **knowledge base** (RAG + GraphRAG, 3rd iteration)
                              - **Ontology agent** that auto-maps relationships between systems
                              - Framework support: **LangGraph + Strands Agents**
                              - Protocols: **MCP + A2A** for interoperability
                              - Observability: everything abstracted to **OTEL**
                              - **Backstage plugin** — A2A-compatible chat interface with streaming, structured form generation
                              - Available at **caipe.io**, weekly community meetups
                              
                              ---
                              
                              ## Key Takeaways
                              
                              1. **Start with knowledge, not actions** — build RAG over your docs, wikis, and tribal knowledge before giving agents the ability to change production systems
                              
                              2. **Centralize the AI as a platform capability** — one agent layer, reused across Slack, CLI, Backstage, JIRA, GitHub; not rebuilt for every channel
                              
                              3. **Multi-agent > single agent** — specialists beat generalists at the cost of orchestration complexity
                              
                              4. **Human-in-the-loop is not optional (yet)** — even the most autonomous flows in the demo end with a PR review or approval step
                              
                              5. **Open source gives you 80% for free** — MCP servers for Backstage, ArgoCD, GitHub, Kubernetes, and AWS APIs already exist; you mostly need to wire them together
                              
                              6. **The hard part is human transformation** — Hasith's closing point: the technology is largely solved; organizational trust, change management, and growth mindset are the real blockers
