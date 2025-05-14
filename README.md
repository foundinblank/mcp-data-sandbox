# mcp-data-sandbox

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue)](https://www.python.org/)
[![DuckDB](https://img.shields.io/badge/DuckDB-Latest-orange)](https://duckdb.org/)
[![dlt](https://img.shields.io/badge/dlt-Latest-green)](https://dlthub.com/)
[![MCP](https://img.shields.io/badge/MCP-Enabled-purple)](https://code.visualstudio.com/docs/copilot/chat/mcp-servers)

A quick start guide to setting up a minimal data stack using [dlt](https://dlthub.com/) and [DuckDB](https://duckdb.org/), and then leveraging the [DuckDB MCP server](https://github.com/motherduckdb/mcp-server-motherduck) with [VSCode + GitHub Copilot](https://github.com/features/copilot) for AI-powered SQL operations. Written as a companion to my [MCP Servers for Dummies blog post](https://foundinblank.hashnode.dev/mcp-servers-for-dummies). 

This guide will help you:
1. Quickly set up dlt and DuckDB.
2. Understand what an MCP server is and why it's useful.
3. Connect GitHub Copilot to your DuckDB data using a MCP server.

**Prerequisites:** 
* VSCode with [MCP Server enabled](https://code.visualstudio.com/docs/copilot/chat/mcp-servers#_enable-mcp-support-in-vs-code).
* GitHub Copilot in VSCode
* Python 3.9 or higher

## Set Up Your Minimal Data Stack

1. Install [uv](https://docs.astral.sh/uv/):
    ```bash
    curl -LsSf https://astral.sh/uv/install.sh | sh
    ```

2. Create and navigate to your project directory:
    ```bash
    mkdir mcp-data-sandbox
    cd mcp-data-sandbox
    ```

3. Initialize a new project, create a virtual environment, and install dependencies:
    ```bash
    uv init
    uv venv
    source .venv/bin/activate
    uv add duckdb
    uv add dlt[duckdb]
    uv sync
    ```

4. Set up a dlt pipeline to ingest Chess.com data into DuckDB:
    ```bash
    dlt init chess duckdb
    ```

5. Run the data load job. You should see a new `chess_pipeline.duckdb` file appear in your directory:
    ```bash
    python chess_pipeline.py
    ```

6. Explore your dataset using DuckDB's GUI:
    ```bash
    duckdb chess_pipeline.duckdb -ui
    ```
    For a CLI-based option, see [harlequin](https://harlequin.sh/).

Now you have data to work with. While LLMs can help you write code and queries, they don't natively interface with DuckDB. Let's fix that by using a MCP server!

## Set Up the MCP Server

MCP servers are standardized interfaces (like REST APIs) that let large language models (LLMs) interact [with external tools such as DuckDB or Figma or Chrome](https://github.com/punkpeye/awesome-mcp-servers). 

Today, many LLM-powered apps use proprietary solutions to achieve this. For example, GitHub Copilot in VS Code can access your files and monitor your terminal. The ChatGPT desktop app for macOS can read content from apps like Notes or Terminal.

MCP is different. It’s open, programmable, and designed to be a standard. It gives LLMs a consistent way to integrate with external systems—-without vendor lock-in.

Think of it as a power-up 🍄 for your LLM. Without MCP, LLMs are boxed into their immediate environment. By using MCP servers, LLMs gain the ability to "agentically" access other tools, systems, or data sources. 

It's similar to exposing a remote database through a REST API endpoint. Suddenly, your data becomes accessible to any app or user (with the right credentials), and the possibilities multiply.

Now let’s apply that idea: with [the DuckDB MCP server by MotherDuck](https://github.com/motherduckdb/mcp-server-motherduck), your LLM can power up 💪 with the ability to query DuckDB and MotherDuck databases directly.

1. Create a `.vscode/mcp.json` file. This saves your MCP configuration in your workspace settings (i.e., your project folder) instead of your user settings:

    ```bash
    mkdir .vscode
    cd .vscode
    touch mcp.json
    ```

2. Add DuckDB MCP configuration by copy and pasting this code block into `mcp.json`:

    ```json
    {
      "servers": {
        "motherduck": {
          "command": "uvx",
          "args": [
            "mcp-server-motherduck",
            "--db-path",
            "${workspaceFolder}/chess_pipeline.duckdb"
          ]
        }
      }
    }
    ```

3. Open a Copilot chat in VSCode and ask it a question about your DuckDB file. For example:
   - "How many schemas and tables do I have in my DuckDB file?"
   - "Tell me something interesting about the data in my DuckDB file."
   - "Create a query that shows the top 10 chess players by rating."
   - "Create a visualization of the opening moves frequency in my chess data."

## Troubleshooting

If you encounter issues:

1. MCP Server not connecting:
    - Ensure MCP support is enabled in VSCode
    - Check that your `mcp.json` file has the correct path to your DuckDB file
    - Restart VSCode and try again

2. DuckDB file not found:
    - Verify that you've run `python chess_pipeline.py` successfully
    - Check the path in your `mcp.json` file
