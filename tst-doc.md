# Local-LLM Project Brain
## A practical architecture and operating guide for reverse engineering, knowledge capture, code generation, review, and functional-model enrichment on a 48-GB Apple Silicon laptop

**Target environment**

- macOS on Apple Silicon (M4 Pro, 48 GB unified memory)
- LM Studio running one model at a time
- GitHub Copilot CLI as the approved agent/tooling shell
- Local models:
  - **Devstral Small 2** — primary reverse-engineering and implementation model
  - **GPT-OSS 20B** — independent technical reviewer/reasoning model
  - **Gemma 4 31B QAT** — functional/human-readable synthesis model
- Initial model context target: **32,768 tokens**
- Project technologies: Java, Spring Boot, Neo4j, Kafka, Cucumber, Gatling, shell utilities, Azure/GitHub pipelines, multiple deployable modules
- Security goal: proprietary code and company information stay inside the company environment; public documentation may be fetched from approved official sources; executable/script downloads from the public Internet require explicit human review and manual acquisition.

---

# 1. Executive recommendation

Do **not** try to make a 32K local model understand the whole repository in one session. Treat the local models as workers that continuously build, verify, and consume a **persistent external project brain** stored as compact structured files on disk.

The project brain is the key. The model context becomes a temporary working memory; the project brain becomes the long-term memory.

The recommended lifecycle is:

```text
SOURCE OF TRUTH
Code + Neo4j + Cucumber + Gatling + Pipelines + Scripts + Functional Docs
                                  |
                                  v
                         DEVSTRAL DISCOVERY
                    small evidence-bounded chunks
                                  |
                                  v
                    RAW TECHNICAL KNOWLEDGE
                                  |
                                  v
                       GPT-OSS VERIFICATION
                    contradictions / gaps / risks
                                  |
                                  v
                    VERIFIED TECHNICAL MODEL
                                  |
                 +----------------+----------------+
                 |                                 |
                 v                                 v
        FUNCTIONAL DOCUMENTS                  RUNTIME FACTS
        terminology/rules                     endpoint usage
                 |                             performance facts
                 +---------------+-----------------+
                                 |
                                 v
                           GEMMA SYNTHESIS
                                 |
                                 v
                         PROJECT BRAIN / MAP
                                 |
                                 v
               RETRIEVE ONLY RELEVANT KNOWLEDGE
                                 |
                                 v
                         DEVSTRAL IMPLEMENTS
                                 |
                                 v
                      BUILD / TEST / CUCUMBER
                                 |
                                 v
                         GPT-OSS REVIEWS
                                 |
                      findings?  |  clean
                         yes     |   no
                          |      |    |
                          +----> Devstral
                                 |
                                 v
                           VERIFIED CHANGE
                                 |
                                 v
                         GEMMA UPDATES BRAIN
```

This is how a smaller local model can become useful on a large enterprise codebase: **do not repeatedly ask it to rediscover the application**.

---

# 2. The core design principle: working memory vs project memory

Your model has a limited active context. That is normal. Do not fight it by increasing context until the laptop swaps heavily.

Use two separate memory layers.

## 2.1 Working memory

This is the 32K model context for the current task only.

A typical working context should contain:

- task statement;
- one small mental-model summary;
- relevant business rules;
- 5–15 relevant source/test files or selected excerpts;
- current review findings;
- build/test output needed for the current decision.

It should **not** contain the complete repository, complete graph dataset, all Cucumber files, or all pipelines.

## 2.2 Persistent project memory

Store durable knowledge under a local directory, for example:

```text
.ai-local/
  brain/
  evidence/
  reverse-engineering/
  functional/
  runtime/
  tasks/
  reviews/
  research/
  generated/
```

The model reads only the pieces of this brain relevant to the current task.

This design gives you effectively unlimited project memory without requiring unlimited LLM context.

---

# 3. Recommended model responsibilities

Do not ask all three models to do the same job. Specialization is the value of your pipeline.

## Devstral Small 2 — Engineer / Reverse Engineer

Use Devstral for:

- repository exploration;
- Java/Spring Boot code understanding;
- Neo4j repository/query analysis;
- Kafka consumer/data-flow analysis;
- Cucumber/Gatling/test reverse engineering;
- shell-script and pipeline analysis;
- implementation planning;
- code generation/refactoring;
- compilation/test correction loops;
- updating raw technical evidence.

Devstral is allowed to inspect and modify application code when you explicitly move into implementation mode.

## GPT-OSS 20B — Independent Technical Reviewer

Use GPT-OSS for:

- challenging Devstral conclusions;
- verifying architectural relationships;
- examining current `git diff`;
- identifying hidden failure modes;
- detecting unsupported assumptions;
- reviewing transaction/query/concurrency logic;
- reviewing performance-sensitive Neo4j paths;
- checking test completeness;
- refining technical documentation without making it more “business friendly.”

The reviewer should normally remain read-only against production source.

## Gemma 4 31B QAT — Mental-Model / Functional Translator

