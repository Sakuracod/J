# JARVIS Research OS

> A multi-user, low-latency, laboratory-scale AI operating system for scientific research, experiment orchestration, long-term memory, literature management, compute scheduling, collaboration, and laboratory device control.

---

## 1. Vision

JARVIS Research OS is designed as a persistent scientific operating layer for a laboratory rather than as a single chatbot.

The system is intended to run on a dedicated Ubuntu AI workstation with approximately:

- **4 × Blackwell-class GPUs**
- **~384 GB total GPU memory**
- **512 GB system RAM**
- High-speed NVMe storage
- 10/25 GbE laboratory networking
- Multiple laboratory computers, displays, instruments, and user devices

The system is expected to support:

- **10–20 laboratory users**
- One highest-privilege **Owner**
- Two lower-privilege **System Maintainers**
- Multiple ordinary **Researchers**
- Optional temporary **Guests**
- Persistent local and cloud AI models
- Voice-first and text-based interaction
- Multiple displays and multi-device login
- Long-term per-user memory
- Per-project and per-task sessions
- Experiment execution and monitoring
- Literature search and Zotero organization
- Scientific provenance and reproducibility
- Shared laboratory collaboration sessions
- Proactive alerts and research briefings

The primary interaction language is **English**.

---

# 2. Core Design Principle

JARVIS is not defined by a specific LLM.

The system is defined as:

```text
JARVIS
=
Research OS
+ Model Runtime
+ Agent Runtime
+ Memory System
+ Experiment Runtime
+ Laboratory Control Plane
+ Trust Layer
+ Realtime Human Interface
```

Models such as GPT, Qwen, DeepSeek, and future models are interchangeable intelligence backends.

The system should always prefer:

> **the fastest backend that can correctly and safely complete the requested task.**

---

# 3. Six Core Systems

```text
                 JARVIS RESEARCH OS

┌───────────────────────────────────────────────┐
│ 1. HUMAN INTERFACE                            │
│ Voice / Web / Desktop / Mobile / Displays    │
├───────────────────────────────────────────────┤
│ 2. INTELLIGENCE                               │
│ Model Router / Agents / Reasoning / Tools     │
├───────────────────────────────────────────────┤
│ 3. MEMORY & KNOWLEDGE                         │
│ User / Project / Session / Papers / Graph     │
├───────────────────────────────────────────────┤
│ 4. RESEARCH EXECUTION                         │
│ Experiments / Code / Data / Zotero / Jobs    │
├───────────────────────────────────────────────┤
│ 5. LAB CONTROL                                │
│ GPUs / PCs / Displays / Instruments / Nodes   │
├───────────────────────────────────────────────┤
│ 6. TRUST LAYER                                │
│ Identity / Permission / Provenance / Audit    │
│ Sandbox / Secrets / Backup / Recovery         │
└───────────────────────────────────────────────┘
```

These six systems are connected through a common event bus and shared identity, policy, memory, and provenance services.

---

# 4. Implementation Philosophy: C/C++ First

A major implementation rule of this project is:

> **Use C or C++ wherever practical. Use Python only where the scientific or AI ecosystem makes Python clearly advantageous.**

Python should not become the default implementation language for every service.

The reasons include:

- Heavy interpreter startup
- Large dependency trees
- Slow and fragile import chains
- Virtual-environment complexity
- Packaging instability
- Runtime overhead in high-frequency services
- Reduced control over memory, threading, IPC, and latency

## Preferred implementation languages

| Component | Preferred language |
|---|---|
| Realtime gateway | C++ |
| Event bus client services | C++ |
| Voice streaming | C++ |
| VAD / audio pipeline | C++ |
| Model routing hot path | C++ |
| GPU monitoring | C++ |
| Job scheduler adapters | C++ |
| Node agent | C++ |
| Display agent backend | C++ |
| Authentication gateway | C++ |
| Policy enforcement | C++ |
| Filesystem watchers | C++ |
| Instrument drivers | C/C++ |
| WebRTC transport | C++ |
| IPC / shared memory | C++ |
| Model inference runtime | C++ when possible |
| CUDA extensions | C++ / CUDA |
| Frontend | TypeScript |
| Scientific notebooks | Python where needed |
| Scientific fitting scripts | Python where ecosystem advantage is substantial |
| One-off research analysis | Python allowed |
| PyTorch-only workflows | Python allowed |
| Legacy scientific packages | Python wrappers allowed |

The architecture must allow Python research code without making the entire system dependent on Python.

A Python worker should normally be treated as an **isolated research execution worker**, not as the core operating substrate of JARVIS.

---

# 5. High-Level Architecture

