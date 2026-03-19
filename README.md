# renderview

A protocol for interactive rendering surfaces

RenderView defines a simple specification for a renderable surface that emits
structured input events (pointer, keyboard, resize) and displays frames. It can
be implemented in most programming languages, browsers, native applications,
remote renderers, notebooks, etc.

This repository contains the specification and a reference implementation in JavaScript.

Read the spec at https://pygfx.org/renderview/


## Contributing

The JS code is formatted and linted with `standardjs`, which can be installed using `npm install --global standard`.

To lint/format, run `standard src --fix` in the root of the repo before committing.
