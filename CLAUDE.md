# EternalClient API Development Guidelines

## IMPORTANT: API Reference Location
The EternalClient API reference files are located at: /path/to/your/EternalClientApi/
ALWAYS use the files at this location as your primary reference when developing scripts.

## REQUIRED: Use MCP Tools for API Access
**You MUST use these MCP tools to access API documentation:**

1. **For regular files** (packages, framework docs):
   - `mcp__filesystem__read_file` - Read complete files
   - `mcp__filesystem__list_directory` - List directory contents
   - `mcp__filesystem__search_files` - Search for files by pattern

2. **For javadocs_complete.json** (NEVER read directly - too large):
   - Use Bash with jq to query the local JSON file directly
   - Query examples:
     ```bash
     # Get a specific class (note: package names are lowercase)
     jq '.classes["Package net.eternalclient.api.containers.bank.Bank"]' /path/to/your/EternalClientApi/javadocs_complete.json
     
     # Find all Bank-related classes
     jq '.classes | keys[] | select(contains("Bank"))' /path/to/your/EternalClientApi/javadocs_complete.json
     
     # Get specific methods
     jq '.classes["Package net.eternalclient.api.containers.Inventory"].methods[] | select(.name == "count")' /path/to/your/EternalClientApi/javadocs_complete.json
     ```

## Workflow Instructions

### 1. Script Architecture and Logic Flow (MANDATORY FOR ALL SCRIPTS)
**ALWAYS** use the Tree-Branch-Leaf framework from `/path/to/your/EternalClientApi/TREE_BRANCH_LEAF_FRAMEWORK.md` and `/path/to/your/EternalClientApi/EXAMPLE_RUNECRAFTING_SCRIPT.md`:

**USE THE OFFICIAL ETERNALCLIENT API FRAMEWORK CLASSES:**
```java
import net.eternalclient.api.frameworks.tree.Tree;
import net.eternalclient.api.frameworks.tree.Root;
import net.eternalclient.api.frameworks.tree.Branch;
import net.eternalclient.api.frameworks.tree.Leaf;
```

**Implementation Rules:**
- Main.java initializes the Tree with the official Tree class: `private final Tree tree = new Tree();`
- **IMPORTANT: Main.java IS your script root - DO NOT create a separate Root class**
- The Tree class internally creates its own Root - you use `tree.addBranches()` to add your branches
- In onStart(), use `tree.addBranches()` to add your main branch(es) to the tree
- Extend the official Branch class for decision nodes
- Extend the official Leaf class for action nodes
- Create packages for your custom branches and leaves (e.g., `branches` and `leaves`)
- The onLoop() method in Main.java should only call `return tree.onLoop();`
- Structure all conditional logic in Branch classes using isValid() methods
- Implement all actual actions in Leaf classes

### 2. Package Discovery Process
When implementing specific functionality, follow this strict order:

1. **Identify the feature category** and consult the appropriate package file:
   - **Entity/NPC/Object interactions**: `/path/to/your/EternalClientApi/ENTITY_OBJECT_INTERACTION_PACKAGES.md`
   - **Banking/GE/Equipment**: `/path/to/your/EternalClientApi/LOADOUT_BANK_GE_PACKAGES.md`
   - **Walking/Navigation**: `/path/to/your/EternalClientApi/WALKING_AND_NAVIGATION_PACKAGES.md`
   - **Game ticks/Events**: `/path/to/your/EternalClientApi/GAME_TICK_AND_EVENT_LISTENERS.md`
   - **Interface/Tab interactions**: `/path/to/your/EternalClientApi/TAB_INTERACTION_PACKAGES.md`

2. **Extract package names** from the relevant file