```text
                            USER DEVICES
             ┌────────────────────────────────────┐
             │ Voice / Browser / Laptop / Tablet │
             │ Phone / Workstation / Lab Console │
             └─────────────────┬──────────────────┘
                               │
                     WebRTC / WebSocket / HTTPS
                               │
             ┌─────────────────▼──────────────────┐
             │        REALTIME CORE GATEWAY       │
             │ Identity / Sessions / Streaming   │
             └─────────────────┬──────────────────┘
                               │
               ┌───────────────▼────────────────┐
               │       INTELLIGENCE ROUTER      │
               │                                │
               │ Fast / Deep / Local / Cloud   │
               └─────────┬──────────┬───────────┘
                         │          │
              ┌──────────▼───┐  ┌──▼────────────┐
              │ Local Models │  │ Cloud Models  │
              │ Qwen/DS/etc. │  │ GPT/etc.      │
              └──────────┬───┘  └──┬────────────┘
                         └──────┬────┘
                                │
                   ┌────────────▼─────────────┐
                   │      AGENT RUNTIME       │
                   │ Research / Code / Paper │
                   │ Compute / Display / Lab │
                   └────────────┬─────────────┘
                                │
                 ┌──────────────▼──────────────┐
                 │       TRUST LAYER           │
                 │ Policy / Auth / ACL / Audit │
                 └──────────────┬──────────────┘
                                │
         ┌──────────────────────▼──────────────────────┐
         │                  EVENT BUS                  │
         └───────┬──────────┬──────────┬──────────────┘
                 │          │          │
        ┌────────▼───┐ ┌────▼─────┐ ┌──▼─────────────┐
        │   Memory   │ │ Compute  │ │ Lab / Display │
        │   System   │ │ Runtime  │ │ Node Agents   │
        └────────────┘ └──────────┘ └────────────────┘
```

---

# 6. Multi-User System

JARVIS is designed for approximately 10–20 active laboratory members.

## 6.1 Default Roles

### Owner

One user.

The Owner has the highest authority over:

- User management
- Role assignment
- Security policies
- Model policies
- Resource quotas
- Project ownership
- System configuration
- Emergency override
- Audit review
- Laboratory-wide memory policies

The Owner should not need to operate as root for ordinary use.

---

### System Maintainer

Two users.

Maintainers are responsible for:

- Service health
- GPU and compute monitoring
- Model deployment
- Software upgrades
- Driver maintenance
- Job recovery
- Storage maintenance
- Network diagnostics
- Log analysis

Maintainer privilege is intentionally lower than Owner privilege.

System administration privileges must **not automatically grant unrestricted access to private user research memory**.

---

### Researcher

Ordinary laboratory users.

Researchers may:

- Run their own experiments
- Use approved models
- Search literature
- Maintain project memory
- Access projects for which they have permission
- Use shared displays
- Create collaborative sessions
- Manage their own jobs and artifacts

---

### Guest

Temporary or external collaborators.

Guest access should be:

- Time-limited
- Project-limited
- Device-limited
- Read-only by default

---

# 7. Permission Model

JARVIS should use both:

```text
RBAC + ABAC
```

## RBAC

Role Based Access Control determines general capabilities:

```text
Owner
Maintainer
Researcher
Guest
```

## ABAC

Attribute Based Access Control evaluates:

- Who is requesting the action?
- Which project is involved?
- Who owns the data?
- Which device is involved?
- What type of action is requested?
- How sensitive is the resource?
- Is step-up authentication required?
- Is this an interactive or autonomous action?

Example:

```text
User: Alice
Role: Researcher
Action: stop_job
Target owner: Bob

Decision:
DENY
```

Example:

```text
User: Maintainer-1
Role: System Maintainer
Action: stop_job
Reason: GPU deadlock
Target owner: Bob

Decision:
ALLOW
AUDIT
NOTIFY OWNER OF JOB
```

---

# 8. Authentication

Supported authentication signals:

- Passkey / WebAuthn
- Fingerprint
- Face authentication
- Password
- Voiceprint
- Hardware security key
- Trusted device identity

Biometric data should not be treated as ordinary files.

Voiceprint should be used as a **presence and convenience signal**, not as the only authentication method for high-risk actions.

## Step-up authentication examples

Required for:

- `sudo`
- System policy changes
- User deletion
- Privilege escalation
- Destructive data deletion
- Reading highly restricted data
- Exporting private laboratory data
- Sending externally visible results
- Stopping another user's critical job

---

# 9. Session and Memory Architecture

JARVIS must support long-running research relationships with every user.

The system should not treat memory as a single endless chat log.

Instead:

