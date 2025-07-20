# EternalClientClaude

An intelligent documentation system for the EternalClient API that enables Claude to autonomously create, compile, and debug scripts from start to finish.

## 🚀 Overview

This repository contains comprehensive API documentation for EternalClient and a specialized `CLAUDE.md` file that guides Claude (Anthropic's AI assistant) to agentically develop scripts. The system allows Claude to:

- Search and discover the correct API methods and classes
- Generate complete script implementations following best practices
- Run `gradle jar` to compile scripts
- Automatically fix compilation errors
- Follow the Tree-Branch-Leaf framework architecture

## 📋 Prerequisites

To use this system with Claude, you need:

1. **Claude Desktop App** with MCP (Model Context Protocol) support
2. **Filesystem MCP Server** installed and configured
3. **EternalClient API** files at your specified path
4. **Java JDK** and **Gradle** for script compilation

### Installing Filesystem MCP

Add the following to your Claude Desktop configuration:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/path/to/your/EternalClientApi"
      ]
    }
  }
}
```

## 🔍 How the Search Process Works

The CLAUDE.md file implements a sophisticated search strategy:

### 1. **Package Discovery**
Claude first identifies the relevant package file based on the feature:
- Entity/NPC interactions → `ENTITY_OBJECT_INTERACTION_PACKAGES.md`
- Banking/GE → `LOADOUT_BANK_GE_PACKAGES.md`
- Walking → `WALKING_AND_NAVIGATION_PACKAGES.md`
- Game ticks → `GAME_TICK_AND_EVENT_LISTENERS.md`
- Interfaces → `TAB_INTERACTION_PACKAGES.md`

### 2. **API Documentation Access**
Claude uses MCP filesystem tools to query the `javadocs_complete.json`:

```bash
# Get specific class details
jq '.classes["Package net.eternalclient.api.containers.bank.Bank"]' /path/to/your/EternalClientApi/javadocs_complete.json

# Find methods by pattern
jq '.classes["Package net.eternalclient.api.containers.bank.Bank"].methods[]? | select(.name | contains("withdraw"))' /path/to/your/EternalClientApi/javadocs_complete.json
```

### 3. **Verification Process**
- Never guesses method names or signatures
- Always verifies exact API details before implementation
- Prefers static utility classes over direct instantiation

## 🏗️ Script Architecture

All scripts follow the mandatory Tree-Branch-Leaf framework:

```
Main (Root) → Branches (decisions) → Leaves (actions)
```

- **Main.java**: Extends AbstractScript, initializes the Tree
- **Branches**: Implement conditional logic with `isValid()`
- **Leaves**: Execute specific actions in `onLoop()`

## 🛠️ Compilation and Error Handling

Claude automatically:
1. Runs `./gradlew jar` after implementation
2. Parses compilation errors
3. Fixes issues by referencing the API documentation
4. Re-runs compilation until successful

## 📁 Repository Structure

```
├── CLAUDE.md                              # Instructions for Claude
├── README.md                              # This file
├── SETUP_GUIDE.md                         # Project setup instructions
├── TREE_BRANCH_LEAF_FRAMEWORK.md          # Framework documentation
├── EXAMPLE_RUNECRAFTING_SCRIPT.md         # Example implementation
├── javadocs_complete.json                 # Complete API documentation
└── *_PACKAGES.md                          # Package reference files
```

## 💡 Usage Instructions

1. **Setup Paths**: Update all `/path/to/your/EternalClientApi/` references in CLAUDE.md to your actual API location

2. **Configure MCP**: Ensure the filesystem MCP server has access to your EternalClient API directory

3. **Start Claude**: Open Claude Desktop with MCP enabled

4. **Request Script**: Ask Claude to create any EternalClient script, for example:
   ```
   "Create a woodcutting script that banks logs at Varrock West"
   ```

5. **Let Claude Work**: Claude will automatically:
   - Search for relevant APIs
   - Implement the script
   - Compile and fix any errors
   - Deliver a working script

## 🔧 Improvements and Suggestions

We welcome suggestions for improving the search process and API discovery! Current areas for enhancement:

### Search Optimization
- Consider implementing a pre-indexed search system for faster API lookups
- Add fuzzy matching for method names to handle common variations
- Cache frequently accessed API patterns

### Error Handling
- Expand compilation error patterns for better automatic fixes
- Add support for runtime error detection and correction
- Implement test generation for common script patterns

### Documentation
- Add more example scripts for common tasks
- Create video tutorials showing the system in action
- Expand framework documentation with advanced patterns

## 🤝 Contributing

Feel free to submit issues or pull requests with:
- Improved search strategies
- Additional example scripts
- Enhanced error handling patterns
- Documentation improvements

## 📝 License

This documentation system is provided as-is for the EternalClient community. Please ensure you comply with EternalClient's terms of service when using these resources.

## 🔗 Links

- [EternalClient Website](https://eternalclient.com)
- [Claude Desktop](https://claude.ai/download)
- [MCP Documentation](https://modelcontextprotocol.io)

---

**Note**: Remember to replace all `/path/to/your/` placeholders with your actual file paths before using this system.