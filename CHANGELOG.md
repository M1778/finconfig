# Changelog

All notable changes to this project will be documented here.

## [Unreleased]

### Added
- finconfig 0.1.0: layered configuration — `.env` + INI parsing
  (comments, `export ` prefix, single/double-quoted values with escapes,
  ` #` comment stripping, `[section]` → dotted keys, last-wins
  duplicates, malformed lines counted in `Cfg.skipped`), precedence
  override > env (mapped spelling: uppercased, `.`/`-` → `_`) > file >
  default, never-blaming typed getters (`cfg_get_int` with int32 range
  guard, `cfg_get_bool`, `cfg_get_list`), single-pass `${KEY}`
  interpolation (`$$` escape, unknown/unterminated left literal).
  Tests + example pass against finc 0.4.0 (exit 0), including live
  env-var precedence. Full lifecycle: README, `finn.toml`, CI, MIT.
  Status: VERIFIED (finc 0.4.0), push-ready, NOT pushed — awaiting
  Eleanor's approval to create M1778/finconfig.
