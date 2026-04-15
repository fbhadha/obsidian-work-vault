# Proposed Agent Workflows — High-Level Proposals

Five agent workflow concepts for the A1A team. Each proposal covers the problem, the approach, how it connects to Acme's strategic priorities, and what it would take to build a prototype. Each one is designed with modularity in mind — component agents and sub-patterns are scoped so they can be reused across future projects.

Each workflow uses one of six proven orchestration patterns — a repeatable way of organizing agents that fits a particular shape of work:

1. **Pipeline** — Agents work in strict sequence, each handing output to the next.
2. **Fan-out / Fan-in** — One agent sends tasks to multiple agents in parallel, a final agent collects results.
3. **Expert Pool** — A classifier decides what type of request it is and routes to the right specialist.
4. **Producer-Reviewer** — One agent creates, a second evaluates. Failures loop back for revision.
5. **Supervisor** — A coordinator assigns tasks dynamically based on what it learns at each step.
6. **Hierarchical Delegation** — A top agent breaks the problem into sub-problems and delegates down.

---

## 1. RFP Response Assembler

### The problem

Enterprise sales teams spend days assembling responses to RFPs. Each response requires pulling from product specifications, pricing sheets, SLA terms, security certifications, and case studies scattered across multiple systems and shared drives. The work is repetitive — many RFP questions are variations of ones already answered — but the stakes are high. A sloppy response loses the deal. A slow response misses the deadline.

### The approach

**Pattern:** Pipeline with Producer-Reviewer loop

Three agents in sequence. A **Requirements Parser** reads the incoming RFP document and extracts each question or requirement into structured format — what's being asked, what category it falls into (technical, commercial, compliance, reference), and what response format is expected. A **Content Retriever** matches each requirement against a knowledge base of pre-approved response blocks using RAG. It pulls the best-fit content for each question and flags gaps where no approved material exists. A **Draft Assembler** composes the full response document, ensures formatting consistency, and passes it through a reviewer agent that checks for completeness (did every question get answered?), consistency (do pricing figures in section 3 match section 7?), and tone. Gaps are surfaced to the sales team for manual input.

### Strategic connection

Directly tied to revenue. Faster, higher-quality RFP responses increase win rates. The knowledge base compounds over time — every answered RFP makes the next one faster. Built with modularity in mind: the document parsing, RAG retrieval, and gap detection components are scoped as independent modules that can be reused in any future workflow that assembles documents from existing content.

### What it takes to prototype

Gemini Enterprise for the three agents. A sample knowledge base built from publicly available Acme product pages and generic telecom RFP content. A sample RFP document (these are easy to find — telecom procurement RFPs are frequently published by government agencies). Streamlit frontend that shows the parsed requirements, matched content, identified gaps, and assembled draft.

---

## 2. Employee Onboarding Coordinator

### The problem

When a new hire is confirmed, four or five departments need to act: IT provisions a laptop and accounts, Facilities creates a badge and assigns workspace, Finance sets up payroll, and the hiring manager plans the first week. These requests currently go out via email. Nobody owns the coordination. Things get dropped. New hires show up on day one without a laptop, without system access, or without a desk. Every dropped task creates a bad first impression and wastes the new employee's first week.

### The approach

**Pattern:** Fan-out / Fan-in

A **Dispatcher** agent takes a confirmed hire notification and sends parallel task requests to each department simultaneously. Each request is tailored to the department — IT gets the role-specific access requirements, Facilities gets the office location and start date, Finance gets the compensation details, and the manager gets a suggested first-week schedule template. A **Tracker** agent monitors completion status across all streams. At 24 hours, it sends a gentle reminder to any incomplete task. At 48 hours, it escalates to the hiring manager. When all tasks are confirmed complete, it generates a bilingual (EN/FR) welcome package and sends it to the new hire. The Tracker produces a completion dashboard showing average onboarding time, most common delays, and departments that consistently miss deadlines.

### Strategic connection

Feeds the cost savings target through operational efficiency — fewer dropped tasks, less time spent chasing departments, faster time-to-productivity for new hires. Built with modularity in mind: the parallel dispatch and status tracking components are scoped as independent modules, reusable for any multi-department coordination workflow (office moves, equipment refreshes, project kickoffs).

### What it takes to prototype

Gemini Enterprise for the Dispatcher and Tracker agents. Synthetic hire data (name, role, department, start date, office location). Mock department endpoints (for the prototype, these are just simulated responses with variable completion times). Streamlit frontend showing the fan-out in real time — four parallel task cards that flip from "pending" to "complete" as departments respond, with the tracker escalating the slow ones.

---

## 3. Network Incident Triage

### The problem

When a network issue hits, the NOC (Network Operations Centre) Tier 1 team has to quickly determine what type of incident it is, how severe it is, who needs to know, and what to tell customers. This triage is currently manual — an operator reads the alert, checks monitoring dashboards, makes a judgment call on severity, and then drafts notifications for both internal teams and affected customers. During major incidents, multiple operators work the same problem without clear coordination. During minor incidents, notifications are slow because the operator is also handling three other alerts.

### The approach

**Pattern:** Expert Pool

A **Classifier** agent reads the incoming incident alert and categorizes it by type (outage, degradation, planned maintenance, false alarm) and severity (critical, major, minor). Based on the classification, it routes to one of three specialist agents. The **Outage Specialist** drafts customer-facing notifications and internal escalation summaries simultaneously, pulling affected region and estimated impact from the alert metadata. The **Degradation Specialist** assesses scope, identifies affected services, and estimates resolution time based on historical patterns. The **Maintenance Specialist** verifies the change window, confirms planned status, and generates a standard advisory notice. All customer-facing communications require human review before sending. The Classifier also flags potential false alarms, saving operators from investigating noise.

