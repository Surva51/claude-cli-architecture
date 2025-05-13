# Claude CLI Permission System

## Introduction

The Claude CLI implements a sophisticated permission system that controls which tools can be executed, under what conditions, and with what level of user interaction. This document explains the architecture and design of this permission system.

## Permission System Architecture

The permission system is built around a central permission function that evaluates each tool execution request. This function takes into account multiple factors to determine whether a tool should be allowed to run:

1. Tool name and type
2. Tool parameters
3. User settings and preferences
4. Environment context (e.g., whether running in a trusted environment)
5. Bypass flags and modes

Based on these factors, the permission function returns one of three decisions:

1. **Allow**: The tool is allowed to execute without user confirmation
2. **Deny**: The tool is not allowed to execute
3. **Need Confirmation**: The tool requires explicit user confirmation before execution

## Central Permission Function

The central permission function follows this general flow:

1. Check if the tool is recognized and registered
2. Check if the tool is globally allowed or denied based on configuration
3. Check if the tool's parameters contain sensitive patterns or values
4. Check if the tool type is subject to specific permission rules
5. Check if bypass mode is active and applies to this tool
6. Determine the final permission decision
7. Return the decision along with a reason for logging and user display

## Permission Behaviors

Different tools have different permission behaviors:

### Always Allowed Tools

Some tools are always allowed without confirmation, such as:
- Basic information retrieval tools
- Safe read-only operations
- Tools that don't modify the system or access sensitive data

### Confirmation Required Tools

Tools that require user confirmation include:
- File system write operations
- Shell command execution
- Network operations with potential security implications
- Tools that might expose sensitive data

### Restricted Tools

Some tools might be completely restricted unless specific bypass flags are set:
- System modification tools
- Tools that could potentially cause harm if misused
- Tools with elevated privileges

## Bypass Mode

The permission system includes a bypass mode that allows privileged operations in trusted environments. Bypass mode can be activated through:

1. Command-line flags
2. Environment variables
3. Configuration settings

Bypass mode can be:
- **Full Bypass**: All permission checks are bypassed
- **Partial Bypass**: Only specific categories of tools are bypassed
- **Environment-Based Bypass**: Bypass is determined by the environment (e.g., running in a container)

## User Confirmation Flow

When a tool requires confirmation:

1. The permission function returns "Need Confirmation"
2. The CLI presents the user with:
   - The tool name and description
   - The parameters being passed to the tool
   - Why confirmation is required
   - Options to allow, deny, or allow all similar operations
3. The user's decision is recorded and applied
4. If the user selects "allow all," this preference is stored for the session

## Security Considerations

The permission system implements several security measures:

1. **Parameter Inspection**: Examines parameters for sensitive patterns or potential security risks
2. **Tool Category Restrictions**: Applies different rules to different categories of tools
3. **Environment Awareness**: Considers the security context of the execution environment
4. **Auditability**: Logs permission decisions for security auditing
5. **Fail-Safe Defaults**: Defaults to requiring confirmation when in doubt

## Configuration

The permission system can be configured through:

1. `.claude.json` configuration file
2. Environment variables
3. Command-line flags
4. User preferences stored during the session

Configuration options include:
- Default permission level for different tool categories
- Bypass mode settings
- Tool-specific permission rules
- Confirmation behavior preferences

## Permission Decision Logic

The core decision logic follows this general pattern:

```
if (bypass_mode_active && tool_eligible_for_bypass):
    return ALLOW
else if (tool_in_denied_list):
    return DENY
else if (tool_in_always_allowed_list):
    return ALLOW
else if (tool_parameters_contain_sensitive_patterns):
    return NEED_CONFIRMATION
else if (tool_type_requires_confirmation):
    return NEED_CONFIRMATION
else:
    return default_permission_for_tool_category
```

## Example Permission Scenarios

### Reading a Non-Sensitive File

1. Tool: `Read`
2. Parameters: `{file_path: "/public/docs/readme.md"}`
3. Decision: `ALLOW`
4. Reason: Reading non-sensitive files is generally allowed without confirmation

### Executing a Shell Command

1. Tool: `Bash`
2. Parameters: `{command: "ls -la /home/user"}`
3. Decision: `NEED_CONFIRMATION`
4. Reason: Shell commands require confirmation to prevent potentially harmful operations

### Writing to a System Directory

1. Tool: `Write`
2. Parameters: `{file_path: "/etc/hosts", content: "127.0.0.1 example.com"}`
3. Decision: `DENY`
4. Reason: Writing to system directories is denied for security reasons

## Conclusion

The Claude CLI permission system provides a flexible, secure framework for controlling tool execution. By combining static rules, dynamic parameter analysis, and user confirmation, it balances security with usability. The bypass mode provides flexibility for trusted environments while maintaining security in standard usage scenarios.