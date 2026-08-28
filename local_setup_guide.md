# Local LLM + GitHub Copilot CLI Setup Guide
## Devstral + GPT-OSS + Gemma 4 on LM Studio for a 48 GB Apple Silicon Mac

**Purpose:** Configure three downloaded local LLMs in LM Studio and use them through GitHub Copilot CLI as a sequential engineering pipeline:

1. **Devstral** — coding, reverse engineering, implementation, Java/Spring/Neo4j/Cucumber/Mockito/shell work.
2. **GPT-OSS** — independent technical review, architecture critique, technical-document refinement.
3. **Gemma 4 31B QAT** — human-readable technical/functional synthesis and project mental-model enrichment.

The system is designed for a corporate laptop where:
- proprietary source code must not be sent to public internet services;
- official public technical documentation may be fetched;
- company artifact repositories are allowed;
- arbitrary scripts, executables, installers, binaries, packages, or Docker images must not be automatically downloaded and executed;
- any external script/tool acquisition requires explicit human review and manual approval;
- only one local model should be resident in memory at a time.

---

# 1. Target Architecture

```text
                         YOUR MACBOOK

                    GitHub Copilot CLI
                           |
            +--------------+--------------+
            |              |              |
            |              |              |
        CODING         REVIEWING       DOCUMENTATION
        SESSION        SESSION         SESSION
            |              |              |
        Devstral       GPT-OSS          Gemma 4
            \              |              /
             \             |             /
              +------- LM Studio -------+
                       localhost
                    127.0.0.1:1234
                           |
                    One model loaded
                         at a time
```

The local path is:

```text
Repository -> Copilot CLI -> 127.0.0.1 -> LM Studio -> Local model
```

The cloud path remains separate and deliberate:

```text
Repository -> normal approved GitHub Copilot CLI -> GitHub-hosted model
```

Do not mix both modes in the same launcher.

---

# 2. Machine Profile and Recommended Limits

This guide assumes:

- Apple Silicon MacBook Pro
- M4 Pro
- 48 GB unified memory
- IntelliJ IDEA and normal corporate applications remain running
- LM Studio already installed
- Devstral already downloaded
- GPT-OSS already downloaded
- Gemma 4 31B QAT already downloaded
- GitHub Copilot CLI already approved by the organization

## Recommended local-model settings

Use initially:

```text
Context window: 32768 tokens
Loaded models: 1
Idle unload TTL: 10-15 minutes
Auto-Evict/JIT: enabled
LM Studio bind: 127.0.0.1 only
CORS: disabled unless specifically required
Remote MCP: disabled initially
```

Do not start with 128K/256K context.

GitHub recommends large context windows for best BYOK results, but the project-memory strategy in this guide deliberately compensates for the smaller local context.

---

# 3. Important Security Model

Use four layers.

```text
Layer 1: Local model
Layer 2: Copilot CLI permissions
Layer 3: Build/package repository configuration
Layer 4: Corporate firewall/proxy/EDR
```

Never rely only on an LLM instruction such as:

> Do not send source code to the internet.

The infrastructure should prevent or require approval for network-capable operations.

## Allowed

- Read company repository.
- Write company repository after normal Copilot approval.
- Send company repository content to `127.0.0.1:1234`.
- Run local Java/JDK.
- Run existing approved Maven/Gradle installations.
- Read an approved company artifact repository.
- Fetch public documentation from official documentation sites.
- Generate new code or a new shell script locally.
- Read already-existing local scripts.

## Not automatically allowed

- Download a `.sh` file.
- Download an executable.
- Download a package manager installer.
- `curl ... | sh`
- `curl ... | bash`
- `wget ... && chmod +x ...`
- Public Docker image pulls.
- Public npm/pip/brew installs.
- Remote MCP servers that have not been approved.
- Uploading repository code in a public search/web request.

If the agent needs such an item, it must stop and report:

```text
REQUIRED EXTERNAL RESOURCE
Name:
Version:
Reason:
Official URL:
Expected use:
Why existing local tools are insufficient:
```

You then inspect and acquire it manually.

---

# 4. Verify LM Studio CLI

Open macOS Terminal.

Run:

```bash
lms --help
```

Expected result: LM Studio CLI help is displayed.

If the command is unavailable, open LM Studio and enable/install the `lms` CLI from LM Studio.

Then run:

```bash
lms ls
```

This lists downloaded models.

Write down the **exact model key** for:

```text
Devstral
GPT-OSS
Gemma 4 31B QAT
```

Example only:

```text
mistralai/...
openai/...
google/...
```

Do not copy example IDs from the internet. Use exactly what your own `lms ls` prints.

---

# 5. Record Your Model Keys

Create a private local note:

```bash
mkdir -p ~/.local-ai
nano ~/.local-ai/model-keys.txt
```

Record:

```text
DEVSTRAL=<exact key from lms ls>
GPTOSS=<exact key from lms ls>
GEMMA=<exact key from lms ls>
```

Save:

- `Ctrl+O`
- Enter
- `Ctrl+X`

Do not commit this file to Git.

---

# 6. Confirm No Model Is Loaded

Run:

```bash
lms ps
```

Unload everything:

```bash
lms unload --all
```

Run again:

```bash
lms ps
```

The list should now be empty.

Use this command whenever you want to return to a clean memory state.

---

# 7. Estimate RAM Before Loading

For Devstral:

```bash
lms load --estimate-only "<DEVSTRAL_MODEL_KEY>" --context-length 32768
```

For GPT-OSS:

```bash
lms load --estimate-only "<GPTOSS_MODEL_KEY>" --context-length 32768
```

For Gemma:

```bash
lms load --estimate-only "<GEMMA_MODEL_KEY>" --context-length 32768
```

Record estimates in:

```bash
nano ~/.local-ai/model-memory.txt
```

Example structure:

