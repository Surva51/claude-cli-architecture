# Built-in Tools Implementation

## Introduction

The Claude CLI includes a set of built-in tools that allow Claude to interact with the local environment. These tools provide capabilities such as executing shell commands, reading and writing files, and more. This document explains the architecture and implementation of these built-in tools.

## Tool Registration System

At the core of the tool implementation is the tool registration system:

1. **Tool Registry**: A central registry that maintains all available tools
2. **Tool Definitions**: Each tool has a definition that includes its name, description, parameter schema, and implementation
3. **Tool Categories**: Tools are organized into categories (file system, shell, networking, etc.)
4. **Tool Discovery**: The CLI discovers and registers tools during initialization

## Core Built-in Tools

The Claude CLI includes several core built-in tools:

### File System Tools

1. **Read**: Reads files from the file system
   - Parameters: file_path, offset, limit
   - Returns: The content of the file, optionally paginated
   - Security: Subject to permission checks for sensitive files and directories

2. **Write**: Writes or overwrites files
   - Parameters: file_path, content
   - Returns: Success/failure status
   - Security: Requires confirmation for most write operations

3. **Edit**: Performs find-and-replace operations on files
   - Parameters: file_path, old_string, new_string, expected_replacements
   - Returns: Success/failure status and details of replacements
   - Security: Requires confirmation for most edits

4. **MultiEdit**: Performs multiple edits on a file in a single operation
   - Parameters: file_path, edits (array of edit operations)
   - Returns: Success/failure status and details of all replacements
   - Security: Requires confirmation like regular edits

5. **LS**: Lists files and directories
   - Parameters: path, ignore (array of glob patterns to ignore)
   - Returns: List of files and directories
   - Security: Generally allowed without confirmation

### Search Tools

1. **Glob**: Searches for files matching glob patterns
   - Parameters: pattern, path
   - Returns: List of matching files
   - Security: Generally allowed without confirmation

2. **Grep**: Searches file contents for patterns
   - Parameters: pattern, path, include
   - Returns: List of matching files and the matching lines
   - Security: Generally allowed without confirmation

### Shell Tools

1. **Bash**: Executes shell commands
   - Parameters: command, timeout, description
   - Returns: Command output (stdout and stderr)
   - Security: Almost always requires confirmation

### Task Management Tools

1. **Task**: Launches a sub-agent with its own set of tools
   - Parameters: description, prompt
   - Returns: The sub-agent's response
   - Security: Inherits the security context of the parent

2. **TodoRead**: Reads the current to-do list
   - Parameters: None
   - Returns: List of to-do items with their status
   - Security: Always allowed without confirmation

3. **TodoWrite**: Updates the to-do list
   - Parameters: todos (array of to-do items)
   - Returns: Success/failure status
   - Security: Always allowed without confirmation

### Web Tools

1. **WebFetch**: Fetches content from a URL
   - Parameters: url, prompt
   - Returns: The processed web content
   - Security: Generally requires confirmation

2. **WebSearch**: Performs a web search
   - Parameters: query, allowed_domains, blocked_domains
   - Returns: Search results
   - Security: Generally requires confirmation

### Jupyter Notebook Tools

1. **NotebookRead**: Reads Jupyter notebooks
   - Parameters: notebook_path
   - Returns: The notebook's cells and outputs
   - Security: Subject to same permission rules as Read

2. **NotebookEdit**: Edits Jupyter notebook cells
   - Parameters: notebook_path, cell_number, new_source, cell_type, edit_mode
   - Returns: Success/failure status
   - Security: Subject to same permission rules as Edit

## Tool Implementation Architecture

Each built-in tool follows a common implementation pattern:

1. **Tool Definition**: Defines the tool's name, description, and parameter schema using JSONSchema
2. **Parameter Validation**: Validates input parameters against the schema
3. **Permission Check**: Calls the permission system to check if execution is allowed
4. **Core Implementation**: Implements the tool's functionality
5. **Result Formatting**: Formats the result for return to Claude
6. **Error Handling**: Implements robust error handling and reporting

## Security Considerations

The built-in tools incorporate several security features:

1. **Permission System Integration**: All tools are integrated with the permission system
2. **Input Validation**: Parameters are strictly validated to prevent injection attacks
3. **Resource Limitations**: Tools implement timeouts and resource limits
4. **Sandboxing**: Where possible, tool execution is sandboxed
5. **Path Sanitization**: File paths are sanitized and validated
6. **Command Sanitization**: Shell commands are carefully validated

## Tool Development Guidelines

New built-in tools are developed following these guidelines:

1. **Minimal Scope**: Each tool should do one thing well
2. **Comprehensive Schema**: Parameter schemas should be complete and accurate
3. **Robust Validation**: Input should be thoroughly validated
4. **Proper Error Handling**: All errors should be properly caught and reported
5. **Appropriate Permissions**: Permission requirements should match the tool's risk level
6. **Documentation**: Each tool should be well-documented

## Batch Processing

The Claude CLI includes a Batch tool that allows multiple tools to be executed in a single operation:

1. **Parallel Execution**: Multiple tool invocations can be executed in parallel
2. **Consolidated Results**: Results from all tool invocations are consolidated
3. **Efficiency**: Reduces context usage and latency for multiple operations
4. **Permission Inheritance**: Each tool in the batch respects its own permission rules

## Extension Points

The built-in tools system is designed to be extensible:

1. **Custom Tools**: New tools can be added to the registry
2. **Tool Overrides**: Existing tools can be overridden with custom implementations
3. **Tool Wrappers**: Tools can be wrapped to add pre/post-processing
4. **MCP Integration**: Tools from MCP servers are integrated into the tool registry

## Conclusion

The built-in tools implementation provides a secure, extensible foundation for Claude to interact with the local environment. The combination of a robust tool registry, comprehensive parameter validation, and tight integration with the permission system ensures that Claude can use these tools effectively while maintaining security and user control. The diverse set of built-in tools covers a wide range of functionalities, from file system operations to web interactions, providing Claude with the capabilities it needs to assist users with various tasks.