```text
LAB MEMORY
    │
USER LONG-TERM MEMORY
    │
PROJECT MEMORY
    │
TASK SESSION
    │
RAW INTERACTION
```

---

# 10. Long-Term User Session

Every user receives a persistent long-term session.

Example:

```text
user/dear/main
```

This session stores the user's ongoing working state rather than every raw message.

Example:

```yaml
user: dear

active_projects:
  - stroboscat
  - ga2o3-tmbs
  - jarvis

recent_focus:
  - diffusion fitting
  - figure generation

recent_experiments:
  - exp_20260921_sa_c60
  - exp_20260922_no_drift

pending_tasks:
  - compare alpha fitting
  - regenerate figure

recent_entities:
  experiment:
    - exp_20260922_no_drift
  dataset:
    - sa_c60_705
```

---

# 11. Project Sessions

Each research project maintains independent memory.

Example:

```text
/projects/stroboscat
/projects/ga2o3-tmbs
/projects/jarvis
```

A project should contain:

```text
project_memory
experiments
datasets
papers
hypotheses
decisions
tasks
artifacts
sessions
members
permissions
```

Project context is shared only according to project ACL rules.

---

# 12. Task Sessions

When a user starts a focused activity, JARVIS creates a lightweight task session.

Example:

```text
TS-20260923-017
```

A task session loads only:

```text
Relevant User Memory
+
Relevant Project Memory
+
Related Entities
+
Recent Related Sessions
+
Required Files
```

This avoids loading years of history into every prompt.

---

# 13. Session Merge and Memory Consolidation

A task session must not be copied directly into permanent memory.

Instead:

```text
Task Session
    ↓
Memory Extraction
    ↓
Classification
    ↓
Deduplication
    ↓
Conflict Detection
    ↓
Importance Evaluation
    ↓
Merge
```

Memory objects should be classified as:

```text
FACT
RESULT
DECISION
HYPOTHESIS
TASK
PREFERENCE
ARTIFACT
RELATION
OBSERVATION
```

Example:

```yaml
type: RESULT
project: stroboscat
dataset: sa_c60_705
experiment: EXP-1837

result:
  alpha: 0.82

fit_range:
  start: 0.2_ns
  end: 10_ns

method:
  MSD_power_law

source_session:
  TS-20260923-017
```

---

# 14. Memory Supersession

Scientific memory must support correction.

Old information must not simply be overwritten.

Example:

```yaml
value: 0.01
status: superseded
superseded_by: memory_49392
```

New record:

```yaml
value: 0.20
status: active
```

JARVIS must be able to answer both:

> What value are we currently using?

and:

> Why did we previously use the old value?

---

# 15. Scientific Memory: Evidence vs Interpretation

Scientific memory must distinguish:

```text
Observation
Result
Interpretation
Hypothesis
Confidence
Evidence
Counter-evidence
Status
```

Example:

```yaml
observation:
  alpha: 0.82

interpretation:
  possible_subdiffusion

confidence:
  moderate

evidence:
  - EXP-1837
  - EXP-1843

counter_evidence:
  - EXP-1901

status:
  open_hypothesis
```

The system must never silently convert a hypothesis into a confirmed fact.

---

# 16. Entity Graph

JARVIS maintains a graph connecting:

```text
User
Project
Experiment
Dataset
Code Commit
Environment
Artifact
Figure
Result
Paper
Hypothesis
Task
Instrument
Session
```

Example:

```text
Dataset
  ↓
Preprocessing
  ↓
Experiment
  ↓
Ablation
  ↓
Result
  ↓
Figure
  ↓
Manuscript
```

This enables questions such as:

> Which figures depend on this dataset?

> Which experiment created this result?

> Which code commit generated Figure 3?

---

# 17. Reference Resolution

JARVIS must understand ambiguous human references.

Example:

> Jarvis, have you finished the experiment last week? Give me the result.

JARVIS should resolve:

```text
Speaker
Time window
Current project
Recent experiments
Focus stack
Job history
Conversation state
Semantic similarity
```

If one candidate is dominant:

> Yes. The SA+C60 diffusion run finished at 03:18...

If two candidates are plausible:

> I completed two experiments last week that you may mean.

> **A — SA+C60 diffusion ablation:** ...
>
> **B — no-drift reconstruction:** ...
>
> Which one did you mean?

After the user replies:

> The first one.

JARVIS should bind:

```text
current_reference.experiment = EXP-1837
```

Later references such as:

> Show me the residual.

must continue to resolve to the same experiment.

---

# 18. Personal Focus Stack

Every user maintains an independent focus stack.

Example:

```text
1. stroboSCAT / SA+C60 diffusion
2. TMBS / Figure 1
3. JARVIS architecture
4. OpenMM / 2DIR
```