```text
Devstral:
  Model:
  Context: 32768
  Estimated total:

GPT-OSS:
  Model:
  Context: 32768
  Estimated total:

Gemma:
  Model:
  Context: 32768
  Estimated total:
```

The goal is not to use all 48 GB. Leave meaningful RAM for:

- macOS;
- IntelliJ;
- Teams;
- Outlook;
- JVM;
- Maven/Gradle;
- Docker/company tools;
- filesystem cache.

---

# 8. Configure LM Studio Server

Open LM Studio.

Go to:

```text
Developer
-> Server
-> Server Settings
```

Use:

```text
Port: 1234
Serve on Local Network: OFF
Enable CORS: OFF
JIT model loading: ON
Auto unload unused JIT models: ON
Only keep last JIT-loaded model / Auto-Evict: ON
```

For the first experiment also use:

```text
Allow per-request MCPs: OFF
Allow servers from mcp.json: OFF
```

The important network setting is:

```text
127.0.0.1
```

Do not expose the server using:

```text
0.0.0.0
```

unless your security team explicitly wants network serving.

---

# 9. Start LM Studio Server

From Terminal:

```bash
lms server start --port 1234 --bind 127.0.0.1
```

Check status:

```bash
lms server status
```

LM Studio OpenAI-compatible API base URL:

```text
http://127.0.0.1:1234/v1
```

Test the model-list endpoint:

```bash
curl http://127.0.0.1:1234/v1/models
```

This `curl` command is a one-time manual localhost test you are deliberately running yourself. It does not contact the public internet.

You should see locally downloaded/available models when JIT model loading is enabled.

---

# 10. Test Devstral Directly Before Copilot

Unload all:

```bash
lms unload --all
```

Load Devstral manually:

```bash
lms load "<DEVSTRAL_MODEL_KEY>" \
  --identifier devstral-local \
  --context-length 32768 \
  --ttl 900
```

Verify:

```bash
lms ps
```

Start a direct chat:

```bash
lms chat devstral-local
```

Test prompt:

```text
Create a minimal Java 21 Spring service using constructor injection and
write a Mockito unit test. Do not use any external tools.
```

Exit the chat.

Unload:

```bash
lms unload --all
```

---

# 11. Test GPT-OSS Directly

```bash
lms load "<GPTOSS_MODEL_KEY>" \
  --identifier gptoss-local \
  --context-length 32768 \
  --ttl 900
```

Verify:

```bash
lms ps
```

Run:

```bash
lms chat gptoss-local
```

Test prompt:

```text
Review this architecture conceptually:
REST Controller -> Service -> Neo4j Repository.
List likely design risks, transaction concerns and test responsibilities.
```

Exit.

Unload:

```bash
lms unload --all
```

---

# 12. Test Gemma Directly

```bash
lms load "<GEMMA_MODEL_KEY>" \
  --identifier gemma-local \
  --context-length 32768 \
  --ttl 900
```

Run:

```bash
lms chat gemma-local
```

Prompt:

```text
Explain Spring dependency injection to a junior developer while preserving
technical accuracy. Include a simple example.
```

Exit.

Unload:

```bash
lms unload --all
```

At this point all three models have been validated independently.

---

# 13. Verify GitHub Copilot CLI

Run:

```bash
copilot --help
```

Then:

```bash
copilot help providers
```

GitHub Copilot CLI BYOK supports OpenAI-compatible providers.

LM Studio provides OpenAI-compatible endpoints.

---

# 14. First Manual Copilot-to-LM-Studio Test

Load Devstral:

```bash
lms unload --all

lms load "<DEVSTRAL_MODEL_KEY>" \
  --identifier devstral-local \
  --context-length 32768 \
  --ttl 900
```

Set the provider only in this terminal session:

```bash
export COPILOT_PROVIDER_TYPE=openai
export COPILOT_PROVIDER_BASE_URL=http://127.0.0.1:1234/v1
export COPILOT_MODEL=devstral-local
```

If LM Studio authentication is disabled, do not set an API key.

Run:

```bash
copilot
```

Start with:

```text
Read pom.xml and report:
1. Java version
2. Spring Boot version
3. Maven modules

Do not modify any files.
```

Confirm in LM Studio that Devstral is processing the request.

Exit Copilot.

Unload:

```bash
lms unload --all
```

---

# 15. Do Not Use Permanent Global BYOK Variables

Do not put these permanently at the top level of `.zshrc`:

```text
COPILOT_PROVIDER_BASE_URL
COPILOT_PROVIDER_TYPE
COPILOT_MODEL
```

Why:

You also need your normal company GitHub-hosted Copilot mode.

Instead, create explicit launch functions.

This gives you an obvious separation:

```text
copilot-devstral
copilot-gptoss
copilot-gemma
copilot-cloud
```

---

# 16. Back Up `.zshrc`

```bash
cp ~/.zshrc ~/.zshrc.backup-local-ai
```

Open:

```bash
nano ~/.zshrc
```

---

# 17. Add Exact Model Keys

At the bottom, add:

```bash
export LOCAL_DEVSTRAL_KEY="<DEVSTRAL_MODEL_KEY>"
export LOCAL_GPTOSS_KEY="<GPTOSS_MODEL_KEY>"
export LOCAL_GEMMA_KEY="<GEMMA_MODEL_KEY>"
```

These are local identifiers only.

---

# 18. Create a Helper to Ensure the LM Studio Server Is Running

Add:

```bash
ensure-lmstudio-server() {
    if ! lms server status >/dev/null 2>&1; then
        echo "Starting LM Studio server on localhost..."
        lms server start --port 1234 --bind 127.0.0.1
    fi
}
```

---

# 19. Devstral Launcher

Add:

```bash
copilot-devstral() {
    ensure-lmstudio-server

    echo "Unloading previous local LLM..."
    lms unload --all

    echo "Loading Devstral with 32K context..."
    lms load "$LOCAL_DEVSTRAL_KEY" \
        --identifier devstral-local \
        --context-length 32768 \
        --ttl 900 || return 1

    export COPILOT_PROVIDER_TYPE=openai
    export COPILOT_PROVIDER_BASE_URL=http://127.0.0.1:1234/v1
    export COPILOT_MODEL=devstral-local
    unset COPILOT_PROVIDER_API_KEY

    echo ""
    echo "MODE: LOCAL DEVSTRAL"
    echo "MODEL ENDPOINT: 127.0.0.1:1234"
    echo ""

    copilot "$@"

    echo "Unloading Devstral..."
    lms unload --all
}
```

Use:

```bash
copilot-devstral
```

---

# 20. GPT-OSS Launcher

Add:

```bash
copilot-gptoss() {
    ensure-lmstudio-server

    echo "Unloading previous local LLM..."
    lms unload --all

    echo "Loading GPT-OSS with 32K context..."
    lms load "$LOCAL_GPTOSS_KEY" \
        --identifier gptoss-local \
        --context-length 32768 \
        --ttl 900 || return 1

    export COPILOT_PROVIDER_TYPE=openai
    export COPILOT_PROVIDER_BASE_URL=http://127.0.0.1:1234/v1
    export COPILOT_MODEL=gptoss-local
    unset COPILOT_PROVIDER_API_KEY

    echo ""
    echo "MODE: LOCAL GPT-OSS"
    echo ""

    copilot "$@"

    echo "Unloading GPT-OSS..."
    lms unload --all
}
```

Use:

```bash
copilot-gptoss
```

---

# 21. Gemma Launcher

Add:

```bash
copilot-gemma() {
    ensure-lmstudio-server

    echo "Unloading previous local LLM..."
    lms unload --all

    echo "Loading Gemma 4 with 32K context..."
    lms load "$LOCAL_GEMMA_KEY" \
        --identifier gemma-local \
        --context-length 32768 \
        --ttl 900 || return 1

    export COPILOT_PROVIDER_TYPE=openai
    export COPILOT_PROVIDER_BASE_URL=http://127.0.0.1:1234/v1
    export COPILOT_MODEL=gemma-local
    unset COPILOT_PROVIDER_API_KEY

    echo ""
    echo "MODE: LOCAL GEMMA"
    echo ""

    copilot "$@"

    echo "Unloading Gemma..."
    lms unload --all
}
```

---

# 22. Normal Company Copilot Cloud Launcher

Add:

```bash
copilot-cloud() {
    echo "Unloading local models..."
    lms unload --all >/dev/null 2>&1

    unset COPILOT_PROVIDER_TYPE
    unset COPILOT_PROVIDER_BASE_URL
    unset COPILOT_PROVIDER_API_KEY
    unset COPILOT_MODEL
    unset COPILOT_OFFLINE

    echo ""
    echo "MODE: NORMAL COMPANY GITHUB COPILOT"
    echo ""

    copilot "$@"
}
```

This is the explicit cloud escalation path.

---

# 23. Save `.zshrc`

In Nano:

```text
Ctrl+O
Enter
Ctrl+X
```

Reload:

```bash
source ~/.zshrc
```

Verify functions:

```bash
type copilot-devstral
type copilot-gptoss
type copilot-gemma
type copilot-cloud
```

---

# 24. Verify Sequential Memory Behavior

Run:

```bash
copilot-devstral
```

Exit.

Then:

```bash
lms ps
```

Expected: Devstral is unloaded by the launcher.

Now:

```bash
copilot-gptoss
```

Exit.

Check:

```bash
lms ps
```

Then:

```bash
copilot-gemma
```

At no point should multiple heavyweight models be intentionally resident.

---

# 25. Optional JIT/Auto-Evict Mode

Once manual launchers work reliably, you can test LM Studio JIT + Auto-Evict.

In LM Studio Server Settings:

```text
Just in Time Model Loading: ON
Auto Unload Unused JIT Models: ON
Only Keep Last JIT Loaded Model: ON
```

JIT requests can automatically load the requested model and Auto-Evict can unload the previous JIT-loaded model.

For the first experiment, explicit `lms unload --all` remains easier to understand and troubleshoot.

---

# 26. Create a Private Project-Brain Directory

From the repository root:

```bash
mkdir -p .ai-local/brain
mkdir -p .ai-local/technical
mkdir -p .ai-local/functional
mkdir -p .ai-local/reviews
mkdir -p .ai-local/tasks
mkdir -p .ai-local/research
mkdir -p .ai-local/benchmarks
```

Structure:

```text
.ai-local/
├── brain/
├── technical/
├── functional/
├── reviews/
├── tasks/
├── research/
└── benchmarks/
```

---

# 27. Do Not Commit the Brain Unless Approved

Use your local repository exclusion file rather than modifying team `.gitignore`:

```bash
echo ".ai-local/" >> .git/info/exclude
```

Verify:

```bash
git status
```

`.ai-local/` should not appear as an untracked folder.

---

# 28. Create Core Brain Files

```bash
touch .ai-local/brain/00-index.md
touch .ai-local/brain/01-system-overview.md
touch .ai-local/brain/02-module-map.md
touch .ai-local/brain/03-domain-model.md
touch .ai-local/brain/04-code-map.md
touch .ai-local/brain/05-neo4j-model.md
touch .ai-local/brain/06-api-map.md
touch .ai-local/brain/07-business-rules.md
touch .ai-local/brain/08-cucumber-model.md
touch .ai-local/brain/09-testing-model.md
touch .ai-local/brain/10-performance-model.md
touch .ai-local/brain/11-coding-standards.md
touch .ai-local/brain/12-shell-utilities.md
touch .ai-local/brain/13-pipelines.md
touch .ai-local/brain/14-traceability.md
touch .ai-local/brain/15-open-questions.md
```

The project brain is the durable memory.

The 32K context window is only temporary working memory.

---

# 29. Chunking Rule

Never prompt:

```text
Read my complete repository and understand everything.
```

Use:

```text
Discover -> select -> read -> extract -> save -> unload context -> continue
```

Recommended unit of work:

```text
one module
or
one endpoint
or
one business workflow
or
one Neo4j query family
or
one testing layer
```

Keep each individual brain file concise.

Prefer links to evidence over copied source code.

Example:

```markdown
## Permission lookup

Implementation:
- `src/main/java/.../PermissionService.java`
- `src/main/java/.../PermissionRepository.java`

Neo4j:
- Reads `Agreement`
- Traverses `HAS_PARTY`
- Filters active grant relationships

Tests:
- `src/test/.../PermissionServiceTest.java`
- `src/test/resources/features/...feature`

Confidence: VERIFIED
```

Do not copy hundreds of lines of source into the mental model.

---

# 30. Create User-Level Copilot Agents

Use user-level agents so the experiment does not modify the repository:

```bash
mkdir -p ~/.copilot/agents
```

Agents will inherit whichever model your launcher selected.

Do not hard-code cloud model names in these agent files.

---

# 31. Devstral Coding Agent

Create:

```bash
nano ~/.copilot/agents/local-coder.agent.md
```

Contents:

```markdown
---
name: local-coder
description: Local code analysis, reverse engineering and implementation agent.
infer: false
---

You are the primary local engineering agent.

Primary technologies:
- Java 21 and Java 25 readiness
- Spring Boot 4.x
- Neo4j
- Cucumber
- Mockito
- Gatling
- Maven
- shell scripting
- Kafka
- Azure/GitHub build configuration

Repository data is confidential.

Before implementing:
1. Read relevant `.ai-local/brain` files.
2. Search the actual repository.
3. Inspect existing patterns.
4. Inspect tests.
5. Identify affected modules.
6. Write an implementation plan.
7. Do not invent project behavior.

For reverse engineering:
- work in small bounded slices;
- record file-path evidence;
- record Neo4j nodes and relationships;
- record configuration;
- record APIs and consumers;
- record Kafka flows;
- record Cucumber/Gatling/Mockito evidence;
- mark uncertainty explicitly.

Never transmit source code or internal details to a public website.

Public internet retrieval is only for generic official technical documentation.

Never automatically download or execute:
- shell scripts;
- executables;
- installers;
- arbitrary archives;
- public packages;
- Docker images.

If such a resource is required, stop and report:
name, version, official URL, purpose and reason.

Generate local shell scripts only when requested.
Do not fetch a shell script from the internet.

After code changes:
- inspect diff;
- compile;
- run focused unit tests;
- run applicable Cucumber/integration tests;
- report unresolved failures.
```

Restart Copilot after changing agent definitions.

---

# 32. GPT-OSS Review Agent

Create:

```bash
nano ~/.copilot/agents/local-reviewer.agent.md
```

Contents:

```markdown
---
name: local-reviewer
description: Independent technical reviewer for code, architecture and reverse-engineered documentation.
infer: false
---

You are an independent adversarial technical reviewer.

Do not assume the implementation or technical documentation is correct.

Review:
- Java correctness;
- Spring Boot architecture;
- dependency boundaries;
- Neo4j graph semantics;
- Cypher correctness;
- transaction boundaries;
- concurrency;
- null handling;
- error handling;
- API compatibility;
- Kafka processing;
- security;
- Cucumber coverage;
- Mockito quality;
- Gatling/performance implications;
- configuration;
- shell utilities;
- pipeline impact;
- maintainability;
- hidden assumptions;
- hallucinated APIs.

Do not modify production code unless explicitly asked.

Write findings to:
`.ai-local/reviews/current-review.md`

Use severity:
BLOCKER
HIGH
MEDIUM
LOW
INFORMATIONAL

For BLOCKER/HIGH include:
- evidence;
- file/location;
- explanation;
- proposed correction.

Verify previous-model claims against repository evidence.
```

---

# 33. Gemma Mental-Model Agent

Create:

```bash
nano ~/.copilot/agents/local-mental-model.agent.md
```

Contents:

```markdown
---
name: local-mental-model
description: Converts verified technical evidence and functional information into a human-readable project mental model.
infer: false
---

You are the project documentation and mental-model specialist.

Inputs may include:
- verified technical documentation;
- repository-derived technical facts;
- functional documentation;
- reviewed architecture findings;
- business terminology.

Create a clear linked model:

business capability
-> workflow
-> business rule
-> API/event
-> Java component
-> Neo4j persistence
-> external integration
-> Cucumber scenario
-> unit/integration/performance tests.

Never invent links.

Use explicit confidence labels:
FACT
VERIFIED
INFERENCE
ASSUMPTION
UNKNOWN
OPEN QUESTION

Prefer:
- concise Markdown;
- tables where useful;
- Mermaid diagrams for relationships;
- evidence references.

Do not modify production code.
```

---

# 34. Reload Copilot Agent Definitions

After creating/changing agents, exit all Copilot CLI sessions and start a new one.

Inside Copilot CLI:

```text
/agent
```

Verify the following appear:

```text
local-coder
local-reviewer
local-mental-model
```

---

# 35. Security: Do Not Use `--allow-all`

For this experiment, never use:

```text
--allow-all
--allow-all-tools
--allow-all-urls
--yolo
```

Do not enter:

```text
/allow-all
/yolo
```

These are inconsistent with the security model.

---

# 36. Official Documentation Access Policy

Allowed official documentation examples:

```text
spring.io
docs.spring.io
openjdk.org
docs.oracle.com
neo4j.com
cucumber.io
mockito.org
javadoc.io
gatling.io
docs.gatling.io
maven.apache.org
repo.maven.apache.org
central.sonatype.com
docs.docker.com
kafka.apache.org
```

Use only domains your company policy approves.

Do not treat a blog, Stack Overflow post, GitHub gist, random GitHub repository, Medium article, Reddit post or community package as an authoritative official source unless separately approved.

