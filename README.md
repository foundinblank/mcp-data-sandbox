# mcp-data-sandbox

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue)](https://www.python.org/)
[![DuckDB](https://img.shields.io/badge/DuckDB-Latest-orange)](https://duckdb.org/)
[![dlt](https://img.shields.io/badge/dlt-Latest-green)](https://dlthub.com/)
[![MCP](https://img.shields.io/badge/MCP-Enabled-purple)](https://code.visualstudio.com/docs/copilot/chat/mcp-servers)

A quick start guide to setting up a minimal data stack using dlt and DuckDB, and then leveraging DuckDB's MCP server for SQL operations.

This guide will help you:
1. Quickly set up dlt and DuckDB.
2. Understand what an MCP server is and why it's useful.
3. Connect GitHub Copilot to your DuckDB data using a MCP server.

**Prerequisites:** 
* VSCode. [MCP Server should be enabled](https://code.visualstudio.com/docs/copilot/chat/mcp-servers#_enable-mcp-support-in-vs-code).
* GitHub Copilot in VSCode
* Python 3.9 or higher
* Git

## Set Up Your Minimal Data Stack

1. **Install [uv](https://docs.astral.sh/uv/):**
    ```bash
    curl -LsSf https://astral.sh/uv/install.sh | sh
    ```

2. **Create and navigate to your project directory:**
    ```bash
    mkdir mcp-data-sandbox
    cd mcp-data-sandbox
    ```

3. **Initialize a new project, create a virtual environment, and install dependencies:**
    ```bash
    uv init
    uv venv
    source .venv/bin/activate
    uv add duckdb
    uv add dlt[duckdb]
    uv sync
    ```

4. **Set up a dlt pipeline to ingest Chess.com data into DuckDB:**
    ```bash
    dlt init chess duckdb
    ```

5. **Run the data load job. You should see a new `chess_pipeline.duckdb` file appear in your directory:**
    ```bash
    python chess_pipeline.py
    ```

6. **Explore your dataset using DuckDB's GUI:**
    ```bash
    duckdb chess_pipeline.duckdb -ui
    ```
    For a CLI-based option, see [harlequin](https://harlequin.sh/).

Now you have data to work with. While LLMs can help you write queries, they don't natively interface with DuckDB.

## Set Up the MCP Server

MCP servers act as standardized interfaces (like REST APIs) that allow LLMs to access and interact with external tools such as DuckDB. 

Many LLM-based apps already use proprietary solutions to help LLMs do this. For example, GitHub Copilot in VSCode can access your workplace files and monitor your terminal, and the ChatGPT desktop app for macOS can access content in other apps like Notes or Terminal. 

But those are proprietary. MCP provides an open, programmable standard for enabling these integrations. 

Think of it as a power-up 🍄 for your LLM. Without power-ups, LLMs are fundamentally restricted to only understanding and interfacing with their immediate environment; by using MCP servers, you allow LLMs to "agentically" access other contexts. 

Let's enable your LLM to work directly with your DuckDB file using [this MCP server by MotherDuck](https://github.com/motherduckdb/mcp-server-motherduck). 

1. **Create a .vscode/mcp.json file.** This saves your MCP configuration in your workspace settings (i.e., your project folder) instead of your user settings. 

    ```bash
    mkdir .vscode
    cd .vscode
    touch mcp.json
    ```

2. Add DuckDB MCP configuration by copy and pasting this code block into `mcp.json`.

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

3. **Open a Copilot chat in VSCode** and ask it a question about your DuckDB file. For example:
   - "How many schemas and tables do I have in my DuckDB file?"
   - "Tell me something interesting about the data in my DuckDB file."
   - "Create a query that shows the top 10 chess players by rating."
   - "Create a visualization of the opening moves frequency in my chess data."

## Troubleshooting

If you encounter issues:

1. **MCP Server not connecting:**
    - Ensure MCP support is enabled in VSCode
    - Check that your `mcp.json` file has the correct path to your DuckDB file
    - Restart VSCode and try again

2. **DuckDB file not found:**
    - Verify that you've run `python chess_pipeline.py` successfully
    - Check the path in your `mcp.json` file