JARVIS uses this stack for context resolution.

Different users may ask:

> Show me yesterday's result.

and receive completely different answers.

---

# 19. Collaborative Sessions

When several researchers discuss one problem, JARVIS creates a shared collaborative session.

Example:

```text
CS-20260923-sample41
```

Participants:

```text
Dear
Alice
Bob
Chen
```

The session may contain:

- Shared display state
- Annotations
- Discussion transcript
- Evidence
- Decisions
- Assigned tasks
- Relevant datasets
- Related experiments

At session end:

```text
Collaborative Session
    ↓
Summary
    ↓
Decision extraction
    ↓
Task assignment
    ↓
Merge into shared project memory
```

Private user memory must remain separate.

---

# 20. Memory Visibility

Each memory object has explicit visibility:

```text
private
project
lab
custom
```

Example:

```yaml
owner: dear
visibility: project

acl:
  read:
    - stroboscat_team
  write:
    - dear
    - alice
```

---

# 21. Intelligence Router

JARVIS should not send every request to the largest model.

The router decides according to:

- Task type
- Required quality
- Expected latency
- Cost
- Privacy
- Model availability
- Tool requirements
- User policy
- Historical task performance

Example:

```text
Intent:
research_analysis

Latency:
interactive

Privacy:
local-preferred

Tools:
project.search
python_worker
artifact.read

Model:
local-medium
```

---

# 22. Model Tiers

Recommended model tiers:

## L0 — Reflex

Always resident.

Used for:

- Wake word
- VAD
- Intent recognition
- Short commands
- Routing
- Safety classification

---

## L1 — Interactive

Always resident or hot.

Used for:

- Normal conversation
- Tool calling
- Literature queries
- Project questions
- Simple coding

---

## L2 — Deep Reasoning

Used for:

- Complex scientific reasoning
- Code review
- Long technical analysis
- Difficult experiment interpretation
- Multi-step tool orchestration

---

## L3 — Specialist Models

Examples:

- Vision
- OCR
- Embeddings
- Reranking
- Speech recognition
- TTS
- Scientific domain models

---

## Cloud Models

Examples:

- GPT
- Other external models

Cloud models must remain optional.

Local-only operation should remain possible for sensitive work.

---

# 23. Model Routing Metrics

JARVIS should learn which model is best for which task.

Track:

```text
quality
latency
cost
tool-call success
failure rate
user corrections
retry count
```

Example optimization target:

```text
Score
=
w1 * Quality
- w2 * Latency
- w3 * Cost
- w4 * FailureRate
```

Routing policy should improve using historical performance.

---

# 24. Low-Latency Interaction

The target interactive latency is approximately:

```text
100–500 ms to first meaningful response
```

This does not mean every scientific task must finish within 500 ms.

Instead:

```text
User:
"Why did sample 23 fail?"

Jarvis:
"I'm checking the acquisition log and fitting results."
```

The first response appears immediately while deep analysis continues.

Latency-critical path:

```text
Microphone
↓
Streaming VAD
↓
Streaming ASR
↓
Intent prediction
↓
Router
↓
Fast response
↓
Streaming TTS
```

Heavy analysis continues asynchronously within the same request execution chain.

---

# 25. Voice Interface

Default interaction language:

```text
English
```

Speech recognition may support multiple languages.

JARVIS may read multilingual papers and documentation while replying in English unless explicitly requested otherwise.

Recommended design:

```text
Mic
↓
VAD
↓
Streaming ASR
↓
JARVIS Core
↓
Model / Tools
↓
Streaming TTS
↓
Speaker
```

Voice interaction should support:

- Barge-in
- Interruption
- Wake word
- Multiple microphones
- Per-user speaker recognition
- Device-aware responses

---

# 26. Research Execution

JARVIS should treat experiments as first-class objects.

Example command:

> Jarvis, rerun the model without the drift term and compare it with the baseline.

Possible execution:

```text
Understand request
↓
Resolve project
↓
Find baseline
↓
Create git worktree
↓
Generate configuration
↓
Run sanity test
↓
Allocate compute
↓
Run experiment
↓
Monitor
↓
Parse metrics
↓
Generate plots
↓
Compare baseline
↓
Register results
↓
Update Experiment Ledger
↓
Notify user
```

---

# 27. Experiment Ledger

Every experiment should receive a permanent record.

Example:

```text
EXP-001873

Name:
SA+C60 alpha fitting

Owner:
Dear

Project:
stroboSCAT

Created:
2026-09-20 16:21

Started:
2026-09-21 00:15

Finished:
2026-09-21 03:18

Purpose:
Determine anomalous diffusion exponent.

Code:
commit 84fa93

Environment:
env_423

GPU:
GPU-2

Result:
alpha = 0.82 ± 0.05

Artifacts:
plot.png
result.csv
fit.ipynb

Related Session:
TS-8832
```

