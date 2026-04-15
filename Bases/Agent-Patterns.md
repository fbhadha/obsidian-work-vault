## Why patterns matter
 
Every AI agent workflow is a team of specialists working together. The question is how you organize that team. Get the structure wrong and you build something fragile, slow, or impossible to debug. Get it right and the same agents produce dramatically better results.
 
Software engineering solved this problem decades ago with design patterns — proven structural templates that engineers reuse instead of reinventing from scratch. Agent orchestration needs the same thing. The six patterns below are drawn from production deployments, open-source frameworks like CrewAI and Harness, and academic research on multi-agent systems (ICLR 2025, arXiv 2026). They are not theoretical. Each one maps to a specific kind of work.
 
The goal is not to memorize these. It's to recognize which pattern fits when you look at a workflow, so you can design agents that work reliably rather than agents that demo well and break in production.
 
---
 
## The Six Patterns
 
### 1. Pipeline
 
**How it works:** Agent A finishes its work and hands it to Agent B, who finishes and hands it to Agent C. Nobody starts until the person before them is done. It's an assembly line.
 
**Best for:** Linear processes with clear handoffs. Each step transforms the output of the previous step. Document processing, data enrichment, report assembly.
 
**Strengths:** Simple to build, simple to debug, simple to explain. If something breaks, you know exactly which stage failed because the chain is linear.
 
**Weaknesses:** Slow — total time is the sum of every step because nothing runs in parallel. A failure at any stage blocks everything downstream. No flexibility to skip steps or change order based on context.
 
**Real-world example:** An invoice arrives. Agent 1 reads the document and extracts line items. Agent 2 validates amounts against the purchase order. Agent 3 formats the approval request and routes it to the right person. Each step depends entirely on the one before it.
 
---
 
### 2. Fan-out / Fan-in
 
**How it works:** One agent takes a task, splits it into independent pieces, and sends each piece to a different agent simultaneously. A final agent collects all the results and combines them.
 
**Best for:** Processes where independent sub-tasks can run at the same time. Data collection from multiple sources, multi-department coordination, parallel notifications.
 
**Strengths:** Speed — four things happen at once instead of in sequence. Scales naturally by adding more parallel agents.
 
**Weaknesses:** The combining step (fan-in) is where complexity lives. What happens when three agents finish and one fails? The aggregator needs to handle partial results, timeouts, and conflicting outputs gracefully.
 
**Real-world example:** A new employee gets hired. A dispatcher agent sends requests to IT (provision laptop), Facilities (create badge), Finance (set up payroll), and the manager (schedule first week) — all at the same time. A tracker agent waits for all four to report back and flags anything incomplete.
 
---
 
### 3. Expert Pool
 
**How it works:** A router agent examines the incoming request, classifies it, and sends it to the right specialist. Only one specialist handles each request. The other specialists never see it.
 
**Best for:** Triage and classification tasks. Support tickets, document routing, request handling — anywhere the first question is "what type of thing is this?"
 
**Strengths:** Each specialist agent can be deeply optimized for its category. Adding a new category means adding a new specialist without changing the router logic.
 
**Weaknesses:** Router accuracy determines system quality. If the router misclassifies a request, the wrong specialist handles it and the output is wrong. The entire system is only as good as the front door.
 
**Real-world example:** A customer message comes in. The router reads it and classifies it: billing dispute, technical issue, cancellation intent, or general inquiry. It sends it to the appropriate specialist agent. The billing agent knows credit policies; the retention agent knows current save offers. They don't need to know each other's domains.
 
---
 
### 4. Producer-Reviewer
 
**How it works:** One agent creates something. A second agent evaluates it against a set of criteria. If the output passes, it moves forward. If it fails, it goes back to the producer with specific feedback for revision. The loop repeats until the output passes or hits a maximum number of attempts.
 
**Best for:** Any workflow where quality control matters more than speed. Content generation, code review, compliance checking, legal document drafting.
 
**Strengths:** Catches errors before they reach the end user. The feedback loop means quality improves with each iteration. Separating creation from evaluation prevents one agent from being both judge and author.
 
**Weaknesses:** Can loop indefinitely if the review criteria are vague or contradictory. Always needs a cap on iterations. Slower than single-pass generation because of the review cycles.
 
**Real-world example:** MAP (our campaign content drafting tool) uses this pattern. The content drafter writes an ad. The compliance checker reviews it for brand violations, legal disclaimer accuracy, and offer-length hallucinations. If something's wrong, the drafter revises. Humans do the final review before anything ships.
 
---
 
### 5. Supervisor
 
**How it works:** A coordinating agent manages a team of worker agents, assigning tasks dynamically based on progress and results. Unlike Pipeline (fixed sequence) or Fan-out (everything at once), the Supervisor decides what to do next based on what it has learned so far.
 
**Best for:** Complex workflows where the sequence is not fully predictable in advance. Incident response, research tasks, project coordination — anywhere the next step depends on what you find in the current step.
 
**Strengths:** Most flexible pattern. Can adapt in real time. Handles uncertainty and branching logic naturally.
 
**Weaknesses:** Hardest to build reliably. The Supervisor must maintain state across the whole workflow. If it loses track of what has been done, the system breaks. Also the hardest to debug because execution paths vary from run to run.
 
**Real-world example:** An incident response. The Supervisor agent gets a network alert, assigns one agent to classify severity, reads the result, then decides the next move. Low severity: dispatch a notification agent. High severity: simultaneously escalate to a human, draft customer communications, and assign a root-cause analysis agent. The plan changes based on what the classifier finds.
 
---
 
### 6. Hierarchical Delegation
 
**How it works:** A top-level agent breaks a big problem into sub-problems and delegates each one to a sub-agent. Those sub-agents might further break their piece down and delegate again. Results flow back up the chain and get assembled at the top.
 
**Best for:** Large, multi-phase processes where each phase has its own complexity. Enterprise reporting, multi-department audits, strategic planning.
 
**Strengths:** Handles complexity by decomposition. You can break arbitrarily large problems into manageable pieces. Each layer can be independently tested and improved.
 
**Weaknesses:** Context loss. By the time you are three layers deep, the bottom agent may not understand the bigger picture of why it is doing what it is doing. Debugging requires tracing through multiple delegation layers. Latency compounds with each layer.
 
**Real-world example:** A quarterly business review. The top agent decides the review needs four sections: financial performance, operational metrics, customer satisfaction, and competitive landscape. It delegates each section to a specialist agent. The financial agent further delegates to sub-agents that pull revenue data, expense data, and variance analysis separately. Everything assembles back up through the chain.
 
---
 
## Combining Patterns
 
Most real workflows are not a single pure pattern. They are combinations. A Pipeline where one stage contains a Producer-Reviewer loop. A Fan-out where the initial routing uses an Expert Pool. A Hierarchical Delegation where the leaf agents use Producer-Reviewer for quality control.
 
The skill is recognizing which pattern governs the primary structure, and where a secondary pattern applies at a specific stage. Agent Forge's Architecture Designer selects the primary pattern first, then layers secondary patterns where the workflow demands them.
 
---