Use Gemma for:

- transforming verified technical details into human-readable concepts;
- connecting functional terminology to implementation;
- creating business/technical traceability;
- documenting workflows such as customer, worker-on-behalf, and worker-only;
- creating readable architecture descriptions;
- explaining Neo4j concepts to non-Neo4j readers;
- preparing concise functional summaries and onboarding documentation;
- updating the enriched project brain after verified technical changes.

Gemma should consume **verified technical evidence**, not invent architecture from source fragments.

---

# 4. Build the project brain as a graph of knowledge, not one giant document

Do not create a single 500-page `project.md`. It will become impossible for small-context models to use efficiently.

Use small linked documents.

Recommended structure:

```text
.ai-local/
├── brain/
│   ├── 00-index.md
│   ├── 01-system-overview.md
│   ├── 02-deployable-modules.md
│   ├── 03-domain-terminology.md
│   ├── 04-business-workflows.md
│   ├── 05-endpoint-catalog.md
│   ├── 06-performance-hotspots.md
│   ├── 07-neo4j-graph-index.md
│   ├── 08-kafka-sync-model.md
│   ├── 09-export-api-model.md
│   ├── 10-testing-model.md
│   ├── 11-pipelines-and-delivery.md
│   ├── 12-shell-utilities.md
│   ├── 13-coding-conventions.md
│   ├── 14-design-patterns.md
│   ├── 15-external-integrations.md
│   ├── 16-security-and-Checks.md
│   ├── 17-known-risks.md
│   └── 18-open-questions.md
│
├── graph/
│   ├── node-types/
│   ├── relationship-types/
│   ├── query-patterns/
│   ├── endpoint-to-query/
│   └── graph-invariants/
│
├── modules/
│   ├── sync/
│   ├── api/
│   ├── export-api/
│   └── shared/
│
├── workflows/
│   ├── business-access-profile.md
│   ├── private-access-profile.md
│   ├── customer-flow.md
│   ├── worker-on-behalf-flow.md
│   └── worker-only-flow.md
│
├── testing/
│   ├── cucumber-style.md
│   ├── cucumber-feature-map.md
│   ├── mockito-style.md
│   ├── integration-test-map.md
│   ├── gatling-model.md
│   └── performance-data-model.md
│
├── traceability/
│   ├── functional-to-endpoint.md
│   ├── endpoint-to-code.md
│   ├── endpoint-to-neo4j.md
│   ├── endpoint-to-cucumber.md
│   └── feature-to-runtime-impact.md
│
├── evidence/
│   ├── code/
│   ├── neo4j/
│   ├── tests/
│   ├── pipeline/
│   └── runtime/
│
├── functional/
│   ├── source-docs/
│   └── normalized/
│
├── tasks/
│   └── current/
│
├── reviews/
│
└── research/
```

Every file should stay relatively small. A good rule is to keep most brain files under roughly **2,000–4,000 words** and split them when they become broader than one coherent topic.

---

# 5. Evidence is mandatory

The project brain must not become a collection of model guesses.

Every important technical claim should contain provenance.

Use a structure such as:

```markdown
## Rule: Customer flow resolves subscription before Check evaluation

Status: FACT
Confidence: HIGH

Evidence:
- `module-api/src/main/java/.../SubscriptionService.java`
- `module-api/src/main/java/.../CheckService.java`
- `integration-tests/src/test/resources/features/...feature`

Observed behavior:
1. ...
2. ...

Open questions:
- ...
```

For Neo4j:

```markdown
## Relationship: OUTLOOK_PARTY -> SUBSCRIPTION

Status: FACT

Evidence from code:
- repository/query file path

Evidence from graph:
- sampled Cypher query
- returned labels/relationship types

Cardinality observed:
- ...

Performance relevance:
- high / medium / low
```

This prevents hallucinated relationships from being promoted to “project truth.”

---

# 6. Reverse engineering order: do not scan randomly

Your project has different deployable components and a very high-volume API. Reverse engineering should be risk- and value-driven.

Use this order.

## Phase A — repository skeleton

Devstral first discovers only structure:

- Maven/Gradle modules;
- deployable components;
- package roots;
- Spring Boot applications;
- configuration trees;
- test modules;
- pipeline files;
- shell/tool directories;
- database/query directories.

Output:

`brain/02-deployable-modules.md`

Do not ask for business interpretation yet.

## Phase B — build and dependency model

Inspect:

- root `pom.xml` / Gradle files;
- module dependencies;
- Spring Boot version;
- Java version;
- Neo4j driver / Spring Data Neo4j versions;
- Kafka dependencies;
- Cucumber/Mockito/Gatling versions;
- internal libraries;
- Azure Artifacts references.

Output:

`brain/01-system-overview.md`

`brain/15-external-integrations.md`

## Phase C — high-value endpoints first

Do **not** reverse engineer every endpoint equally.

You already know that a small group of endpoints dominates runtime traffic. Start with those endpoints.

