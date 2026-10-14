---
catalog_title: Semurg
catalog_description: Give agents persistent, cited memory that runs on your own hardware
catalog_icon: /integrations/assets/semurg.svg
catalog_tags: ["data", "mcp"]
---

# Semurg MCP tool for ADK

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python</span>
</div>

[Semurg](https://one.semurg.io) is a memory server for AI agents that you run on
your own hardware: a knowledge graph, vectors and graph RAG on one engine. Its
MCP server gives your ADK agent seven memory tools, so it can store what it
learns as entities, relations and passages, and recall them later with
citations, across conversations and across agents.

## Use cases

- **Long-term memory across sessions**: The agent stores what it learns and
  checks memory first in the next session, instead of asking the user again.

- **Shared memory between agents**: Agents that use the same scope read and
  write one memory, so a second agent can pick up where the first stopped.

- **Cited answers that say when nothing is stored**: Every item carries its
  source, and a question that matches nothing answers `nothing_stored` rather
  than a guess.

- **One memory per end user**: An app can keep each of its users' memories
  apart with a `user_id`, and delete one user's memory with one HTTP call.

## Prerequisites

- A Semurg node on a Linux x86_64 machine you control (Windows through WSL2 or
  Docker): [install guide](https://one.semurg.io/install)
- The API token the installer prints, in `SEMURG_API_TOKEN`

## Use with agent

=== "Python"

    ```python
    import os
    from google.adk.agents import Agent
    from google.adk.tools.mcp_tool import McpToolset
    from google.adk.tools.mcp_tool.mcp_session_manager import StreamableHTTPConnectionParams

    root_agent = Agent(
        model="gemini-flash-latest",
        name="memory_agent",
        instruction=(
            "Check memory_context before you answer. Store what you learn with "
            "memory_remember, sending the entities and relations you found as facts."
        ),
        tools=[
            McpToolset(
                connection_params=StreamableHTTPConnectionParams(
                    url="http://127.0.0.1:4000/v1/mcp",
                    headers={
                        "Authorization": f"Bearer {os.environ['SEMURG_API_TOKEN']}",
                        "X-Semurg-Scope": "my-app",
                    },
                ),
            )
        ],
    )
    ```

## Available tools

Tool | Description
---- | -----------
`memory_status` | Whether memory is on, the scopes that hold facts, and the classes and relations stored so far
`memory_remember` | Store facts (entities and relations), passages or text, with the source and date
`memory_recall` | Everything connected to one named entity, in both directions, each item cited
`memory_context` | Answer a question: the facts and passages around the names in it, with a grounding verdict
`memory_relate` | Link two stored entities with a typed relation
`memory_forget` | Forget an entity, one fact, one passage or one source
`memory_document` | Store a PDF, Word, HTML or text document as memory

## Additional resources

- [Semurg agent memory over MCP](https://one.semurg.io/blog/semurg-agent-memory-over-mcp)
- [Setup for agents and frameworks](https://one.semurg.io/agents)
- [ADK example program, run end to end on each release](https://github.com/One-Semurg/semurg-examples/tree/main/google-adk)
- [Every tool and route, for coding agents](https://one.semurg.io/llms.txt)
