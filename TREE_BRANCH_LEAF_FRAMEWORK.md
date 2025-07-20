# Tree-Branch-Leaf Logical Framework

## Introduction
The EternalClient API includes a hierarchical logical framework designed to streamline and organize the flow of execution. This framework consists of three primary components: the Tree, Branches, and Leaves. Understanding how these components interact is crucial for writing scripts effectively. The aim is to provide a comprehensive overview of the framework, ensuring developers can implement and utilize it in their scripts if they so wish.

## Overview
At the core of the framework lies the Tree, which acts as the root container for the entire hierarchical structure. Nested within the Tree are Branches and Leaves, forming a tree-like structure. The Root of the tree is a special kind of Branch with unique characteristics:

- **Root**: Always valid and serves as the starting point for execution.
- **Branches**: Can contain other Branches or Leaves. They represent the intermediary nodes of the tree.
- **Leaves**: The terminal nodes. They do not contain other nodes but execute specific logic.

### Hierarchical Structure
The framework is designed to process elements sequentially, from the Root to its children, and so forth, adhering to a set of rules for conditional processing and looping mechanisms. This structure facilitates dynamic execution paths based on the runtime validity of each node.

## Key Concepts
- **Sequential Execution**: The framework processes nodes starting from the Root, moving to its immediate children, and then to their descendants, in a hierarchical manner.
- **Conditional Processing**: Each node (either a Branch or Leaf) implements an `#isValid` method, determining its eligibility for processing.
  - If `#isValid` returns true, the node is processed.
  - If `#isValid` returns false, the framework skips to the next sibling node.
- **Looping Mechanism**: Post-processing of the last node at any level, the framework loops back to the Root to commence another iteration, ensuring comprehensive traversal.

## Implementation Details

### Tree
```java
public class Tree
{
    private final Root root;

    public Tree()
    {
        root = new Root();
    }

    public Leaf addBranches(Leaf... leaves)
    {
        root.addLeafs(leaves);
        return root;
    }

    public void clear()
    {
        root.children.clear();
    }

    public int onLoop()
    {
        return root.onLoop();
    }
}
```

The Tree class initializes with a Root, allowing the addition and removal of branches or leaves, and initiates the processing loop through `onLoop()`.

### Root and Branch
```java
public class Root extends Branch
{
    @Override
    public boolean isValid()
    {
        return true;
    }
}

public abstract class Branch extends Leaf
{

    public final List<Leaf> children;

    public Branch()
    {
        this.children = new LinkedList<>();
    }


    public Branch addLeafs(Leaf... leaves)
    {
        Collections.addAll(this.children, leaves);
        return this;
    }


    @Override
    public int onLoop()
    {
        return children.stream()
                .filter(c -> Objects.nonNull(c) && c.isValid())
                .findAny()
                .map(Leaf::onLoop)
                .orElseGet(ReactionGenerator::getPredictable);
    }
}
```

The Root is a specialized Branch with a guaranteed validity, ensuring it always processes its children. Branch nodes manage their children, facilitating the addition of Leaves or other Branches and dictating the execution flow based on their validity.

### Leaf
```java
public abstract class Leaf
{
    public abstract boolean isValid();
    public abstract int onLoop();
}
```

Leaves are the execution endpoints, implementing custom logic defined in `onLoop()` and controlled execution through `isValid()`.

## Conclusion
In practical terms, the Tree-Branch-Leaf framework allows for complex, condition-based execution flows within scripts. Developers can structure their logic in a hierarchical manner, dynamically enabling or disabling execution paths based on runtime conditions.

This approach provides a flexible, maintainable way to manage execution logic, especially in scenarios where conditional logic and repetitive tasks are prevalent. By leveraging this framework, developers can create scalable and efficient scripts.