For each high-volume endpoint trace:

```text
HTTP endpoint
  -> controller/resource
  -> service/facade
  -> business rule
  -> graph/repository query
  -> nodes and relationships
  -> external Check calls
  -> response mapping
  -> Cucumber scenarios
  -> unit/integration tests
  -> performance/Gatling scenario
```

Create one compact file per endpoint or endpoint family.

Example:

```text
brain/endpoints/subscription-read.md
brain/endpoints/outlook-party-read.md
```

This gives the project brain useful knowledge early instead of spending days documenting rarely used code.

## Phase D — business workflow tracing

Reverse engineer separately:

- customer;
- worker-on-behalf;
- worker-only;
- business access profile;
- private access profile.

Each workflow should explain:

- entry endpoint;
- identity/context inputs;
- authorization/Check checks;
- external worker Check dependency;
- graph lookups;
- decision points;
- output;
- negative/error cases;
- relevant Cucumber scenarios.

## Phase E — synchronization module

Trace Kafka notification handling:

```text
Kafka topic/event
 -> listener/consumer
 -> deserialization
 -> validation
 -> transformation
 -> domain service
 -> Neo4j writes/deletes/updates
 -> retry/DLQ/error strategy
 -> metrics/logging
```

The key purpose is to discover **how the graph is created and maintained**, because this is often more informative than reading read queries alone.

## Phase F — Export API

Trace:

- consumers;
- endpoint contracts;
- query/fetch path;
- pagination/chunking;
- export transformation;
- file format;
- temporary storage;
- large-data safeguards;
- security/Checks;
- performance tests.

## Phase G — shell utilities and incident tooling

Catalog each script by purpose rather than dumping script contents into the brain.

Example:

```markdown
Name: find-missing-subscription.sh
Purpose: incident investigation
Inputs: ...
Reads: ...
Writes: ...
Network access: ...
Dangerous operations: ...
Related runbook: ...
```

This makes utility knowledge searchable without consuming model context.

---

# 7. Neo4j reverse engineering strategy

Do not feed a 3-GB Neo4j database to the LLM.

The model does not need the entire dataset to understand the schema and business graph.

Use a three-source approach.

## 7.1 Source 1 — code-derived graph model

Devstral scans:

- `@Node` / entity annotations;
- relationship annotations;
- custom Cypher queries;
- repositories;
- mapping layers;
- transaction boundaries;
- graph traversal services;
- indexes/constraints setup;
- database migrations/setup scripts.

Generate candidate:

- node labels;
- relationship types;
- direction;
- cardinality;
- important properties;
- indexes;
- constraints;
- common query patterns.

Mark this as **CODE-DERIVED**.

## 7.2 Source 2 — graph-derived schema facts

Run safe, bounded Neo4j introspection queries locally.

Examples of useful categories:

- list labels;
- list relationship types;
- list indexes/constraints;
- count nodes by label;
- count relationships by type;
- sample a very small number of paths;
- inspect property keys/types;
- inspect query plans for selected high-volume queries.

Do not export millions of records into LLM context.

Instead create summaries such as:

```json
{
  "label": "Subscription",
  "approx_count": 123456,
  "relationships": [
    {"type":"HAS_OUTLOOK_PARTY","direction":"OUT","sample_count":...}
  ],
  "important_properties": ["..."],
  "indexes": ["..."]
}
```

Give these compact summaries to Devstral/GPT-OSS.

## 7.3 Source 3 — runtime/query evidence

For the high-volume endpoints, capture:

- actual Cypher query templates;
- parameters conceptually, without sensitive values;
- `EXPLAIN` / `PROFILE` summaries where permitted;
- index usage;
- cardinality estimates;
- dominant traversal path;
- expensive operators;
- request frequency/importance tier.

This lets the project brain learn not only the graph, but **which part of the graph matters operationally**.

---

# 8. Build a graph dictionary

Create a machine-readable graph catalog, for example:

`graph/graph-catalog.yaml`

Conceptual structure:

```yaml
nodes:
  Subscription:
    purpose: ...
    properties:
      - id
      - type
    indexes:
      - ...
    evidence:
      - path/to/file

relationships:
  HAS_OUTLOOK_PARTY:
    from: Subscription
    to: OutlookParty
    cardinality: one-to-many
    purpose: ...
    evidence:
      - path/to/query

queries:
  findSubscriptionChecks:
    used_by:
      - endpoint-id
    nodes:
      - Subscription
      - OutlookParty
    relationships:
      - HAS_OUTLOOK_PARTY
    performance_tier: hot
```

This is far more useful to a local model than repeatedly reading dozens of repository classes.

---

# 9. Build an endpoint catalog weighted by business/runtime importance

Because traffic is highly concentrated, give every endpoint a priority tier.

Example:

```yaml
endpoint_id: subscription-get
business_capability: subscription Checks
traffic_tier: HOT
workflow_support:
  - customer
  - worker-on-behalf
  - worker-only
controller: ...
service: ...
neo4j_queries:
  - ...
external_calls:
  - worker-Check-api
cucumber_features:
  - ...
gatling_scenarios:
  - ...
```