---

# 37. Prefer Copilot URL Tool for Documentation

GitHub Copilot CLI requires approval for external URLs by default.

For a documentation research session you may preapprove selected official domains.

Example:

```bash
copilot-devstral \
  --allow-url=docs.spring.io \
  --allow-url=spring.io \
  --allow-url=openjdk.org \
  --allow-url=docs.oracle.com \
  --allow-url=neo4j.com \
  --allow-url=cucumber.io \
  --allow-url=mockito.org \
  --allow-url=docs.gatling.io \
  --allow-url=maven.apache.org
```

Keep the list minimal.

Do not use `--allow-all-urls`.

---

# 38. Deny Obvious Downloader Commands

For coding sessions, add deny rules.

A practical launcher variant can pass:

```bash
--deny-tool='shell(curl)' \
--deny-tool='shell(wget)' \
--deny-tool='shell(fetch)' \
--deny-tool='shell(git push)'
```

For example:

```bash
copilot-devstral \
  --deny-tool='shell(curl)' \
  --deny-tool='shell(wget)' \
  --deny-tool='shell(fetch)' \
  --deny-tool='shell(git push)'
```

This is a useful guardrail but is **not a complete network sandbox**.

Why?

Many runtimes can make HTTP requests:

```text
python
node
java
ruby
perl
```

Therefore the actual corporate firewall/proxy/EDR policy remains the strongest technical guarantee.

---

# 39. Stronger Shell Strategy

For sensitive reverse-engineering sessions, prefer limiting shell access to known development operations.

Useful commands typically include:

```text
git status
git diff
git log
git show
rg
grep
find
mvn test
mvn verify
./mvnw test   (only if wrapper source is approved)
java
javac
```

Anything unusual should remain approval-based.

Do not blanket-allow arbitrary shell execution just because the model is local.

---

# 40. Maven/Artifact Repository Check

Your company artifact repository is permitted.

Inspect Maven effective settings:

```bash
mvn help:effective-settings
```

Verify configured mirrors/repositories point to approved company infrastructure where required.

Inspect:

```bash
cat ~/.m2/settings.xml
```

Look for corporate mirrors.

Never change company-managed settings unless authorized.

---

# 41. Maven Wrapper Safety Check

Inspect:

```bash
cat .mvn/wrapper/maven-wrapper.properties
```

Look for:

```text
distributionUrl=
```

A Maven wrapper can download a Maven distribution if it is not locally available.

If your policy forbids public executable acquisition, verify the distribution is:

- already installed/cached; or
- provided through company-approved artifact infrastructure.

Otherwise prefer company-approved installed Maven.

---

# 42. Gradle Wrapper Safety Check

If a module uses Gradle:

```bash
cat gradle/wrapper/gradle-wrapper.properties
```

Check:

```text
distributionUrl=
```

The same rule applies.

---

# 43. Docker Safety Rule

Do not allow the agent to casually run:

```bash
docker pull ...
```

A container image is executable content.

Existing locally available/company-approved images can be used according to company policy.

New external images require manual review.

---

# 44. npm/pip/brew Safety Rule

Do not allow automatic:

```text
npm install
npx
pip install
pip3 install
brew install
```

unless specifically company-approved.

If a dependency is needed, the agent must report it to you.

---

# 45. Phase A Pipeline — Reverse Engineering

Start from project root:

```bash
copilot-devstral
```

Select:

```text
/agent
local-coder
```

Prompt:

```text
REVERSE ENGINEERING ONLY.

Do not modify application code.

Work incrementally because the context limit is 32768 tokens.

Start by discovering the repository structure.

For each bounded slice:
1. identify the component;
2. inspect source;
3. inspect configuration;
4. inspect tests;
5. identify Neo4j impact;
6. identify upstream/downstream dependencies;
7. save compact evidence-backed findings under `.ai-local/brain`;
8. then continue to the next slice.

Never attempt to load the entire repository into one context.
```

---

# 46. Reverse Engineering Slice Order

Recommended sequence:

```text
1. Repository/module layout
2. Build files and dependency management
3. Deployable modules
4. API controllers/endpoints
5. Service layer
6. Neo4j persistence layer
7. Node/relationship model
8. Kafka/synchronization flows
9. Export flows
10. External API integrations
11. Cucumber framework
12. Mockito/unit testing conventions
13. Gatling/performance test model
14. Shell utilities
15. Azure/GitHub pipelines
16. Coding conventions
17. Error handling
18. Logging/metrics
19. Hot/high-volume paths
20. Business-to-technical traceability
```

Do not move to a huge new category until the current category has a concise summary.

---

# 47. Devstral Handoff

At the end of a reverse-engineering slice, require Devstral to write:

```text
.ai-local/technical/<slice-name>.md
```

Use this format:

```markdown
# Slice

## Scope

## Verified facts

## Components

## Data flow

## Neo4j impact

## Config

## Tests

## Evidence

## Unknowns

## Questions for reviewer
```

This file becomes the handoff to GPT-OSS.

---

# 48. Phase B Pipeline — Independent GPT-OSS Review

Exit Devstral.

Confirm:

```bash
lms ps
```

Then start:

```bash
copilot-gptoss
```

Select:

```text
/agent
local-reviewer
```

Prompt:

```text
Review `.ai-local/technical/<slice-name>.md`.

Do not trust the previous model.

Use the actual repository to verify each material claim.

Identify:
- incorrect relationships;
- missing code paths;
- incorrect Neo4j conclusions;
- unsupported assumptions;
- transaction risks;
- test gaps;
- misleading terminology.

Write:
`.ai-local/reviews/<slice-name>-review.md`
```

---

# 49. Phase C — Devstral Reconciliation

Exit GPT-OSS.

Start:

```bash
copilot-devstral
```

Prompt:

```text
Read:
`.ai-local/reviews/<slice-name>-review.md`

For each BLOCKER or HIGH issue:
1. verify it against source;
2. accept only evidence-backed findings;
3. correct the technical document;
4. record rejected reviewer findings and evidence.

Do not implement production changes.
```

Now the technical knowledge has gone through two independent passes.

---

# 50. Phase D — Gemma Human-Readable Enrichment

Exit Devstral.

Start:

```bash
copilot-gemma
```

Select:

```text
/agent
local-mental-model
```

Prompt:

```text
Read the verified technical slice and relevant functional material.

Create a human-readable mental model.

Link:
business meaning
-> workflow
-> endpoint/event
-> code
-> Neo4j
-> tests.

Do not invent missing relationships.

Update relevant files under:
`.ai-local/brain/`
```

---

# 51. Final Brain Format

The brain should not become a giant single file.

Maintain small linked files.

Example:

```text
.ai-local/brain/
  modules/
  endpoints/
  workflows/
  neo4j/
  kafka/
  testing/
  performance/
  utilities/
  terminology/
```

An index should link them.

---

# 52. Feature-Development Pipeline

Create:

```text
.ai-local/tasks/current-task.md
```

Include:

```markdown
# Requirement

## Functional intent

## Acceptance criteria

## Known constraints

## Related brain files

## Open questions
```

Then:

```bash
copilot-devstral
```

Prompt:

```text
Read current-task.md and only the relevant project-brain files.

Then inspect the referenced real code.

Do not code immediately.

First return:
1. understanding;
2. affected components;
3. Neo4j impact;
4. API impact;
5. test impact;
6. implementation plan;
7. risks.
```

Review the plan yourself.

Then:

```text
Proceed.
```

---

# 53. Devstral Implementation Gate

After implementation require:

```text
1. git diff
2. compile
3. focused unit tests
4. focused Cucumber/integration tests
5. static checks already used by the project
6. report every failure
```

Do not permit:

```text
"The implementation should work."
```

as a completion criterion.

---

# 54. GPT-OSS Code Review

Exit Devstral.

Run:

```bash
copilot-gptoss
```

Prompt:

```text
Review the current git diff.

Use the surrounding code and project brain.

Do not modify code.

Focus on:
- correctness;
- compatibility;
- Neo4j behavior;
- transaction behavior;
- concurrency;
- edge cases;
- Cucumber coverage;
- Mockito quality;
- performance;
- regressions;
- project conventions.

Write findings to:
`.ai-local/reviews/current-code-review.md`
```

---

# 55. Devstral Fix Loop

Exit GPT-OSS.

Run:

```bash
copilot-devstral
```

Prompt:

```text
Read `.ai-local/reviews/current-code-review.md`.

Verify each BLOCKER/HIGH finding independently.

Implement only valid findings.

Rerun build and affected tests.

Update review status.
```

Repeat only while there are evidence-backed material issues.

Do not run an endless model-discussion loop.

---

# 56. Objective Stopping Conditions

A coding task is complete when:

```text
Compilation: PASS
Relevant unit tests: PASS
Relevant integration/Cucumber tests: PASS
Existing static checks: PASS
Reviewer BLOCKER: 0
Reviewer HIGH: 0
Unexpected diff: 0
Brain update needed: completed
```

The models agreeing with each other is not a stopping condition.

---

# 57. Gemma Final Documentation Pass

After code is verified:

```bash
copilot-gemma
```

Prompt:

```text
Read:
- current task;
- final git diff summary;
- verified review;
- affected project-brain files.

Update the human-readable mental model.

Explain:
- what changed;
- why;
- functional behavior;
- technical behavior;
- Neo4j impact;
- testing;
- important operational considerations.

Do not modify production code.
```

---

# 58. Cloud Escalation

Use normal approved company Copilot only when required:

```bash
copilot-cloud
```

Examples:

```text
local model failed twice;
models disagree on a high-risk issue;
complex architecture problem;
hard debugging problem;
local model lacks capability.
```

Before cloud escalation, use the existing organizational policy for source-code handling.

The purpose of the local pipeline is to reduce, not necessarily eliminate, cloud usage.

---

# 59. Documentation Research Session

Do public documentation research separately from source-code reverse engineering when practical.

Create:

```bash
mkdir -p ~/local-ai-public-research
cd ~/local-ai-public-research
```

Start a local model with only approved URL access.

Example:

```bash
copilot-devstral \
  --allow-url=docs.spring.io \
  --allow-url=openjdk.org \
  --allow-url=docs.oracle.com \
  --allow-url=neo4j.com \
  --allow-url=cucumber.io \
  --allow-url=mockito.org \
  --allow-url=docs.gatling.io \
  --allow-url=maven.apache.org
```

Ask generic questions.

Example:

```text
Find the current official Spring Boot documentation for Spring Data Neo4j
transaction management.

Summarize only the relevant official documentation.
```

Do not paste company source into this session.

Save useful public summaries under:

```text
.ai-local/research/
```

after manual review.

---

# 60. Shell Script Requirement Handling

If the model says a script is needed, require this output:

```text
EXTERNAL SCRIPT REQUEST

Purpose:
Official publisher:
Official URL:
Version:
Expected SHA/checksum if published:
Why needed:
What commands it would run:
Whether an existing local alternative exists:
```

Then stop.

You manually:

1. inspect the URL;
2. inspect the content;
3. follow company approval;
4. download/install if permitted;
5. make it available locally.

Only afterward tell the agent:

```text
The reviewed script is now available at:
<local path>

You may read/use this local file.
```

---

# 61. Local Generated Shell Scripts

The agent may generate a new local script.

Example:

```text
scripts/analyse-permissions.sh
```

Review it before execution.

Especially inspect:

```text
curl
wget
nc
ssh
scp
python HTTP clients
node HTTP clients
docker pull
brew
pip
npm
rm
sudo
chmod
```

Generated code is not automatically trustworthy merely because it was generated locally.

---

# 62. PDF Processing