3. **Access API Documentation via MCP Filesystem Tools**

   **REQUIRED: Use MCP filesystem tools for all API documentation access:**

   **JSON Structure:**
   The javadocs_complete.json contains two main objects:
   - `"packages"`: Maps package names to their contents (classes, interfaces, enums)
   - `"classes"`: Maps full class names to their details (methods, fields, constructors)

   **Efficient Access Strategy:**
   
   1. **For Package Files** - Use direct file read:
      ```
      mcp__filesystem__read_file(path: "/path/to/your/EternalClientApi/ENTITY_OBJECT_INTERACTION_PACKAGES.md")
      ```

   2. **For Specific Classes** - Use jq on local file:
      ```bash
      # Get complete class details (note: class names have "Package " prefix and lowercase package names)
      jq '.classes["Package net.eternalclient.api.containers.bank.Bank"]' /path/to/your/EternalClientApi/javadocs_complete.json
      ```

   3. **For Package Contents** - Query package info:
      ```bash
      # Get all classes in a package
      jq '.packages["net.eternalclient.api.containers"]' /path/to/your/EternalClientApi/javadocs_complete.json
      ```
      
   4. **For Method Searches** - Find specific methods (use safe queries):
      ```bash
      # Find methods by name pattern (safe version that handles potential issues)
      jq '.classes["Package net.eternalclient.api.containers.bank.Bank"].methods[]? | select(type == "object" and has("name")) | select(.name | contains("withdraw"))' /path/to/your/EternalClientApi/javadocs_complete.json
      
      # Find exact method
      jq '.classes["Package net.eternalclient.api.containers.Inventory"].methods[]? | select(type == "object" and has("name")) | select(.name == "count")' /path/to/your/EternalClientApi/javadocs_complete.json
      
      # Find static methods (handle null modifiers)
      jq '.classes["Package net.eternalclient.api.containers.bank.Bank"].methods[]? | select(type == "object" and has("modifiers")) | select(.modifiers // [] | contains(["static"]))' /path/to/your/EternalClientApi/javadocs_complete.json
      
      # Alternative: Use test() for multiple patterns
      jq '.classes["Package net.eternalclient.api.containers.bank.Bank"].methods[]? | select(type == "object" and has("name")) | select(.name | test("withdraw|close|deposit"))' /path/to/your/EternalClientApi/javadocs_complete.json
      ```
      

   **DO NOT:**
   - ❌ Read the entire javadocs_complete.json file (it's too large)
   - ❌ Use complex regex patterns
   - ❌ Search for patterns that don't exist like `"package": "..."` or `"class": "..."`

### 3. API Reference Rules
- **NEVER guess** class names or method signatures
- **ALWAYS verify** exact names and parameters from `/path/to/your/EternalClientApi/javadocs_complete.json`
- **PREFER** static utility classes (e.g., Players, Npcs, GameObjects) over direct instantiation
- **USE** the exact import statements as shown in the documentation

## Critical Instructions
1. **DO NOT** create methods or classes that don't exist in `/path/to/your/EternalClientApi/javadocs_complete.json`
2. **DO NOT** assume method names based on common patterns - verify everything
3. **ALWAYS** check for static utility classes before trying to instantiate objects
4. **ALWAYS** use the official EternalClient Tree-Branch-Leaf framework from net.eternalclient.api.frameworks.tree
5. **ALWAYS** extend AbstractScript and implement all required lifecycle methods
6. **NEVER** write logic in onLoop() - only call tree.onLoop()
7. **ALWAYS** organize code as: Main (Root) → Branches (decisions) → Leaves (actions)
8. **ALWAYS** run `./gradlew jar` after completing implementation and fix any compilation errors
9. **ALWAYS** verify the build succeeds before considering the task complete

## When User Asks About Project Setup
**ONLY** when specifically requested, refer to `/path/to/your/EternalClientApi/SETUP_GUIDE.md` for:
- Project structure and setup
- Main class template with @ScriptManifest annotation
- Required imports and AbstractScript extension
- Lifecycle methods (onStart, onExit, onPause, onResume, onLoop)
- Gradle dependencies configuration

## File Reference Summary
- **Setup (on request only)**: `/path/to/your/EternalClientApi/SETUP_GUIDE.md`
- **Architecture**: `/path/to/your/EternalClientApi/TREE_BRANCH_LEAF_FRAMEWORK.md`, `/path/to/your/EternalClientApi/EXAMPLE_RUNECRAFTING_SCRIPT.md`
- **Packages**: `/path/to/your/EternalClientApi/*_PACKAGES.md` files (5 total)
- **API Documentation**: `/path/to/your/EternalClientApi/javadocs_complete.json` (search this for exact specifications)