Then your coding agent can retrieve:

```text
endpoint + workflow + graph path + coding conventions
```

instead of searching the entire application.

---

# 10. Reverse engineer Cucumber as functional evidence

Cucumber is extremely valuable because it often sits between business language and implementation.

Do not merely summarize feature files. Build a mapping.

For each feature/scenario capture:

- feature name;
- scenario name;
- tags;
- business capability;
- workflow type;
- endpoint;
- positive/negative case;
- step-definition class;
- fixtures/test-data builders;
- expected graph state;
- expected output;
- linked production code where discoverable.

Create:

`testing/cucumber-feature-map.yaml`

This becomes one of the best sources for Gemma when translating technical detail into functional language.

---

# 11. Reverse engineer Gatling and performance knowledge

Do not ask the LLM to interpret the complete 3-GB performance dataset.

Instead document:

- how the 3-GB database is produced;
- dataset-generation scripts/classes;
- node counts;
- relationship counts;
- skew/distribution assumptions;
- scenarios;
- request mix;
- concurrency model;
- duration/ramp strategy;
- thresholds;
- high-volume endpoints;
- expected latency/error targets;
- known bottlenecks.

Create:

`testing/gatling-model.md`

and:

`runtime/performance-profile.yaml`

This allows code-review agents to ask:

> “Is this change on a HOT endpoint? Does it add another graph traversal? Does the performance model need updating?”

That is exactly the kind of context ordinary coding assistants frequently miss.

---

# 12. Coding-convention extraction

Instead of manually writing every coding standard, make Devstral infer conventions from representative code, then make GPT-OSS verify them.

Sample several high-quality existing examples for:

- controllers;
- service classes;
- repositories;
- Neo4j queries;
- exceptions;
- DTOs/mappers;
- Mockito tests;
- Cucumber steps;
- Gatling scenarios;
- shell scripts.

Extract rules such as:

```text
Constructor injection used instead of field injection
Naming pattern for repository methods
Mockito strictness style
How test fixtures are created
Cucumber tag conventions
Cypher parameter naming
Logging conventions
Exception mapping rules
Shell script error handling
```

Store these in:

`brain/13-coding-conventions.md`

Before Devstral writes code, this file becomes mandatory context.

---

# 13. Chunking strategy for a 32K context model

Do not define chunks purely by number of characters.

Chunk by **semantic responsibility**.

A useful task packet contains:

```text
TASK PACKET
  1 task
  1 mental-model summary
  1–3 business rules
  1 endpoint/workflow map
  5–15 source/test files or excerpts
  current review findings
```

The agent should never receive “the next 32K characters of the repository.” That destroys relationships across files.

Good chunk:

```text
SubscriptionController
SubscriptionService
SubscriptionRepository
relevant custom Cypher
DTO/mapper
3 related unit tests
2 related Cucumber scenarios
endpoint mental-model page
```

Bad chunk:

```text
files 1–47 alphabetically until token limit
```

---

# 14. Use discovery passes before deep reading

For every component, use three passes.

## Pass 1 — index

Read filenames, symbols, annotations, imports, public method signatures.

Output a dependency candidate map.

## Pass 2 — focused read

Read only high-value files selected from Pass 1.

## Pass 3 — verification

Search for callers, tests, configuration and graph queries to verify conclusions.

This saves enormous context compared with reading every file in full.

---

# 15. Create compact machine-readable indexes

Markdown is good for people, but YAML/JSON is better for retrieval.

Maintain indexes such as:

- `symbols.yaml`
- `endpoints.yaml`
- `modules.yaml`
- `graph-catalog.yaml`
- `workflows.yaml`
- `external-integrations.yaml`
- `test-map.yaml`

A task can first query the index and then open only the referenced detailed files.

This is effectively a lightweight deterministic retrieval system without needing a full vector database on day one.

---

# 16. Do you need RAG immediately?

No.

Start with **structured file-based retrieval** because:

- your laptop resources are limited;
- the repository structure itself provides excellent metadata;
- coding tasks need exact files, not only semantically similar text;
- you need traceability;
- debugging retrieval errors is easier.

A future phase can add embeddings/vector search if the brain grows very large, but do not introduce that complexity before proving the basic local-model quality.

---

# 17. Recommended retrieval hierarchy

When implementing a task, retrieve context in this order:

```text
1. task/current requirement
2. business capability/workflow
3. endpoint catalog
4. graph/query catalog
5. coding conventions
6. relevant source symbols/files
7. related tests
8. current git diff/review
```

This hierarchy makes the model understand **why** before it sees **how**.

---

# 18. Detailed first-time reverse-engineering workflow

The following is the recommended sequence for your first experiment.

## Run 1 — Devstral repository inventory

Goal: create only structural inventory.

Prompt concept:

```text
Do not implement code.
Inventory this repository and create module/package/deployable-component maps.
Read build descriptors and application entry points first.
Do not infer business behavior yet.
Persist findings under .ai-local/brain and .ai-local/modules.
Every claim must contain file evidence.
```

Stop after the inventory.

Unload Devstral.

## Run 2 — GPT-OSS inventory audit

Verify:

- missed modules;
- incorrect deployment assumptions;
- build dependency mistakes;
- shared/common packages incorrectly classified.

Persist review.

Unload GPT-OSS.

## Run 3 — Devstral corrects inventory

Verify reviewer comments against code and update only valid findings.

Unload.

## Run 4 — Devstral graph extraction

Inspect graph entities, repositories, Cypher, indexes/constraints.

Build graph catalog.

Unload.

## Run 5 — run bounded Neo4j introspection locally

Use approved local tooling to create compact schema/statistics files.

Do not send raw 3-GB data to a model.

## Run 6 — GPT-OSS graph audit

Compare code-derived graph vs graph-derived facts.

Flag:

- relationships existing in DB but absent from code model;
- query assumptions;
- direction/cardinality errors;
- unused/legacy labels;
- inconsistent naming.

## Run 7 — high-volume endpoint reverse engineering

Use Devstral one endpoint family at a time.

Create endpoint traces.

## Run 8 — Cucumber mapping

Map high-volume endpoint behavior to Cucumber scenarios.

## Run 9 — workflow mapping

Map customer / worker-on-behalf / worker-only and business/private access profiles.

## Run 10 — Gemma enrichment

Only now give Gemma:

- verified technical summaries;
- normalized functional documentation;
- Cucumber mappings;
- runtime endpoint importance.

Ask for human-readable project brain and terminology map.

This staged approach gives you useful output after each session and does not require all models simultaneously.

---

# 19. Model loading sequence and memory discipline

Use exactly one model at a time.

Recommended operational pattern:

```text
lms unload --all
load Devstral @ 32768
perform one bounded phase
save output to disk
exit
lms unload --all

load GPT-OSS @ 32768
review saved artifacts
save findings
exit
lms unload --all

load Devstral @ 32768
verify/fix findings
save
exit
lms unload --all

load Gemma @ 32768
synthesize verified docs
save
exit
lms unload --all
```

Do not keep model A loaded while starting model B.

Do not use concurrent subagents initially.

Do not increase context because one task became too large; split the task instead.

---

# 20. How to keep context small during code implementation

Before implementation, create a compact task package under:

`tasks/current/`

Recommended files:

```text
task.md
business-context.md
technical-context.md
files-to-inspect.txt
review.md
verification.md
```

The task agent reads the project brain first and then opens only the listed source files.

After implementation, the reviewer reads:

- task;
- relevant brain pages;
- `git diff`;
- surrounding files only where necessary;
- tests/build output.

The reviewer should not re-read the full repository.

---

# 21. Feature implementation pipeline

For every new coding task use this process.

```text
A. Normalize requirement
B. Retrieve project-brain context
C. Devstral analyzes existing implementation
D. Devstral writes plan
E. Human checks plan for major misunderstanding
F. Devstral implements
G. Local compiler/tests/Cucumber run
H. Devstral fixes direct failures
I. GPT-OSS reviews diff independently
J. Devstral verifies review comments
K. Devstral fixes valid findings
L. Tests run again
M. Gemma updates project brain and creates readable summary
```

Never skip step C. The single biggest source of random generated code is implementing before reading existing patterns.

---

# 22. Objective completion gates

Do not stop because the models agree.

Use objective gates:

```text
Compilation                                  PASS
Relevant unit tests                         PASS
Relevant integration/Cucumber tests         PASS
Static/style checks                         PASS
Unexpected git changes                      NONE
GPT-OSS BLOCKER findings                    0
GPT-OSS HIGH findings                       0
Project-brain update                        COMPLETE
Evidence links                              PRESENT
```

For performance-sensitive endpoints add:

```text
Gatling/performance impact considered       YES
Neo4j query plan impact considered          YES
Additional traversal/query introduced       DOCUMENTED
```

---

# 23. High-volume endpoint rule

Because a very small endpoint subset carries most calls, any code change affecting a HOT endpoint should automatically trigger a stronger review checklist.

The mental model should tag endpoints:

```text
HOT
WARM
COLD
```

For HOT endpoints the reviewer should always evaluate:

- number of Neo4j round trips;
- query shape;
- indexes;
- result cardinality;
- object allocations/mapping;
- external API calls;
- caching;
- logging volume;
- backward compatibility;
- Gatling scenario relevance.

This is a concrete way for your project brain to improve code quality beyond a generic cloud assistant.

---

# 24. External worker-modeling

Treat external worker Checks as a first-class integration.

Document:

- which workflows call it;
- input contract;
- output contract;
- timeout/retry behavior;
- failure semantics;
- caching;
- fallback behavior;
- circuit breaker if present;
- how results affect Neo4j queries/Check decisions;
- tests/mocks;
- operational dashboards/logging.