For confidential PDFs, prefer local processing.

LM Studio supports local document-chat workflows for common document types.

Initial test:

1. Open LM Studio.
2. Load Gemma or an appropriate local model.
3. Create a new local chat.
4. Attach the PDF.
5. Ask for structured extraction.
6. Compare results against the source.

Suggested extraction prompt:

```text
Extract only information explicitly supported by this document.

Produce:
- terminology;
- business rules;
- workflows;
- actors;
- systems;
- APIs/events;
- constraints;
- open questions.

For each claim, record page/section evidence where possible.

Do not invent missing details.
```

Then save the verified result into `.ai-local/functional/`.

---

# 63. PPT/PPTX Processing

For PowerPoint:

Preferred safe workflow:

```text
PowerPoint
-> Export as PDF
-> local LM Studio processing
```

For highly visual slides:

```text
PowerPoint
-> Export selected slide(s) as PNG
-> use a local vision-capable model
```

Do not automatically install a PPT parser from a random package repository for this experiment.

---

# 64. Image Processing

If your downloaded Devstral/Gemma variant supports vision:

1. Test on a non-sensitive image first.
2. Verify image input works in LM Studio.
3. Compare extracted facts manually.
4. Only then use confidential screenshots.

For architecture diagrams ask:

```text
Identify:
- systems;
- components;
- arrows;
- direction;
- labels;
- boundaries;
- external systems.

Separate visible facts from inferred relationships.
```

---

# 65. Confluence Processing

Initial safe baseline:

```text
Confluence
-> export/copy approved content
-> PDF or Markdown/text
-> .ai-local/functional/
-> local pipeline
```

Do not build a new Confluence MCP integration until the local experiment is proven useful and the integration is approved.

---

# 66. 32K Context Operating Method

Use 32K as a workbench.

Do not treat it as permanent memory.

Use:

```text
1. Search.
2. Read only relevant files.
3. Extract important facts.
4. Persist facts to project brain.
5. Start a new clean task.
6. Reload only relevant brain files.
```

This is more important than increasing the context window.

---

# 67. Context Budget Per Task

A practical rough target:

```text
20-30%   project brain / instructions
40-50%   actual relevant source
10-20%   task and previous findings
remainder model output/tool results
```

If a task needs far more source than that, split the task.

---

# 68. Avoid Large Raw Build Logs

Do not feed entire multi-megabyte logs.

Use:

```text
first failure;
root cause;
relevant stack trace section;
failed test names;
summary of repeated errors.
```

Persist long logs to disk and let the agent search them.

---

# 69. Avoid Large Neo4j Dumps

Do not give a 3 GB Neo4j dataset to the LLM.

Instead capture:

```text
labels;
relationship types;
sample bounded records;
constraints;
indexes;
query patterns;
cardinality/statistics;
hot paths;
representative Cypher;
performance-test generation logic.
```

The model needs structure and representative evidence, not all data.

---

# 70. Memory Monitoring

During a local session:

```bash
lms ps
```

In macOS Activity Monitor watch:

```text
Memory Pressure
Swap Used
LM Studio memory
```

Do not worry only about the "Memory Used" number.

Healthy:

```text
Memory pressure remains green
Swap stable
System responsive
```

Problem:

```text
pressure becomes yellow/red
swap continuously grows
UI becomes sluggish
tokens/sec collapses
```

If that happens:

```bash
lms unload --all
```

Restart the intended model with 32K context.

---

# 71. Do Not Increase Context First

If the model loses project understanding, first improve:

```text
brain structure;
retrieval instructions;
scope;
evidence files;
traceability.
```

Only after this is proven insufficient should you test a larger context.

---

# 72. Benchmark the Experiment

Create:

```text
.ai-local/benchmarks/results.md
```

Track for every task:

```markdown
## Task

Complexity: SMALL / MEDIUM / HARD

Model:
Quantization:
Context:

Correct files found:
Correct business understanding:
Correct Neo4j understanding:
Compile:
Tests:
Cucumber:
Hallucinated APIs:
Human corrections:
Reviewer valid findings:
Time:
Needed cloud escalation:
```

---

# 73. Recommended Initial Experiment

Do not reverse engineer the complete application on day one.

Choose one bounded, representative workflow.

Pipeline:

```text
Devstral reverse engineers
-> GPT-OSS challenges
-> Devstral reconciles
-> Gemma creates human-readable brain
-> give Devstral a small change in same workflow
-> GPT-OSS reviews
-> Devstral fixes
-> tests
-> compare against normal Copilot quality
```

If that workflow produces materially better context-aware code, expand.

---

# 74. Daily Command Cheat Sheet

## Start coder

```bash
copilot-devstral
```

## Start reviewer

```bash
copilot-gptoss
```

## Start documentation/mental-model agent

```bash
copilot-gemma
```

## Use approved cloud Copilot

```bash
copilot-cloud
```

## Check loaded model

```bash
lms ps
```

## Force clean RAM state

```bash
lms unload --all
```

## Check downloaded model keys

```bash
lms ls
```

## Check server

```bash
lms server status
```

## Start server

```bash
lms server start --port 1234 --bind 127.0.0.1
```

---

# 75. Troubleshooting: Copilot Cannot Connect

Check:

```bash
lms server status
```

Then:

```bash
curl http://127.0.0.1:1234/v1/models
```

Then:

```bash
echo "$COPILOT_PROVIDER_BASE_URL"
echo "$COPILOT_MODEL"
```

Expected local base URL:

```text
http://127.0.0.1:1234/v1
```

---

# 76. Troubleshooting: Wrong Model Is Used

Run:

```bash
lms ps
```

Exit Copilot.

Run:

```bash
lms unload --all
```

Restart with exactly one launcher:

```bash
copilot-devstral
```

Do not manually mix model/provider environment variables.

---

# 77. Troubleshooting: Model Runs Out of Context

Do not immediately increase context.

