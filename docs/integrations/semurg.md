---
catalog_title: Semurg
catalog_description: Self-hosted memory for agents over MCP, with a knowledge graph, vectors and cited answers
catalog_icon: /integrations/assets/semurg.svg
catalog_tags: ["mcp", "data"]
---

# Semurg MCP tool for ADK

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python</span>
</div>

[Semurg](https://one.semurg.io) is a self-hosted memory server for AI agents. It serves its memory tools over MCP (streamable HTTP)
from a node you install on your own Linux machine. This integration gives your ADK agent persistent memory across sessions: it stores
facts and passages, recalls entities by name, and retrieves context for a question with a citation for every item it returns.

## Use cases

- **Remember across sessions**: The agent stores what it learns about people, projects and decisions as entities and relations, and
  recalls them by name in a later session.
- **Answer with citations**: The agent asks the memory a question and gets back ranked items, each citing the document and sentence
  it came from, so its answer can show where a fact came from.
- **One memory per end user**: An app that serves many users passes a user id with each call, and each user's memory stays their own.

## Prerequisites

- A Semurg node on an x86_64 Linux machine with AVX2 (Ubuntu 24.04 with the installer, or any x86_64 Linux in Docker); see
  [install a node](https://one.semurg.io/llms.txt).
- The node's API token, in the `SEMURG_API_TOKEN` environment variable.

## Installation

Install a node (the first command shows the script without running it):

```bash
curl -fsSL https://one.semurg.io/install | less
curl -fsSL https://one.semurg.io/install | sudo bash
```

Then install ADK in your agent's environment:

```bash
pip install google-adk
```

## Use with agent

=== "Agent"

    ```python
    import os
    from google.adk.agents import Agent
    from google.adk.tools.mcp_tool.mcp_toolset import McpToolset, StreamableHTTPConnectionParams

    SEMURG_MCP_URL = os.environ.get("SEMURG_MCP_URL", "http://127.0.0.1:4000/v1/mcp")
    HEADERS = {
        "Authorization": "Bearer " + os.environ["SEMURG_API_TOKEN"],
        "X-Semurg-Scope": "default",  # the memory scope this agent reads and writes
    }

    root_agent = Agent(
        model="gemini-flash-latest",
        name="memory_agent",
        instruction=(
            "You have a persistent memory. Before answering, call memory_context with the names in the question. "
            "When you learn something worth keeping, call memory_remember with the entities and relations you found."
        ),
        tools=[McpToolset(connection_params=StreamableHTTPConnectionParams(url=SEMURG_MCP_URL, headers=HEADERS))],
    )
    ```

=== "Direct tool calls"

    This program stores one fact and recalls it, without a model. It runs as written in Semurg's release tests.

    ```python
    import asyncio, os
    from google.adk.tools.mcp_tool.mcp_toolset import McpToolset, StreamableHTTPConnectionParams

    URL = os.environ.get("SEMURG_MCP_URL", "http://127.0.0.1:4000/v1/mcp")
    HEADERS = {"Authorization": "Bearer " + os.environ["SEMURG_API_TOKEN"],
               "X-Semurg-Scope": os.environ.get("SEMURG_MCP_SCOPE", "default")}
    FACTS = {"entities": [{"name": "Ada", "type": "person"}, {"name": "Acme", "type": "organisation"}],
             "relations": [{"source": "Ada", "target": "Acme", "type": "WORKS_AT"}]}


    async def main():
        toolset = McpToolset(connection_params=StreamableHTTPConnectionParams(url=URL, headers=HEADERS))
        try:
            by = {t.name: t for t in await toolset.get_tools()}
            await by["memory_remember"].run_async(args={"facts": FACTS}, tool_context=None)
            print(await by["memory_recall"].run_async(args={"subject": "Ada"}, tool_context=None))
        finally:
            await toolset.close()


    asyncio.run(main())
    ```

## Available tools

Tool | Description
---- | -----------
`memory_status` | Whether memory is on, which scopes hold facts, and a scope's entity and relation types
`memory_remember` | Store facts (entities and relations) and passages of text in a scope
`memory_recall` | Everything stored about one entity, found by its name or a close spelling
`memory_relate` | Add one typed relation between two entities
`memory_context` | Ranked items relevant to a question, each with a citation to its source
`memory_document` | Store a document (PDF, Word, HTML or text) as memory in one call
`memory_forget` | Forget one entity, fact, passage or source
`memory_export` | Export everything a scope holds, one page per call
`memory_import` | Store the records an export produced

## Resources

- [Install and connect (for coding agents)](https://one.semurg.io/llms.txt)
- [Semurg MCP repository](https://github.com/One-Semurg/semurg-mcp)
- [Agent and framework integrations, tested on each release](https://one.semurg.io/agents)