### Strategic connection

Network reliability is the core product. Speed of triage directly affects customer experience, churn, and regulatory compliance (CRTC has reporting timelines for service disruptions). Built with modularity in mind: the classification and notification drafting components are scoped as independent modules, reusable for any triage workflow (IT service desk, fraud detection, facilities incident management). Feeds the cost savings target by reducing operator time on routine classification and notification drafting.

### What it takes to prototype

Gemini Enterprise for the Classifier and three specialist agents. Synthetic incident alerts (JSON payloads with fields like alert type, affected region, affected service, timestamp, monitoring system source). A small knowledge base of notification templates by incident type and severity. Streamlit frontend showing the classification result, the routing decision, and the drafted notification side by side. Demo impact: paste an alert, watch it classify and route in seconds, see the drafted customer notification ready for human review.

---

## 4. Contract Clause Reviewer

### The problem

The legal team reviews vendor and customer contracts manually, reading through pages of boilerplate to find the clauses that actually matter: liability caps, termination provisions, auto-renewal terms, SLA commitments, and indemnification language. Most of a contract is standard. The legal team's value is in catching the non-standard clauses — the ones that create risk. But finding those clauses requires reading everything, which means lawyers spend most of their time on boilerplate and less time on the parts that need judgment.

### The approach

**Pattern:** Producer-Reviewer

A **Clause Extractor** agent reads the contract and identifies key clauses, extracting each one with its section reference, clause type (liability, termination, renewal, SLA, indemnification, confidentiality, governing law), and the specific language used. A **Deviation Scorer** agent compares each extracted clause against a library of Acme's approved standard language for that clause type. It scores each clause as "standard" (matches approved language), "minor deviation" (differs in wording but not substance), or "requires legal review" (materially different from standard terms). Minor deviations get a plain-language explanation of what's different. Clauses scored as requiring review are surfaced to the legal team with the specific deviation highlighted and the standard language shown side by side for comparison. The legal team focuses only on the flagged clauses.

### Strategic connection

Legal bottlenecks slow down procurement and sales cycles. Faster contract review means faster deal closure. Feeds the cost savings target by reducing outside counsel spend on routine contract review. Built with modularity in mind: the clause extraction and comparison-against-reference components are scoped as independent modules, reusable for any document comparison workflow (policy updates, regulatory change impact analysis, insurance policy review).

### What it takes to prototype

Gemini Enterprise for the Extractor and Scorer agents. A small library of standard clause language (5-10 clause types, with Acme's preferred wording for each). Two or three sample contracts with a mix of standard and non-standard clauses (generic telecom vendor agreements are widely available as templates). Streamlit frontend showing the extracted clauses in a table, colour-coded by score (green/yellow/red), with the side-by-side comparison for flagged items.

---

## 5. Regulatory Filing Preparer

### The problem

Acme files regularly with the CRTC and other regulatory bodies — annual reports, spectrum licence renewals, tariff filings, accessibility compliance reports. Each filing has a specific template, required data from multiple internal systems, mandatory sections, and hard deadlines. The work involves pulling network coverage data from engineering, service quality metrics from operations, financial disclosures from finance, and legal language from regulatory affairs. Coordination across these groups is manual. Missing a section or filing late has real consequences — fines, licence conditions, and reputational damage with the regulator.

### The approach

**Pattern:** Hierarchical Delegation

A **Filing Coordinator** agent receives the filing type and deadline, determines which sections are required based on the regulatory template, and delegates each section to a specialist sub-agent. A **Network Data Agent** pulls coverage statistics and infrastructure metrics. A **Service Quality Agent** pulls customer complaint volumes, outage records, and resolution times. A **Financial Disclosure Agent** pulls the required revenue and investment figures. A **Legal Language Agent** assembles the mandatory regulatory declarations and attestations from approved templates. Sub-agents may further delegate — the Financial Disclosure Agent, for example, pulls from SAP, BigQuery, and the annual report separately and reconciles the figures. A **Filing Assembler** collects all sections, runs a completeness check against the regulatory template (are all required sections present? are all mandatory data fields populated?), and flags any gaps. Human review is mandatory before submission — no filing goes to the regulator without sign-off from the Director of Regulatory Affairs.

### Strategic connection

Regulatory compliance is non-negotiable at a telco. Late or incomplete filings create risk with the CRTC at a time when the regulatory relationship is already strained. Feeds the cost savings target by reducing the coordination overhead across four or five departments for each filing cycle. Built with modularity in mind: the hierarchical decomposition, multi-source data collection, and completeness validation components are scoped as independent modules, reusable for any multi-department reporting workflow (board reporting, investor disclosures, audit preparation).

### What it takes to prototype

Gemini Enterprise for the Coordinator, section-level sub-agents, and Filing Assembler. A sample CRTC filing template (these are publicly available on the CRTC website). Synthetic data for each section (mock coverage stats, service quality metrics, financial figures). Streamlit frontend showing the delegation tree — the Coordinator at the top, section agents below, sub-agents below those — with each node showing completion status. Demo impact: enter "CRTC Annual Report — due June 30" and watch the system decompose it into sections, delegate, collect, and assemble, flagging one missing section for human attention.
