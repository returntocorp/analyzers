# `r2c/typed-ast`

This analyzer produces [ESTree](https://github.com/estree/estree)-compatible
ASTs for each JavaScript file in its input. In addition, it runs TypeScript's
type inference engine to annotate as many nodes as possible in the tree with
their inferred types. Specifically, we add a new property named `inferredType`
whose value is TypeScript's type for that node. This can either be a string such
as `"any"` or `"{foo: number}"`, but can also be a numeric literal such as `3`.

This supports module resolution within the target, but does not support trying
to infer types from any node modules the target depends on. However, it is aware
of the types of builtin Node modules.

In terms of language features, this is able to parse everything up to and
including ES9 and is aware of the types of all language builtins. Type inference
is somewhat in flux, but it supports things like inference for await statements
and async functions.

## Output format

The output is a filesystem hierarchy with one file for each JavaScript file in
the input we were able to successfully parse, with `.ast.json` added to its
extension. For example, `foo/bar/baz.js` will be output as
`foo/bar/baz.js.ast.json`.

Each individual JSON file is the syntax tree of the corresponding input file in
[ESTree](https://github.com/estree/estree) format, with line numbers included. We also augment it with type information where available.

### Type information

Some nodes will have an `inferredType` property. If present, this is the string
representation of the inferred type in the same output format that TypeScript
uses. For example:

- In `const x = "blah"`, the type of `x` is inferred to be `"blah"` (i.e., the
  `inferredType` is a string that contains quotation marks). Similarly, in
  `const x = 3`, the `inferredType` of `x` will be the number 3.
- Types of functions include their parameter names: the type of `Math.round` is
  `(x: number) => number`.

For more documentation on the syntax, see the TypeScript handbook's
documentation on [basic
types](https://www.typescriptlang.org/docs/handbook/basic-types.html), [advanced
types](https://www.typescriptlang.org/docs/handbook/advanced-types.html). For
documentation about the inference engine, see the handbook's section on [type
inference](https://www.typescriptlang.org/docs/handbook/type-inference.html).

Some caveats:

- Many nodes will have type `any`, since they don't have a well-defined type,
  such as nodes corresponding to variable declarations. We may simply not
  include an `inferredType` on those nodes in the future.
- We include type definitions for node.js and browser builtins, but nothing else.
- A function like `const id = x => x` will have its type inferred as `(x: any) => any`, even though we know its type is actually `<T>(x: T) => T`. This is a
  TypeScript limitation.

We do intend to provide a more machine-readable format for types; stay tuned!

## Future features

- TypeScript
- Flow
- Going above and beyond TypeScript's inference
- Providing the types in a more machine-parseable form.
- Looking at the types of dependencies
