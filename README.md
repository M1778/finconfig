# finconfig

Layered configuration for Fin: `.env` files, INI sections, environment
variables, and explicit overrides — one precedence order, typed getters
that never blame.

> **Verification status:** VERIFIED against finc 0.4.0 (contract 3):
> full test suite passes with exit 0 and the example runs, including
> live env-var precedence (`APP_PORT=9090` beats the file value).

## Why not std `cli` / `argparse`?

Upstream `lib/std/` already covers command-line parsing: `argparse.fin`
(flags/options/positionals) and the newer `cli.fin` (real argv via
`/proc/self/cmdline`, subcommands, help/version, env passthrough).
What was still missing is the *config file* half: nobody parses `.env`
or INI files, layers file + env + overrides, or answers typed values
without blaming. That is this library's niche — it complements `cli.fin`,
it does not compete with it. Pair them: `cli.fin` for argv, finconfig
for everything else.

## Install

Requires the `finn` package manager (https://github.com/M1778/finn):

```sh
finn add M1778/finconfig
```

## Quick Example

```fin
import { Cfg, cfg_new, cfg_free, cfg_load_file, cfg_get,
         cfg_get_int, cfg_get_bool } from finconfig::fin;

fun main() <noret> {
    let c <Cfg> = cfg_new();
    cfg_load_file(&c, ".env");              // -1 when unreadable
    printf("port: %d\n", cfg_get_int(&c, "APP_PORT", 80));
    printf("debug: %d\n", cfg_get_bool(&c, "DEBUG", false));
    cfg_free(&c);
}
```

> Import spelling: `finconfig::fin` means module `finconfig` (the file
> `src/finconfig.fin`), namespace `fin`. Import the names you need
> **by name** — qualified calls (`fin.cfg_get(...)`) are refused by the
> backend; see `docs/guide/10-modules-and-imports.md` in the Fin repo.

## Precedence

1. `cfg_set` overrides (highest)
2. Environment variables, under the key's env spelling: uppercased,
   `.` and `-` rewritten to `_` (`db.host` → `DB_HOST`)
3. Loaded files, in load order (last wins)
4. The caller-supplied default (lowest)

An explicitly empty env var is a value, not a miss: it answers `""`,
matching `cli.fin`'s `get_env_or` convention.

## File syntax

One unified parser for `.env` and INI:

```ini
# comments start with # or ;
export APP_ENV=production      # `export ` prefix accepted
APP_PORT = 8080                # spaces around = trimmed
EMPTY=
HASHED=abc#def                 # # without a space is kept
NOTE=value # trailing note     # ` #` comment stripped
SINGLE='literal $x \n'        # single quotes: literal, no escapes
DOUBLE="a\nb\t\"q\"\\"         # double quotes: \\ \n \t \r \" escapes

[db]                           # sections become dotted keys
host = localhost               # read back as db.host
```

Duplicate keys: last wins. Malformed lines (no `=`, empty key) are
skipped and counted in `Cfg.skipped` — loading never aborts.

## API Reference

| Function | Description |
|---|---|
| `cfg_new() <Cfg>` | Fresh, empty configuration. |
| `cfg_free(c: &Cfg) <noret>` | Frees all stored keys/values and maps. |
| `cfg_load_text(c, text) <int>` | Parses a document into the file layer; returns keys loaded. |
| `cfg_load_file(c, path) <int>` | Reads + parses a file; `-1` when unreadable. |
| `cfg_set(c, key, val) <noret>` | Explicit override (clones key and value). |
| `cfg_has(c, key) <bool>` | True when any layer provides the key. |
| `cfg_get(c, key, default) <string>` | Full precedence lookup. Borrowed — never free. |
| `cfg_get_file(c, key) <string>` | File layer only (no override, no env). Borrowed. |
| `cfg_get_int(c, key, default) <int>` | Never blames: missing, non-numeric, or out-of-range values answer the default. Accepts optional `+`/`-`. |
| `cfg_get_bool(c, key, default) <bool>` | True: `1 true yes on`; false: `0 false no off` (case-insensitive); else default. |
| `cfg_get_list(c, key, sep) <&Collection<string>>` | Splits a value on a char. Fresh heap: free each part with `free_str`, then `delete` the container. Missing keys give an empty collection. |
| `cfg_expand(c, value) <string>` | Expands `${KEY}` (full precedence), one pass, no recursion. `$$` → `$`; unknown/unterminated refs stay literal. Fresh heap — `free_str` it. |
| `cfg_get_expanded(c, key, default) <string>` | `cfg_get` with expansion applied. Fresh heap. |

Ownership: `cfg_set` clones, so a caller may free its key/value right
after the call. `cfg_get` results are borrowed into the `Cfg` (or
process-owned env strings, or the caller's default) — never free them.

## Known limitations (v0.1.0)

- No float getter: `strings::std` has no float parser, and inventing
  one here would duplicate std's job. `cfg_get` + a future std float
  parse covers it.
- No config *writing*: read-only by design for now; `fs::std` covers
  the write side if a caller needs it.
- INI only takes `=` pairs (no `key: value` form) and no multi-line
  values; both are documented, not silent.
- Interpolation is single-pass by design: a referenced value's own
  `${...}` is left literal, so expansion cannot loop.

## License

MIT
