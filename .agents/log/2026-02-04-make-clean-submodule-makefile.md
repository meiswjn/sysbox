# 2026-02-04: Root Makefile robustness (submodule makefile precedence)

## Problem
User hit `make[1]: *** No rule to make target 'clean'. Stop.` when root `make sysbox` invoked `make clean` inside `sysbox-runc`.

A plausible cause is the presence of a lowercase `makefile` in the submodule directory (GNU make searches `GNUmakefile`, then `makefile`, then `Makefile`). If `makefile` exists but lacks a `clean` target, `make clean` fails even if `Makefile` contains it.

## Fix
Updated root `Makefile` to invoke submodule targets via `$(MAKE) -C <dir> -f Makefile ...` for build/tidy/clean/distclean targets, ensuring the intended `Makefile` is used even if a stray lowercase `makefile` exists.

## File changed
- Makefile
