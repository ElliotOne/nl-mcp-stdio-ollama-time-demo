# nl-mcp-stdio-ollama-time-demo

An end-to-end example demonstrating how to use **Model Context Protocol (MCP)** over **STDIO** in .NET to expose server-side tools and consume them from a client, then feed tool results into a local **Ollama-hosted LLM** for reasoning and response generation.

This project is intentionally simple and educational: a C# MCP server exposes a `GetCurrentTime` tool, and a C# MCP client discovers and calls that tool, then passes the result into an Ollama chat model.

## Overview

**What this project shows:**

- How to build an **MCP server** in .NET using STDIO transport
- How to expose tools using `[McpServerTool]` attributes
- How to build an **MCP client** that:
  - Launches the server as a child process
  - Discovers available tools
  - Calls a tool with structured arguments
- How to integrate MCP tool output into an **Ollama** chat flow
- How to correctly handle **STDOUT vs STDERR** to avoid protocol corruption

## Prerequisites

- **.NET 9.0 SDK** or later  
  https://dotnet.microsoft.com/download/dotnet/9.0
- **Ollama** installed and running locally  
  https://ollama.ai/
- Required Ollama model:
  ```bash
  ollama pull mistral:7b
  
## Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/your-username/mcp-stdio-ollama-time-demo.git
   cd mcp-stdio-ollama-time-demo

2. Restore NuGet packages:
   ```bash
   dotnet restore

## Project Structure

```text
.
├── MCPClient/
│   ├── Program.cs             # MCP client entry point
│   └── OllamaHttpChatClient.cs # Minimal Ollama HTTP chat client
│
└── MCPServer/
    ├── Program.cs             # MCP server host & configuration
    └── TimeTool.cs            # MCP tool definition
```

## MCP Server

The MCP server:

- Uses **STDIO** as its transport
- Exposes tools via attributes
- Writes **NO logs to STDOUT**

### Key Points

- **STDOUT** is reserved exclusively for MCP protocol messages
- **ALL logs** must go to **STDERR**
- Violating this will cause random client crashes or deserialization errors

### Logging Configuration (Critical)

```csharp
// CRITICAL FOR MCP OVER STDIO:
// MCP uses STDOUT as a strict protocol channel.
// Any log output written to STDOUT will corrupt the MCP message stream.
builder.Logging.ClearProviders();
builder.Logging.AddConsole(options =>
{
    options.LogToStandardErrorThreshold = LogLevel.Trace;
});
```
## Example Tool
```csharp
  [McpServerToolType]
public static class TimeTool
{
    [McpServerTool, Description("Get the current time for a city")]
    public static string GetCurrentTime(string city) =>
        $"It is {DateTime.Now:HH:mm} in {city}.";
}
```

## MCP Client

The MCP client is responsible for launching the MCP server, discovering available tools, invoking them with structured arguments, and forwarding the tool output to a local Ollama-hosted language model for reasoning and response generation.

### Responsibilities

The MCP client:

- Launches the MCP server executable as a child process using **STDIO**
- Establishes the MCP handshake and capability negotiation
- Discovers tools exposed by the server at runtime
- Invokes MCP tools with structured, named arguments
- Extracts and processes tool responses
- Passes tool output into an Ollama chat model for final response generation

### Client Entry Point

`Program.cs` is the main entry point for the MCP client. It performs the following steps:

1. Configures logging for diagnostics
2. Starts the MCP server using the STDIO transport
3. Creates and initializes the MCP client
4. Discovers available MCP tools
5. Calls the `GetCurrentTime` tool
6. Forwards the tool result to Ollama
7. Prints the final AI-generated response

### Logging Setup

Logging is enabled at **Trace** level to assist with debugging MCP protocol behavior.

```csharp
using var loggerFactory = LoggerFactory.Create(builder =>
{
    builder.AddConsole();
    builder.SetMinimumLevel(LogLevel.Trace);
});
```

## MCP Client

The MCP client:

- Launches the MCP server executable as a child process
- Discovers tools dynamically
- Calls tools with structured arguments
- Feeds tool output into an Ollama chat model

### IMPORTANT: Server Executable Path

```csharp
// IMPORTANT: Update the Command path to point to the MCP server executable
// on your own machine. This path is environment-specific.
Command = @"C:\Path\To\MCPServer\bin\Debug\net9.0\MCPServer.exe";
```
Each developer or deployment environment must update this path.

## Ollama Integration

- Ollama is used **after MCP tool execution**
- MCP performs the action (tool call)
- Ollama performs the reasoning / explanation
- **Model used:** mistral:7b  

## Running the Project

### 1. Start Ollama

Make sure Ollama is running locally:

```bash
ollama serve
```
## Verify Ollama

Make sure Ollama is running and responding correctly:

```bash
curl http://localhost:11434/api/tags
```
## Build the MCP Server

```bash
cd MCPServer
dotnet build
```
## Run the MCP Client

```bash
cd MCPClient
dotnet run
```

## Client Behavior

The client will:

- Launch the MCP server via STDIO  
- Discover available MCP tools  
- Call `GetCurrentTime`  
- Pass the result into Ollama  
- Print the final AI response  

## Example Output
MCP tools discovered:
GetCurrentTime — Get the current time for a city

MCP Tool Response:
It is 14:32 in Illzach, France.

AI Response:
The current time in Illzach, France is 14:32 CET.

## Key Technologies

- **.NET 9.0** (C# Console Apps)  
- **Model Context Protocol (MCP)**  
- **STDIO Transport**  
- **Ollama** (local LLM runtime)  
- **Mistral 7B** (chat model)  
- **Microsoft.Extensions.Logging**  

## Why STDIO Matters

MCP over STDIO is extremely strict:

- ✅ **STDOUT** → MCP protocol messages ONLY  
- ✅ **STDERR** → logs, diagnostics, exceptions  
- ❌ Any accidental `Console.WriteLine` breaks everything  

This repository intentionally highlights correct setup to avoid subtle bugs.

## License

See the [LICENSE](https://github.com/ElliotOne/nl-mcp-stdio-ollama-time-demo/blob/master/LICENSE) file for details. &#x20;

## Contributing

Contributions are welcome.

If you want to extend this example:

- Async MCP tools  
- Structured JSON tool outputs  
- HTTP-based MCP transport  
- Streaming responses  
- Tool error handling  

Open an issue or submit a Pull Request.