---

# 28. Scientific Provenance

Every important scientific result should be traceable to:

```text
Result
├── Dataset
├── Experiment
├── Task Session
├── Code Commit
├── Parameters
├── Environment
├── Model Version
├── Hardware
├── Generated Artifacts
└── Timestamp
```

JARVIS must be able to answer:

> Where did that number come from?

> Which raw data produced this figure?

> Which model version generated this result?

---

# 29. Reproducibility

Any important experiment should be replayable.

JARVIS should preserve:

- Git commit
- Dirty working tree state
- Container image or environment hash
- Model version
- Input data hash
- Configuration
- Random seed
- GPU type
- Runtime arguments
- Relevant package versions
- Output hashes

Preferred rule:

> A result that cannot be reproduced should not be promoted to trusted project memory.

---

# 30. Research Tools

JARVIS agents should not receive unrestricted shell access.

Prefer capability-based tools:

```text
git.create_worktree()
experiment.run()
job.submit()
job.cancel()
artifact.read()
artifact.write()
paper.search()
zotero.add()
display.cast()
dataset.register()
```

Avoid:

```text
shell("anything")
```

High-risk operations must pass through the Trust Layer.

---

# 31. Zotero Integration

Zotero serves as a major literature database.

Supported operations should include:

- Search library
- Import paper
- Import PDF
- Organize collection
- Apply tags
- Add note
- Deduplicate DOI
- Retrieve metadata
- Retrieve full text
- Link paper to project
- Link paper to hypothesis
- Link paper to experiment

Example:

> Jarvis, find the most relevant papers from the last six months on operando optical microscopy of battery phase transitions.

Possible workflow:

```text
Query expansion
↓
External literature search
↓
DOI normalization
↓
Deduplication
↓
Reranking
↓
PDF retrieval
↓
Full-text indexing
↓
Project relevance analysis
↓
Zotero import
↓
Collection / tags / notes
```

---

# 32. Compute Runtime

JARVIS must manage:

- GPUs
- CPU
- RAM
- NVMe scratch
- Containers
- Job queues
- Model servers

Example query:

> Jarvis, can I run a four-GPU experiment now?

JARVIS should inspect resource state before answering.

Possible output:

```text
GPU0: realtime services
GPU1: Qwen
GPU2: DeepSeek
GPU3: idle

RAM: 226 / 512 GB
Scratch: 61%

Three GPUs can be freed.
One realtime partition should remain reserved.
```

---

# 33. Compute Isolation

Interactive JARVIS services should not fail because a user experiment consumes all resources.

Reserve resources for:

- Voice
- Router
- Identity
- Policy
- Event bus
- Core memory services
- Monitoring

Experimental workloads should execute under quotas or scheduling policies.

---

# 34. Event Bus

All major services communicate through a common event layer.

Examples:

```text
experiment.started
experiment.completed
experiment.failed

gpu.oom
gpu.temperature.warning

paper.discovered
paper.imported

dataset.created
dataset.modified

user.login
user.logout

display.connected
display.disconnected

instrument.measurement.finished
```

Core services subscribe to events rather than creating tightly coupled point-to-point dependencies.

---

# 35. Proactive JARVIS

JARVIS should react to events without waiting for a user prompt.

Example:

```text
02:14

Experiment EXP-1943
loss diverged
GPU utilization dropped
process still alive
```

Possible policy:

```text
low confidence:
continue monitoring

medium confidence:
notify user

high confidence + safe recovery:
restart from checkpoint
notify user
```

Morning interaction:

> Good morning. Two overnight experiments completed successfully. One stalled at 02:14 and was restarted from its latest checkpoint.

---

# 36. Attention Budget

Not every event should interrupt the user.

Priority classes:

```text
P0 Emergency
P1 Important
P2 Relevant
P3 Informational
P4 Background
```

Example:

```text
GPU at dangerous temperature → P0
Critical experiment failure  → P1
New relevant paper           → P2
Normal job completion        → P3
Embedding index updated      → P4
```

---

# 37. Scientific Inbox

Each user receives a research inbox.

Example:

```text
JARVIS INBOX

3 experiments finished
1 experiment failed
4 papers worth reading
2 Git changes need review
1 collaborator shared dataset
GPU scratch storage: 83%
2 unresolved hypotheses
```

Example:

> Jarvis, give me the morning briefing.

The system should summarize only information relevant to that user.

---

# 38. Multi-Display Collaboration

