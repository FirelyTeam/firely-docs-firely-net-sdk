# Navigating POCOs as a tree

```{note}
This section describes advanced functionality that can be safely skipped when you are learning the basics of the SDK.
```

Using the techniques described in [the section on dynamic features](dynamic-features), you can use `EnumerateElements()` and `TryGetValue()` to navigate the POCO model dynamically. By applying these methods recursively you can walk the entire POCO tree. In some scenarios, however, navigating only downward is not enough: both FhirPath and the validator need a couple of additional capabilities:

* Ability to navigate back to the parent node
* Ability to obtain a path from the root to the current node

Users of earlier SDK versions may recognize these features from the `ITypedElement` interface. Two limitations of that approach were that `ITypedElement` did not expose a parent (unless you wrapped nodes with `ScopedNode`) and it was a separate abstraction outside the POCO model, which made direct access to POCO-specific behavior harder.

SDK6 introduces `PocoNode`, a lightweight wrapper around POCOs that presents a POCO as a node in a tree. For backwards compatibility `PocoNode` implements both the `ITypedElement` and `ISourceNode` interfaces, so it can be used wherever `ITypedElement` was used before, while still providing direct access to the underlying POCO.

You can obtain a `PocoNode` for any element by calling `ToPocoNode()` on any `Base` instance.

```{note}
`ToPocoNode()` accepts an optional `ModelInspector` (see [model metadata](model-metadata)). Providing a `ModelInspector` is useful when deriving accurate type information for `ITypedElement` or for FhirPath's `ofType()` when working with shared datatypes (see [datatypes](datatypes.md)). In those cases the concrete POCO types may not fully align with the FHIR specification for a particular version.
```

Once you have a `PocoNode`, you can navigate both up and down the tree using the `Parent` property and the `Children()` method (to enumerate all children). Use `Child()` to get a specific child by name.

When you navigate down from the node created by `ToPocoNode()`, the `PocoNode` instances record the path back to that root node. Each `PocoNode` therefore exposes:

* The node `Name`
* The `Index` if the node is an element in a list

Use the extension method `GetLocation()` to obtain the full path from the root to the current node as a string.

```{note}
The parent pointers and location are established when you call `ToPocoNode()`. Holding references to parent resources can retain large object graphs in memory (for example, large Bundles), so exercise caution in memory-constrained or performance-critical environments.
```

## Nodes which are lists

In the POCO tree, a node can be either a single node (represented by `PocoNode`) or a list of nodes (represented by `PocoNodeList`). Both derive from the abstract class `PocoNodeOrList` and implement `IEnumerable<PocoNode>`. This allows you to traverse the tree using LINQ — for example `SelectMany(Children)` — similar to how `ITypedElement` is often consumed.

Because any child can be a single POCO or a collection of POCOs, the return type of `Parent`, `Child()` and `Children()` is `PocoNodeOrList`. Likewise, a node itself may be an element of a parent list, so the `Parent` property is also a `PocoNodeOrList`.

## Common uses for `PocoNode`

The SDK provides several extension methods that leverage the extra information in `PocoNode`. These make `PocoNode` a suitable replacement for the earlier `ScopedNode` used by the FhirPath engine and the profile validator.

Examples include:

* `Resolve()` — resolves a local reference in the POCOs represented by this `PocoNode`.
* `ContainedResources()` — returns the nodes that represent contained resources.
* `Descendants()` — returns all child nodes recursively.
* `NavigateTo()` — navigates to a simple dotted path below the current node.

These utilities make tree-based operations and validation straightforward while keeping direct access to the underlying POCOs. Navigating POCOs as a tree
