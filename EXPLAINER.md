# Current solutions
It seems that several JS libraries already exist that offer satisfactory worker DOM implementations. [Virtual DOM](https://github.com/Matt-Esch/virtual-dom) and React's render model both seem effective. However, as adoption of these libraries increase, it may be worthwhile to 'pave the cow path' and build these constructs into the platform.

# `WorkerNode`
This proposal specifies a new immutable object model for use inside workers or on the main thread. The objects provide the instructions for how to build a DOM subtree and how to inflate that subtree into a real DOM subtree. `WorkerNode` is intended to work in tandem with [async append](https://github.com/drufball/async-append).

This has advantages over postMessage and innerHTML because the engine knows intent when creating a CompactNode tree and can pre-allocate data structures that the elements will eventually be inflated into. There are also ergonomic benefits to using nodes and tree structures that developers are already familiar with.

This proposal uses JS objects to represent DOM, which incurs some overhead from binding costs. In our experience analyzing pages, the overhead of bindings is negligible. The increased ergonomics from using familiar DOM manipulation paradigms greatly outweighs any small performance impact from bindings.

## API
__Note: This API is very much a rough draft / strawman. Suggestions for improvement welcome.__

```
[Exposed=(Window,Worker)]
interface CompactNode { };

[NoInterfaceObject]
interface CompactParentNode extends CompactNode {
  readonly attribute sequence<CompactNode> childNodes;
};

[Constructor(String name, String value),
 Exposed=(Window,Worker)]
interface CompactAttr {
  readonly attribute String name;
  readonly attribute String value;
};

[Constructor(String tagName, sequence<CompactAttr> attributes, sequence<CompactNode> childNodes, sequence<CompactShadowRoot> shadowRoots),
 Exposed=(Window, Worker)];
interface CompactElement extends CompactParentNode {
  readonly attribute String tagName;
  readonly attribute sequence<CompactAttr> attributes;
};

[Constructor(sequence<CompactNode> childNodes),
 Exposed=(Window, Worker)];
interface CompactShadowRoot extends CompactParentNode {};

[Constructor(String data),
 Exposed=(Window, Worker)]
interface CompactText extends CompactNode {
  readonly attribute data;
};

typedef (NodeInflationRequest or CompactNode) NodeInflationInfo;

[Constructor(CompactNode node, optional NodeInflationInit init),
 Exposed=Window]
interface NodeInflationRequest {
  readonly attribute CompactNode node;
read
  readonly unsigned long priority;
  void cancel();
};

dictionary NodeInflationInit {
  unsigned long priority = 0;
};

partial interface Node {
  static Promise<Node> inflate(NodeInflationInfo target, optional NodeInflationInit init);
};
```