This prevents an agent from making a “simple local code change” that accidentally violates an external Check workflow.

---

# 25. Kafka sync knowledge model

For every important Kafka notification type create a mapping:

```text
notification type
 -> consumer
 -> parser/schema
 -> validation
 -> domain operation
 -> graph mutations
 -> affected node types
 -> affected relationships
 -> idempotency rule
 -> retry/error handling
 -> relevant Cucumber/integration tests
```

This becomes critical when coding agents modify read APIs because they must understand **how the data got into Neo4j**.

---

# 26. Functional-document ingestion

Do not give raw Confluence/PDF/PPT exports directly to the final mental-model writer and ask it to “understand everything.”

Use a normalization pipeline.

```text
source document
 -> extract locally
 -> document-specific summary
 -> terminology/rules/events/actors table
 -> GPT-OSS consistency check against technical model
 -> Gemma synthesis
```

For every functional source document capture:

- document title;
- date/version;
- business actors;
- terms;
- rules;
- workflow steps;
- exceptions;
- referenced APIs/processes;
- conflicts with current implementation;
- unresolved questions.

Do not erase contradictions. Preserve them as explicit open questions.

---

# 27. PDF strategy

For PDFs:

1. Prefer local extraction/text parsing when the PDF is textual.
2. Split very large PDFs by chapter/section.
3. Feed one logical section at a time.
4. Create section summaries on disk.
5. Build an index that points to the source file and page range.
6. Merge only summarized facts into the project brain.

For scanned/image-heavy PDFs, use a vision-capable local model only if your exact LM Studio model build supports it reliably. Otherwise export relevant pages as images and benchmark vision quality separately.

Do not process a 300-page PDF in one model call.

---

# 28. PowerPoint strategy

For PPT/PPTX:

- export to PDF for text-heavy slides;
- export selected diagrams/slides as PNG for visual interpretation;
- process deck section by section;
- store a slide-index summary such as:

```text
Slides 1–5: architecture overview
Slides 6–11: Check workflows
Slides 12–17: graph model
```

Then Gemma can consume the normalized summaries instead of the raw deck repeatedly.

---

# 29. Image/diagram strategy

For architecture diagrams, screenshots, and graph diagrams:

- provide one diagram at a time;
- ask the model to extract entities, arrows, labels, conditions and boundaries;
- ask it to label uncertain visual interpretations;
- verify extracted relationships against code or documentation;
- never promote image-only inference directly to FACT without corroboration when it affects code behavior.

Store the result next to the source image reference.

---

# 30. Confluence strategy

For the initial experiment, do not build a Confluence crawler.

Export or copy only high-value pages:

- architecture;
- domain terminology;
- Check workflows;
- API behavior;
- graph/data model;
- performance/runbooks;
- incident procedures.

Normalize them locally and add them to the brain.

Only after proving value should you consider an approved Confluence integration.

---

# 31. Public documentation retrieval

When the local model lacks current framework knowledge, retrieve only generic official documentation.

Good requests:

```text
Spring Boot 4 transaction management documentation
Neo4j Java Driver query API documentation
Mockito current strict stubbing documentation
Cucumber JVM parameter types documentation
Java 25 language feature documentation
```

Bad external requests:

```text
Here is our internal SubscriptionService.java; why does it fail?
Here is our Neo4j query and schema; optimize it on the Internet.
```

Public documentation should enter your system as a local research note under:

`research/`

Then Devstral can use the local note while company code remains local.

---

# 32. Executable-download policy

The coding agent may generate local code/scripts itself, but must never automatically fetch and execute public code.

The rule should be:

```text
Need external executable/script?
        |
        v
STOP AUTOMATION
        |
        v
Report:
- exact tool/script
- reason required
- version
- official source URL
- expected checksum/signature if known
        |
        v
Human reviews and manually acquires it
```

Company-approved artifact repositories remain separate and may continue functioning according to your corporate build configuration.

---

# 33. Avoid a common mistake: “project brain” is not generated prose

A useful project brain must contain:

- facts;
- evidence;
- links;
- structured indexes;
- traceability;
- confidence;
- open questions;
- current version/update dates.

It should not be a collection of fluent explanations that cannot be traced to code.

The value is **retrievability and correctness**, not length.

---

# 34. Introduce confidence levels

Every inferred relationship should have one of:

```text
FACT — directly verified in code/data/tests
CORROBORATED — supported by multiple independent sources
INFERENCE — likely but not directly proven
ASSUMPTION — currently assumed
UNKNOWN — insufficient evidence
CONFLICT — sources disagree
```

This gives downstream coding agents a way to decide whether they should verify something before coding.

---

# 35. Freshness and invalidation

The project brain can become stale. Add metadata to every brain page:

```yaml
last_verified_commit: abc123
last_verified_date: 2026-08-28
sources:
  - ...
owner_area: api
confidence: high
```

