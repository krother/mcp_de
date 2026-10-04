
# Connecing AI Applications with MCP

## Course Goals

- create a MCP-compatible web app
- define a CLI application that an LLM can use
- connect an existing LLM to an MCP service
- build a practical usage exampe, e.g. recording an incident

Below, six separate modules are described that can be delivered over 2 days.

## Day 1

### Module 1: Fundamentals

- ACTIVITY: once upon a time
- VISUAL: Architecture of a LLM 
- concept: from neurons to transformers – what a model actually computes (next-token prediction)
- TABLE: frontier models vs. open-weight models
- Q: habt ihr schon welche von denen benutzt? Welche wie gut waren sie?
- recent improvements, context as a scarce resource  
- VISUAL: pelicans
- ACTIVITY: text with gaps using words: dneural network, LLMs, context, tokens, prompt, user prompt, system prompt, temperature
- ACTIVITY: group discussion: current and future business scenarios?


### Module 2: Building MCP Servers

- Q: wer war denn im FastAPI Kurs?
- Q: möchten wir pymcp oder fastmcp verwenden? Probiere auf pypi.org
- EXERCISE: implement a Fibonacci function, compare to ChatGPT
- VISUAL: MCP as an implementation of the Bridge Pattern
- concept: the N×M integration problem and how MCP standardizes it ("USB-C for AI")
- concept: architecture – host, client, server; JSON-RPC 2.0 messages
- concept: MCP vs. plain function calling vs. REST APIs

### 2.1 hello world
- EXERCISE: fibonacci mcp
  - set up project with uv
  - name bug by creating folder 'mcp'
  - install fastmcp
  - implement a hello world MCP service in Python with FastMCP
  - add fastmcp
  - run the MCP service as a HTTP service
  - add convenience app with Prefab to run in dev mode
  - visit the mcp in the browser

#### 2.2 information for the prompt

- EXERCISE: add instructions, docstring, parameter descriptions

### Module 3: MCP Clients

- VISUAL: The agent loop
  - concept: the agent loop – send tool schemas to the LLM, execute `tool_use` blocks, return results
- TABLE: stdio vs HTTP vs SSE: stdio for local, single-user processes; HTTP-based transports for remote, multi-client services, streaming-enabled; SSE obsoleted by Streamable HTTP in the current spec

#### 3.1 MCPInspector

- EXERCISE: install npm + MCPInspector
- EXERCISE: inspect the raw JSON-RPC traffic of an existing MCP server

#### 3.2 Ollama

- EXERCISE: run the smallest Qwen3 model via Ollama and llmcp

#### 3.3 Claude

- connect MCP to Claude in VSCode
- connect MCP to Claude in cli
- run prompt in both

## Tools, Resources and Prompts

- VISUAL: building blocks of MCPs: Tools, Resources and Prompts
- EXERCISE: extend the server to read resources
- EXERCISE: extend the server render prompt templates
- concept: logging and debugging – why `print()` breaks stdio servers, logging to stderr instead
- EXERCISE: broken code: debug a server with typical faults (wrong config path, missing type hints, stdout pollution) and fix them


## Day 2

### Module 4: Using MCPs

  - exercise: let the LLM record an incident from a free-text description ("the printer on floor 3 is on fire")

  - VISUAL: project structure

- workflow automation in LangGraph
  - concept: graphs, nodes, edges and state; when a fixed workflow beats a free agent
  - concept: loading MCP tools into LangGraph via `langchain-mcp-adapters`
  - exercise: a workflow "classify incident → assign priority → notify responsible team"
  - Coding Kata: the group extends the workflow with a conditional branch (escalation for high priority)



- Generating structured calls to LLMs with docstrings and pydantic
  - type annotations
  - concept: how FastMCP turns type hints and docstrings into JSON schemas
  - concept: pydantic models as tool inputs and outputs; validation, enums, `Field(description=...)`
  - exercise: improve a tool with vague parameters (`data: str`) into a well-typed pydantic interface and compare LLM behavior
  - activity: "bad docstring contest" – which description makes the LLM call the tool incorrectly?
- error recovery patterns
  - concept: retries, timeouts, idempotency, fallbacks
  - exercise: add validation and meaningful error messages to the incident server, observe self-correction of the LLM
  - debugging: a tool that fails silently and makes the LLM hallucinate a success

- relational and vector databases
  - VISUAL: TF Projector
  - concept: embeddings as vectors of meaning

  - concept and vetor-based search over documents
  - concept: persisting MCP data in SQLite/PostgreSQL via SQLAlchemy
  - USE A DATABASE THAT IS pip-installable
  - exercise: a `search_similar_incidents` tool backed by embeddings

