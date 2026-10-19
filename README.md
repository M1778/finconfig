# {{PKG_NAME}}

{{DESCRIPTION}}

> **Verification status:** UNVERIFIED until CI passes — written against the Fin
> v0.4.0 [agent guide](https://github.com/M1778/Fin/blob/master/docs/agent-guide.md).

## Install

Requires the `finn` package manager (https://github.com/M1778/finn):

```sh
finn add M1778/{{PKG_NAME}}
```

## Quick Example

```fin
import { /* symbols */ } from lib::{{MODULE}};

fun main() <noret> {
    // TODO: add example
}
```

> Import spelling: `lib::{{MODULE}}` means module `lib` (the file `src/lib.fin`),
> namespace `{{MODULE}}`. Import the names you need **by name** — qualified
> calls (`{{MODULE}}.foo(...)`) are refused by the backend; see
> `docs/guide/10-modules-and-imports.md` in the Fin repo.

## API Reference

<!-- TODO: document public API -->

## License

MIT