After a code change, identify which brain pages are affected and update only those pages.

Do not regenerate the entire brain after every commit.

---

# 36. Incremental update rule

For each merged feature/change:

```text
Git diff
 -> identify affected concepts
 -> update only linked brain nodes
 -> reviewer verifies update
```

Examples:

- new Neo4j relationship → graph catalog + endpoint traces + workflow maps;
- new endpoint → endpoint catalog + Cucumber + runtime tier;
- changed Kafka notification → sync model + graph mutation map;
- changed coding convention → coding-conventions page.

This keeps maintenance affordable.

---

# 37. Recommended task prompt for Devstral reverse engineering

Use a reusable prompt similar to:

```text
ROLE: reverse-engineering engineer.

GOAL:
Understand one bounded capability without modifying production code.

REQUIRED PROCESS:
1. Read the relevant project-brain index first.
2. Search repository symbols rather than opening unrelated files.
3. Identify entry points.
4. Trace callers/callees.
5. Trace persistence/Neo4j queries.
6. Find relevant tests/Cucumber scenarios.
7. Record only evidence-supported conclusions.
8. Mark uncertainty explicitly.
9. Persist compact results under .ai-local.
10. Stop when the requested capability is fully mapped; do not expand scope automatically.

CONTEXT BUDGET:
Prefer summaries and targeted excerpts. Do not load unrelated source files.
```

---

# 38. Recommended task prompt for Devstral implementation

```text
ROLE: implementation engineer.

BEFORE CODING:
1. Read task.md.
2. Read linked project-brain pages.
3. Inspect existing source patterns.
4. Inspect relevant tests.
5. Confirm impacted Neo4j queries/relationships.
6. Confirm whether the endpoint is HOT/WARM/COLD.
7. Produce a short implementation plan.

IMPLEMENTATION:
Follow existing project naming, Spring, Neo4j, Mockito and Cucumber conventions.
Never invent an API if uncertain.
Use current source/build dependencies as the source of truth.

VERIFICATION:
Compile, run targeted tests, inspect diff, and report unresolved risks.
```

---

# 39. Recommended GPT-OSS reviewer prompt

```text
ROLE: Independent adversarial technical reviewer.

INPUTS:
- task requirement
- relevant brain pages
- current git diff
- test/build results

DO NOT assume the implementer is correct.

CHECK:
- business-rule mismatch
- wrong workflow handling
- Neo4j traversal/query mistakes
- transaction semantics
- concurrency/idempotency
- external Check integration
- backward compatibility
- HOT-endpoint performance impact
- error/edge cases
- Mockito/Cucumber coverage
- style/pattern consistency
- unsupported framework API usage

OUTPUT:
BLOCKER / HIGH / MEDIUM / LOW / INFORMATIONAL findings.
Every BLOCKER/HIGH must contain evidence and a concrete fix direction.
```

---

# 40. Recommended Gemma synthesis prompt

```text
ROLE: Project mental-model writer.

INPUTS:
- verified technical model
- functional documentation
- Cucumber/business examples
- runtime importance
- reviewer-approved facts

GOAL:
Create a human-readable explanation without losing traceability.

CONNECT:
actor -> workflow -> business rule -> endpoint -> service -> Neo4j -> external integration -> test.

RULES:
- Do not invent missing links.
- Preserve FACT/INFERENCE/UNKNOWN status.
- Explain domain terms before implementation details.
- Keep detailed technical evidence linked rather than copied everywhere.
```

---

# 41. Do not use the models in a free-running loop

Avoid:

```text
Devstral -> GPT-OSS -> Devstral -> GPT-OSS -> ... forever
```

Use maximum bounded review cycles, for example:

```text
Implementation attempt
 -> test
 -> review
 -> one correction cycle
 -> test
 -> second review only if BLOCKER/HIGH remains
 -> human escalation/cloud escalation if still unresolved
```

The objective is reliable engineering, not model consensus.

---

# 42. Cloud Copilot escalation strategy

Your local system does not need to replace premium cloud models completely.

Use cloud Copilot only when one of these occurs:

- local implementation fails repeatedly;
- build/test failures remain unexplained after bounded attempts;
- task needs reasoning beyond local model quality;
- project brain has conflicting/insufficient facts;
- a very large unfamiliar cross-cutting feature requires broader reasoning;
- you explicitly want a premium second opinion.

Before escalation, prepare a compact packet:

```text
requirement
verified mental-model summary
relevant files
current diff
local reviewer findings
failing test/build output
```

This reduces cloud token use dramatically compared with asking the cloud model to rediscover the repository from zero.

---

# 43. What “Claude-like quality” realistically requires

A 24B/20B/31B local model will not consistently match a frontier cloud model in raw reasoning.

The way to close the gap is through system quality:

```text
better project context
+ evidence-linked project brain
+ retrieval of only relevant files
+ coding conventions
+ compiler
+ tests
+ Cucumber
+ Neo4j verification
+ independent reviewer
+ performance awareness
+ bounded correction loops
```