Instead:

```text
1. Save current findings.
2. Start a fresh session.
3. Read only relevant brain file.
4. Read only relevant code.
5. Continue.
```

If recurring failure persists, consider testing 48K/64K only after memory measurement.

---

# 78. Troubleshooting: Poor Coding Quality

Check:

```text
Did it read the brain?
Did it inspect actual existing code?
Did it inspect tests?
Was the task too large?
Did it hallucinate an API?
Did it use official docs when framework behavior was uncertain?
Did GPT-OSS independently review?
Did objective tests pass?
```

Do not assume a bigger quantization/context automatically solves poor project context.

---

# 79. Troubleshooting: Model Wants Internet Script

Reject execution.

Ask:

```text
Do not download it.

Return only:
- exact official URL;
- purpose;
- version;
- why required;
- expected installation result;
- commands it would execute.
```

Review manually.

---

# 80. Troubleshooting: Model Wants Public Package Installation

Same rule.

Ask for:

```text
package;
version;
official repository;
license;
reason;
company-approved equivalent;
whether the build can work without it.
```

Do not allow automatic installation.

---

# 81. Security Checklist Before Every Major Experiment

Verify:

```text
[ ] LM Studio bound to 127.0.0.1
[ ] only one model loaded
[ ] context 32768
[ ] remote MCP disabled unless approved
[ ] no --allow-all / --yolo
[ ] public URL access restricted
[ ] downloader commands not blanket-approved
[ ] Maven/Gradle repository configuration verified
[ ] no automatic public package install
[ ] project brain excluded from Git if required
[ ] company network controls remain active
```

---

# 82. Model Responsibility Checklist

## Devstral

```text
reverse engineer
search repository
code
refactor
write tests
write local scripts
run build/tests
update technical facts
```

## GPT-OSS

```text
challenge technical assumptions
review code
review graph design
review tests
review technical documentation
find gaps
```

## Gemma

```text
combine verified technical + functional knowledge
explain system
create human-readable documentation
create mental model
create traceability
```

Do not blur these responsibilities unnecessarily.

---

# 83. Recommended Final Pipeline

```text
                REQUIREMENT
                    |
                    v
          Read project brain
                    |
                    v
               DEVSTRAL
          analysis + coding
                    |
                    v
           compiler / tests
                    |
                    v
               GPT-OSS
          independent review
                    |
             issues found?
              /        \
            yes         no
             |           |
             v           |
          DEVSTRAL       |
             |           |
             +-> tests --+
                         |
                         v
                       GEMMA
                 mental-model update
                         |
                         v
                       DONE
```

Only one model is loaded at each model stage.

---

# 84. What This Setup Can and Cannot Guarantee

This setup provides strong application-level controls.

It does **not** replace:

```text
corporate firewall
proxy
EDR
endpoint security
company data-loss-prevention policy
```

A command-line tool running with your user privileges can theoretically use many mechanisms to reach a network.

Therefore, if the organizational requirement is an absolute technical guarantee that proprietary code can never leave the corporate boundary, that guarantee must also be enforced at network/DLP level.

---

# 85. Recommended First-Day Plan

Do only this on Day 1:

```text
1. Verify lms CLI.
2. Record model keys.
3. Verify memory estimates.
4. Configure LM Studio localhost server.
5. Test each model directly.
6. Create four Copilot launchers.
7. Verify local Devstral through Copilot.
8. Verify model unload.
9. Create the three user agents.
10. Test one read-only repository task.
```

Do not attempt the full project brain yet.

---

# 86. Recommended Second-Day Plan

```text
1. Select one bounded module/workflow.
2. Devstral reverse engineering.
3. GPT-OSS verification.
4. Devstral reconciliation.
5. Gemma mental-model pass.
6. Manually inspect accuracy.
```

---

# 87. Recommended Third-Day Plan

```text
1. Select one small real change.
2. Devstral implementation.
3. Build/tests.
4. GPT-OSS review.
5. Devstral corrections.
6. Gemma documentation.
7. Compare with normal Copilot result.
```

Only after this should you decide whether to scale the approach.

---

# 88. Official Documentation References Used for This Guide

Verify current behavior against these vendor-owned sources before changing security-sensitive settings:

- GitHub Docs — Using your own LLM models in GitHub Copilot CLI  
  https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/use-byok-models

- GitHub Docs — Allowing and denying tool use  
  https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/allowing-tools

- GitHub Docs — Configuring GitHub Copilot CLI  
  https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/configure-copilot-cli

- GitHub Docs — Creating custom agents for Copilot CLI  
  https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-custom-agents-for-cli

- GitHub Docs — Copilot CLI command reference  
  https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference

- LM Studio — OpenAI-compatible endpoints  
  https://lmstudio.ai/docs/developer/openai-compat

- LM Studio — `lms load` and memory estimation  
  https://lmstudio.ai/docs/cli/local-models/load

- LM Studio — Local server  
  https://lmstudio.ai/docs/cli/serve/server-start

- LM Studio — Idle TTL and Auto-Evict  
  https://lmstudio.ai/docs/developer/core/ttl-and-auto-evict

- LM Studio — Server settings  
  https://lmstudio.ai/docs/developer/core/server/settings

---

# Final Operating Rule

Use this mental model:

```text
32K context = temporary workbench
.ai-local = durable project memory
Devstral = engineer
GPT-OSS = independent reviewer
Gemma = communicator / mental-model builder
tests = authority
LM Studio = local inference runtime
Copilot CLI = agent/tool runtime
corporate network controls = security boundary
```

The objective is not to make one 24-31B local model behave exactly like a premium cloud model.

The objective is to build a **local, evidence-backed, iterative engineering system** in which:

- project context is reusable;
- local models handle routine work;
- independent review compensates for model weakness;
- objective tools verify correctness;
- cloud Copilot is reserved for difficult escalation;
- proprietary data stays within approved boundaries.
