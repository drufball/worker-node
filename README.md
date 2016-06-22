# worker-node
Allowing DOM operations on worker threads

# Problem
In order to meet the [RAIL performance model](https://developers.google.com/web/tools/chrome-devtools/profile/evaluate-performance/rail?hl=en), sites must respond to user input within __100ms__. For large, complex web apps this means that every ms of work counts. If the main thread isn't free within 100ms, the user will experience jank.

The two methods of protecting the main thread are to chunk work into small pieces or to offload large tasks to separate threads. When working with DOM, though, neither of these approaches is feasible. The rendering lifecycle executes as a monolithic chunk, easily costing __30-60ms__ each time a new layout is required. [async-append](https://github.com/drufball/async-append) is a proposal to make the rendering lifecycle chunkable to avoid this.

In addition to the rendering lifecycle, there is often a lot of JS work required to calculate what DOM mutations are necessary. This is especially true of large apps which have thousands of elements on the page. Because DOM can only be created/manipulated on the main thread, it is difficult to calculate DOM changes without also doing it on the main thread.

# Worker DOM
In order to allow developers to move their long-running DOM manipulation code off the main thread, we need to allow them to interact with DOM in workers. This is more complex than it may seem at first. Certain DOM operations, such as reading `height` and `width` attributes, requires a new layout which can only be done on the main thread. For more explorations of potential solutions, see the [EXPLAINER](EXPLAINER.md).

There are several existing JS libraries that offer [virtual DOM](https://github.com/Matt-Esch/virtual-dom), a lightweight JS representation of DOM that can be manipulated from workers. These frameworks get around the layout issue mentioned above by restricting what actions are allowed in workers. Essentially, only operations that do not require a layout are allowed.