Every laboratory display may run a lightweight Display Agent.

Example registered devices:

```text
microscopy-display-01
meeting-room-left
meeting-room-center
meeting-room-right
office-display-02
```

Example commands:

> Jarvis, put Figure 3 on the center screen.

> Put the raw dataset on the left and the fitted residual on the right.

> Share this problem with everyone.

Collaborative mode may use:

```text
LEFT
Raw data

CENTER
Main anomaly

RIGHT
Model comparison
```

---

# 39. Multi-Device Login

A user session may be active on:

- Workstation
- Laptop
- Browser
- Tablet
- Phone
- Lab terminal
- Meeting-room display
- Voice endpoint

Devices should share a logical user session while maintaining device-specific permissions.

Example:

```text
phone:
notifications + approvals

laptop:
full research interface

meeting-room display:
read-only shared presentation

voice terminal:
voice interaction
```

---

# 40. Laboratory Node Agent

Every laboratory computer should optionally run a JARVIS Node Agent.

Example nodes:

```text
Microscope-PC-01
Spectroscopy-PC
TCAD-Node
Analysis-PC-03
Meeting-Room-PC
```

Standard capabilities:

```text
status()
screen()
files()
launch()
jobs()
devices()
notify()
```

The Node Agent should be implemented in C++ where possible.

---

# 41. Instrument Abstraction

Instrument integrations should use standardized adapters.

Example interface:

```text
connect()
status()
configure()
start()
stop()
snapshot()
stream()
export()
health()
```

Instrument-specific drivers should not directly expose arbitrary host execution.

---

# 42. Safety Kernel

The AI model must not directly control unrestricted privileged operations.

Required path:

```text
LLM
↓
Structured Action Request
↓
Policy Engine
↓
Authentication / Authorization
↓
Tool Adapter
↓
Execution
↓
Audit
```

Never:

```text
LLM
↓
sudo shell
```

---

# 43. Sandboxing

Research code generated or modified by an AI agent should run inside a sandbox by default.

Example:

```text
LLM
↓
Experiment Worker
↓
Container
↓
Restricted Filesystem
↓
Allocated GPU/CPU
```

A research worker may receive:

```text
/projects/stroboscat/
```

but not:

```text
/root/
/etc/
/home/other_user/
```

unless explicitly permitted.

---

# 44. Secrets Management

Secrets must never be stored in:

```text
chat history
long-term memory
prompt context
source repository
plain-text .env files
experiment notes
```

Examples:

- API keys
- SSH keys
- Git tokens
- Cloud credentials
- Instrument passwords
- Database passwords

JARVIS agents should receive only logical secret handles.

Example:

```text
credential_handle = github_lab_token
```

The execution layer resolves the secret only at tool runtime.

---

# 45. Audit Log

Important actions should be auditable.

Record:

```text
who
when
device
project
action
resource
policy decision
authentication method
result
```

Examples:

- Job termination
- User creation
- Permission change
- Data deletion
- External export
- Privilege escalation
- Model configuration update

---

# 46. Failure Recovery

JARVIS must survive:

- Model crash
- GPU OOM
- Network interruption
- Worker failure
- Database restart
- Power failure
- Storage failure
- Broken Python environment
- Failed model upgrade

Core services should be independently restartable.

A failed research worker must not crash the realtime voice interface.

---

# 47. Backup and Disaster Recovery

At minimum back up:

- PostgreSQL
- Project memory
- Experiment ledger
- Provenance graph
- User metadata
- ACLs
- Audit log
- Critical configuration
- Zotero metadata
- Model routing configuration

Large raw datasets may use separate storage policies.

Recommended design:

```text
local ZFS snapshots
+
offline backup
+
optional remote encrypted backup
```

---

# 48. Suggested Technology Stack

The implementation should remain replaceable.

A possible starting stack:

| Layer | Technology |
|---|---|
| OS | Ubuntu LTS |
| Core services | C++ |
| Build system | CMake |
| Package management | Conan/vcpkg where appropriate |
| RPC | gRPC / Protobuf |
| Realtime transport | WebRTC |
| Event bus | NATS |
| Database | PostgreSQL |
| Vector search | pgvector |
| Cache | Redis |
| Storage | ZFS / MinIO |
| Containers | Docker / Podman |
| Compute scheduler | Slurm |
| Monitoring | Prometheus / Grafana |
| Auth | Keycloak + WebAuthn |
| Frontend | TypeScript / React / Next.js |
| Local LLM serving | vLLM / TensorRT-LLM / Triton |
| Experiment tracking | MLflow or internal service |
| Scientific workers | C++ or isolated Python |
| GPU telemetry | NVML / DCGM |
| Voice | native C++ streaming pipeline where possible |

