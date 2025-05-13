# MCP (Machine Capabilities Provider) Client Architecture

## Introduction

The MCP (Machine Capabilities Provider) client is a key component of the Claude CLI that enables extensibility through external servers. This document explains the architecture and design of the MCP client system, which allows the CLI to connect to external services and extend its capabilities.

## MCP System Overview

The MCP system consists of the following components:

1. **MCP Client**: The client-side component in the Claude CLI that connects to MCP servers
2. **MCP Servers**: External servers that provide additional tools and capabilities
3. **MCP Configuration**: Settings that define which MCP servers to connect to
4. **MCP Tool Registry**: System for registering tools provided by MCP servers
5. **MCP Tool Execution**: Mechanism for executing tools via MCP servers

## MCP Server Types

The MCP client supports two primary types of MCP servers:

1. **stdio MCP Servers**: Local processes that communicate via standard input/output
2. **SSE (Server-Sent Events) MCP Servers**: Remote servers that communicate over HTTP with server-sent events

## MCP Client Architecture

### Server Configuration

MCP servers are configured through:

1. The `.claude.json` configuration file in the user's home directory
2. Command-line configuration via `claude mcp add` and related commands
3. Environment variables for CI/CD and scripted environments

Each MCP server configuration includes:
- Server type (stdio or SSE)
- Server identifier
- Connection details (command to execute for stdio or URL for SSE)
- Authentication credentials (if required)
- Tool prefix for namespacing tools from this server

### Connection Establishment

The MCP client establishes connections to configured MCP servers during CLI initialization:

1. Load MCP server configurations from all sources
2. For each configured server:
   - Validate the configuration
   - Initialize the appropriate connection type (stdio or SSE)
   - Establish the connection
   - Handle connection errors and implement retry logic
   - Log connection status

### Tool Discovery and Registration

Once connected to MCP servers, the MCP client discovers and registers tools:

1. Send a discovery request to each connected MCP server
2. Receive tool definitions from the server, including:
   - Tool names and descriptions
   - Parameter schemas
   - Permission requirements
3. Register tools in the global tool registry with appropriate prefixes
4. Make tools available for Claude to use via function calling

### Tool Execution

When Claude invokes a tool provided by an MCP server:

1. The function call parser identifies the tool as an MCP tool
2. The permission system checks if the tool is allowed to execute
3. The MCP client routes the call to the appropriate MCP server
4. The server executes the tool and returns the result
5. The MCP client forwards the result back to Claude

## Communication Protocols

### stdio Protocol

For stdio MCP servers:

1. The MCP client spawns a child process using the configured command
2. Communication uses a JSON-based protocol over standard input and output
3. Each message includes a type, ID, and payload
4. Request/response pairs are matched using message IDs
5. Error handling includes timeouts and process monitoring

### SSE Protocol

For SSE MCP servers:

1. The MCP client establishes an HTTP connection to the server URL
2. The server keeps the connection open and sends server-sent events
3. Tool invocations are sent as HTTP POST requests
4. Responses are received as server-sent events
5. Authentication typically uses bearer tokens or API keys

## Fault Tolerance and Error Handling

The MCP client implements several fault tolerance mechanisms:

1. **Connection Retries**: Automatically retry failed connections with backoff
2. **Timeouts**: Implement timeouts for all MCP server operations
3. **Graceful Degradation**: Continue operation even if some MCP servers are unavailable
4. **Error Reporting**: Provide clear error messages when MCP tools fail
5. **State Recovery**: Handle reconnection after temporary failures

## Security Considerations

The MCP system incorporates several security measures:

1. **Tool Namespacing**: Tools from MCP servers are prefixed to prevent naming conflicts
2. **Permission Controls**: MCP tools are subject to the same permission system as built-in tools
3. **Input Validation**: Parameters are validated against schemas before being sent to MCP servers
4. **Secure Connections**: SSE connections use HTTPS by default
5. **Credential Management**: Sensitive credentials are stored securely

## MCP Server Discovery and Auto-configuration

In addition to manual configuration, the MCP client supports discovery and auto-configuration:

1. Project-specific MCP configurations through local `.claude.json` files
2. Environment-based configuration through environment variables
3. Interactive configuration through the `claude mcp` commands

## Extensibility

The MCP system is designed for extensibility:

1. **New Server Types**: The architecture can be extended to support new connection protocols
2. **Tool Discovery**: Dynamic tool discovery allows MCP servers to add or update tools
3. **Authentication Methods**: Support for various authentication schemes
4. **Custom MCP Implementations**: Organizations can implement custom MCP servers

## Conclusion

The MCP client architecture provides a flexible, secure way to extend the Claude CLI's capabilities through external servers. By supporting both local (stdio) and remote (SSE) MCP servers, it enables a wide range of integrations and extensions. The system's focus on security, fault tolerance, and extensibility makes it a powerful foundation for building custom tools and capabilities for Claude.