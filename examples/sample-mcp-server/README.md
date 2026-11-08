# invoice-desk (sample)

A small MCP server with design problems planted on purpose, used to test agent-review. Data is in memory and nothing is sent anywhere.

Try it without your own project:

```
/agent-review:review-tools examples/sample-mcp-server/server.py
```

Then compare the report with [the answer key](../answer-keys/sample-mcp-server.md). Don't read the answer key first if you want to judge the review fairly.

To run the server itself (optional):

```bash
pip install "mcp[cli]>=2"   # MCP Python SDK v2
python server.py
```