No individual technology is considered permanent.

All core interfaces should be abstracted behind stable internal APIs.

---

# 49. Suggested Repository Layout

```text
jarvis/
│
├── README.md
├── LICENSE
├── CMakeLists.txt
│
├── docs/
│   ├── architecture/
│   ├── security/
│   ├── memory/
│   ├── experiments/
│   └── api/
│
├── proto/
│   ├── identity.proto
│   ├── memory.proto
│   ├── experiment.proto
│   ├── compute.proto
│   ├── display.proto
│   └── events.proto
│
├── core/
│   ├── gateway/
│   ├── router/
│   ├── session/
│   ├── event_bus/
│   └── common/
│
├── trust/
│   ├── identity/
│   ├── policy/
│   ├── secrets/
│   ├── audit/
│   └── sandbox/
│
├── memory/
│   ├── user/
│   ├── project/
│   ├── session/
│   ├── graph/
│   ├── consolidator/
│   └── retrieval/
│
├── intelligence/
│   ├── model_router/
│   ├── local_models/
│   ├── cloud_models/
│   ├── tools/
│   └── agents/
│
├── research/
│   ├── experiment/
│   ├── provenance/
│   ├── datasets/
│   ├── papers/
│   ├── zotero/
│   └── artifacts/
│
├── compute/
│   ├── scheduler/
│   ├── gpu/
│   ├── slurm/
│   ├── containers/
│   └── telemetry/
│
├── lab/
│   ├── node_agent/
│   ├── display_agent/
│   ├── instruments/
│   └── device_registry/
│
├── voice/
│   ├── vad/
│   ├── asr/
│   ├── tts/
│   └── streaming/
│
├── frontend/
│   ├── web/
│   └── shared/
│
├── workers/
│   ├── cpp/
│   └── python/
│
├── deployment/
│   ├── systemd/
│   ├── containers/
│   ├── slurm/
│   └── monitoring/
│
└── tests/
    ├── unit/
    ├── integration/
    ├── security/
    └── performance/
```

---

# 50. Example Daily Usage

## Start a research task

User:

> Jarvis, create a new session for the SA+C60 diffusion analysis.

JARVIS:

```text
Created:
TS-20260923-017

Project:
stroboSCAT

Loaded:
dataset SA-C60-705
latest diffusion model
three related sessions
six relevant papers
```

---

## Run an experiment

User:

> Remove the drift term, rerun the model, and compare it with the current baseline.

JARVIS:

```text
Resolved baseline:
EXP-1811

New experiment:
EXP-1873

Resources:
GPU-3

Execution:
sandboxed

Status:
running
```

---

## Ask later

User:

> Jarvis, did the experiment from last week finish? Give me the result.

JARVIS may answer:

```text
I completed two experiments last week that you may mean.

A — SA+C60 diffusion ablation:
completed; alpha = 0.82 ± 0.05.

B — no-drift reconstruction:
completed; validation residual increased by 18%.

Which one did you mean?
```

User:

> The first one.

JARVIS binds that experiment as the current reference.

---

## Continue naturally

User:

> Show me the residual.

JARVIS displays the residual from experiment A without requesting clarification.

User:

> Put it on the center screen and the baseline on the left.

JARVIS updates the registered laboratory displays.

---

## Literature workflow

User:

> Find recent papers relevant to this result and add the useful ones to Zotero.

JARVIS:

```text
17 papers found
6 high-relevance papers retained
3 added to Zotero
2 already existed
1 requires manual access
```

---

## Morning briefing

User:

> Good morning, Jarvis. Give me the briefing.

JARVIS may report:

```text
Two overnight experiments completed.
One failed and was restarted from checkpoint.
Four new papers are relevant to your active projects.
GPU scratch storage is at 83%.
Alice shared one dataset in the stroboSCAT project.
Two project hypotheses remain unresolved.
```

---

# 51. Example Collaboration Workflow

User:

> Jarvis, share sample 41 with the group.

JARVIS:

1. Creates a collaborative session.
2. Opens relevant data on shared displays.
3. Loads experiment provenance.
4. Invites authorized project members.
5. Records annotations.
6. Records decisions.
7. Creates follow-up tasks.
8. Merges the final shared summary into project memory.

---

# 52. Service Health

Users should be able to ask:

> Jarvis, what is your status?

Expected summary:

```text
Core gateway: healthy
Memory service: healthy
PostgreSQL: healthy
Event bus: healthy
Voice: healthy

GPU0: realtime services
GPU1: local model
GPU2: local model
GPU3: experiment workload

Active jobs: 7
Queued jobs: 2
Failed jobs: 0

Storage:
scratch 61%
archive 47%
```