The local model does not need to be stronger than Claude if the **local engineering system supplies context Claude usually has to rediscover**.

For routine project-specific changes, this can substantially improve consistency and reduce premium-credit usage.

---

# 44. Performance safeguards for the laptop

Keep these rules initially:

- one model loaded at a time;
- 32,768-token context;
- no concurrent/background subagents;
- use the MLX/quantized model that leaves RAM headroom;
- unload model between pipeline stages;
- persist handoffs to files instead of carrying huge chats;
- keep brain files compact;
- do not process raw 3-GB Neo4j data through the model;
- do not load entire functional-document collections at once;
- split by endpoint/workflow/module.

Monitor macOS Memory Pressure rather than trying to keep “Memory Used” artificially low.

---

# 45. Suggested first experiment scope

Do not begin by reverse engineering the entire repository.

Choose one high-value vertical slice, preferably one of the highly used subscription or outlook-party endpoints.

Reverse engineer the complete slice:

```text
functional purpose
 -> endpoint
 -> flow (customer / worker-on-behalf / worker-only)
 -> Java implementation
 -> Neo4j query and graph path
 -> external worker Check dependency
 -> Cucumber
 -> Mockito/integration tests
 -> Gatling/performance relevance
```

Then ask the three-model pipeline to build the brain for only that slice.

Finally give Devstral one realistic small code change in the slice and compare its output with your normal Copilot workflow.

This gives you a meaningful answer in days rather than spending weeks building a brain before proving it helps.

---

# 46. Suggested evaluation scorecard

For the selected vertical slice measure:

```text
Repository discovery accuracy
Graph relationship accuracy
Business workflow accuracy
Correct files chosen for implementation
Implementation compile success
Unit-test success
Cucumber success
Number of hallucinated APIs
Number of reviewer defects
Manual corrections required
Time to completion
Peak memory pressure
Cloud escalation required?
Cloud credits saved
```

Repeat for:

- 5 small tasks;
- 5 medium tasks;
- 2–3 difficult tasks.

This tells you whether the experiment is economically and technically worthwhile.

---

# 47. Minimum viable project brain for the first slice

Before asking Devstral to implement anything, the selected vertical slice should have these artifacts:

```text
endpoint.md
workflow.md
graph-path.md
technical-trace.md
cucumber-map.md
test-map.md
coding-conventions.md
performance-tier.md
open-questions.md
```

That is enough to test whether persistent local context improves code quality.

Do not wait for a perfect whole-system model.

---

# 48. Final operating model

Your long-term working method should become:

```text
                    PROJECT BRAIN
                         |
       +-----------------+------------------+
       |                 |                  |
 business model     technical model     runtime model
       |                 |                  |
       +-----------------+------------------+
                         |
                         v
                    TASK RETRIEVER
                         |
                         v
                     DEVSTRAL
                    IMPLEMENTER
                         |
                         v
                 COMPILER / TESTS
                         |
                         v
                     GPT-OSS
                     REVIEWER
                         |
                   issues / clean
                     |       |
                     v       v
                 DEVSTRAL   GEMMA
                    FIX     SYNTHESIS
                     |       |
                     +---+---+
                         |
                         v
                  UPDATED PROJECT BRAIN
```

The project brain—not the chat history—is the durable knowledge source.

---

# 49. Recommended next actions

Start with one vertical slice and complete these actions in order:

1. Create `.ai-local` structure.
2. Add it to `.git/info/exclude` if it must remain local.
3. Inventory modules with Devstral.
4. Select one HOT endpoint family.
5. Reverse engineer its code path with Devstral.
6. Extract the associated graph path from code.
7. Run bounded Neo4j introspection to verify the graph.
8. Map Cucumber and tests.
9. Map the workflow/business terminology.
10. Have GPT-OSS audit the technical model.
11. Have Devstral verify/correct reviewer findings.
12. Have Gemma create the human-readable traceability model.
13. Give Devstral one small real change using only the project brain plus relevant files.
14. Run compiler/tests.
15. Have GPT-OSS review the diff.
16. Measure quality and compare against your normal Copilot result.
17. Expand to the next endpoint only if the experiment produces useful results.

---

# 50. Bottom line

The optimal approach for is **not** to squeeze the entire monolith, Neo4j database, test suite and functional documentation into a larger context window.

The optimal approach is to create a **verified, structured, incremental project brain** that externalizes long-term knowledge and allows each 32K local-model session to work on one coherent slice.

If implemented carefully, the local models can handle a substantial amount of:

- reverse engineering;
- repository understanding;
- technical documentation;
- business/technical traceability;
- small and medium code changes;
- test generation;
- independent code review;
- project onboarding knowledge.

Premium cloud models then become an escalation mechanism rather than the default repository-discovery mechanism.

For a project with high endpoint concentration, graph-heavy persistence, strong Cucumber coverage and identifiable workflow families, this approach is particularly suitable because the application naturally decomposes into retrievable business/technical slices.