- decoupling MCP servers from existing applications: FastMCP submodules vs.
  - concept: mounting/composing several FastMCP servers into one
  - exercise: expose an existing FastAPI web app as an MCP service

- Standalone microservices vs. MCP Gateways
  - concept: Docker, HTTP transport
  - concept: MCP gateways/proxies – central routing, authentication, logging, tool filtering
  - ACTIVITY: sketch an architecture for 3 MCP services

### Module 5: Data Protection and Security

- security risks when using AI
  - concept: prompt injection (direct and indirect), tool poisoning
  - concept: excessive agency and confused-deputy problems; OWASP Top 10 for LLM applications
  
- data protection risks when using AI
  - concept: local vs. cloud models from a data-protection perspective; retention and training opt-outs
  - exercise: trace the data flow of one agent session and mark all personal data
  - activity: Fachlandkarte data protection – data categories, processors, legal bases

- design patterns for securing AI applications
  - concept: least privilege – read-only tools, scoped credentials, tool allowlists
  - exercise: add audit logging of every tool call

- Human-in-the-Loop: implementing explicit approval steps
  - QUESTION: which actions need approval (irreversible, outward-facing, costly)
  - concept: interrupts in LangGraph
  - Programmiere mit mir: an approval step before `close_incident` is executed
  - exercise: implement a LangGraph interrupt that waits for a human decision

### Module 6: Practical Considerations when using MCPs

- lethal trifecta
- software engineering best practices
  - functional and non-functional requirements
  - concept: testing MCP servers – client test with pytest
  - exercise: write pytest tests for the incident server
  - concepts: latency, deployment, authentication, scaling

- using the MCP as a backend worker
  - concept: MCP servers without an LLM – calling tools programmatically from scripts and pipelines
  - concept: long-running tasks, progress notifications and cancellation
  - exercise: a batch script that imports a CSV of incidents via the MCP server
  - discussion: when is MCP the right abstraction, and when is a plain REST API simpler?


- summary and conclusions
  - discussion: next steps, transfer to participants' projects, further resources
  - feedback round

## Didaktische Methode

Der Kurs wird nach einer vom Trainer über die letzten 20 Jahre etablierten Methode durchgeführt. Die folgenden Aktivitäten werden während des Kurses stattfinden:

Programmiere mit mir: der Trainer schreibt kurze Codebeispiele, um den
Teilnehmern neue Konzepte nahezubringen. Dies erfolgt langsam genug, so daß die Teilnehmer mitprogrammieren und Fragen stellen können.

Fachlandkarten: wichtige Begriffe und deren Beziehungen aus jedem Modul werden in Form einer Infografik (Fachlandkarte) übersichtlich dargestellt. Die Informationsdichte ist passend dosiert, um den Lernprozess zu beschleunigen.

Reduzierte Beispiele: die Teilnehmer werden unvollständige Programme erhalten, die sie unter Anwendung der Inhalte eines Moduls vervollständigen.

Debugging: die Teilnehmer erhalten fehlerhafte Programme, die sie debuggen.
Coding Kata: die Gruppe schreibt gemeinsam ein Programm, um eine vorgegebene
Aufgabe zu lösen.

## Text

This two-day course enables participants to build AI interfaces for business applications. They
learn to expose their own projects as MCP services and use commercial LLMs such as Claude
or GitHub copilot to interact with them in an agentic fashion. The course covers use cases such
as:

- create a MCP-compatible web app
- define a CLI application that an LLM can use
- connect an existing LLM to the above services
- build
- build a practical usage exampe, e.g. recording an incident

The course targets participants with some working experience in Python. To participate in the
practical part, they will need a local Python installation and use libraries such as FastAPI or
anthropic. Some knowledge of FastAPI and Object-Oriented Programming is useful but not
required.
A prerequisite for the course is that participants have access to an LLM like Claude Desktop or
the Claude Command Line interface. Alternatively, the Copilot equivalent will also work. It
should be pointed out that some of the activities wiill not work with the Cloud variant, as the
MCP services will not be visible outside the internal network.

## Exercises
  - EXERCISE 2: a step-by-step guide for implementing a MCP server for incident reporting
    goal: complete an incident-recording server with tools  `create_incident`, `list_incidents`, `close_incident`
    - project setup with uv, the `mcp` / `fastmcp` package
    - implement a first tool with `@mcp.tool()`
    - add type hints
    - add a docstring
    - add a resource (`@mcp.resource("incident://{id}")`)
    - add a prompt template


Discussion:
- concept: selection criteria – cost, latency, context size, tool-use capability, data residency