---

# 53. Development Stages

## Stage 0 — Core Foundations

Implement:

- Identity
- RBAC / ABAC
- PostgreSQL schema
- Event bus
- Audit
- Core C++ service framework
- gRPC / Protobuf contracts
- Device registry

---

## Stage 1 — Text Research JARVIS

Implement:

- Text interface
- Model router
- Local LLM
- Cloud LLM
- Tool calling
- Git integration
- Filesystem tools
- Experiment ledger
- GPU monitoring
- Zotero integration
- Project memory

This stage should already be scientifically useful.

---

## Stage 2 — Memory and Experiment Intelligence

Implement:

- Long-term user session
- Task session
- Memory consolidation
- Entity graph
- Reference resolution
- Focus stack
- Provenance
- Experiment lineage
- Reproducibility

---

## Stage 3 — Realtime Voice

Implement:

- Wake word
- VAD
- Streaming ASR
- Streaming TTS
- Barge-in
- Speaker recognition
- Fast routing

---

## Stage 4 — Laboratory Collaboration

Implement:

- Node Agent
- Display Agent
- Multi-screen casting
- Shared sessions
- Annotations
- Mobile approval
- Multi-device session continuity

---

## Stage 5 — Proactive Research OS

Implement:

- Event-driven monitoring
- Automatic experiment diagnosis
- Scientific inbox
- Morning briefing
- Attention budget
- Safe recovery workflows
- Paper monitoring
- Hypothesis tracking

---

## Stage 6 — Instrument Integration

Implement adapters for:

- Cameras
- Microscopes
- Spectrometers
- Pump-probe systems
- TCAD nodes
- Other laboratory instruments

Instrument control should remain behind policy enforcement.

---

# 54. Non-Negotiable Principles

## 54.1 No single model owns JARVIS

Models are replaceable backends.

---

## 54.2 No unrestricted LLM root shell

All dangerous actions go through tools and policies.

---

## 54.3 Memory is structured

Permanent memory is not an infinite transcript.

---

## 54.4 Scientific claims require provenance

Important results must be traceable.

---

## 54.5 Hypotheses are not facts

Confidence and evidence must be explicit.

---

## 54.6 Research data ownership is explicit

Every object has ownership and visibility.

---

## 54.7 System administration is not equivalent to private-data access

Operational privilege and data privilege are separated.

---

## 54.8 Realtime services are isolated from batch workloads

A failed experiment must not make JARVIS unresponsive.

---

## 54.9 C/C++ is preferred for core infrastructure

Python is a research tool, not the operating foundation.

---

## 54.10 Every important action is auditable

The system must be able to explain what happened.

---

# 55. Long-Term Goal

The final target is not:

```text
a chatbot connected to some tools
```

The target is:

```text
a persistent laboratory intelligence layer
```

capable of understanding:

- who is speaking,
- what project they are working on,
- what experiment they mean,
- which results are current,
- what happened overnight,
- which evidence supports a conclusion,
- what resources are available,
- which people may access which data,
- and what action can be safely performed next.

A successful JARVIS should feel less like opening a chat window and more like interacting with a persistent scientific collaborator that understands the laboratory's state.

---

# 56. Example Target Interaction

```text
User:
Jarvis.

Jarvis:
Yes?

User:
Did you finish the experiment from last week?
Give me the result.

Jarvis:
I completed two experiments last week that you may mean.

A — SA+C60 diffusion ablation:
completed; alpha = 0.82 ± 0.05.

B — no-drift reconstruction:
completed; removing the drift term increased the residual by 18%.

Which one did you mean?

User:
A.

Jarvis:
The run completed successfully.
The early-time residual remains the dominant mismatch.
I have the experiment, baseline, code commit, raw dataset,
and fitting artifact linked in the experiment ledger.

User:
Show it on the center display and put the baseline on the left.

Jarvis:
Done.
```

That interaction summarizes the intended system behavior:

```text
Identity
+ Memory
+ Reference Resolution
+ Experiment Runtime
+ Scientific Provenance
+ Multi-Display Collaboration
+ Natural Voice Interaction
```

---

# 57. Project Status

Current status:

```text
Architecture definition
```

The next implementation milestone should be:

```text
JARVIS Architecture v0.1
```

with stable definitions for:

1. Identity schema
2. Permission schema
3. Memory object schema
4. Session lifecycle
5. Experiment object schema
6. Event schema
7. Provenance schema
8. Core gRPC interfaces
9. C++ service boundaries
10. MVP deployment layout

Once these contracts are stable, individual AI models, inference engines, web frontends, and scientific workers can evolve independently without destabilizing the whole system.
