# RISE LLVM Fuzz CI

## Summary

The LLVM Fuzz CI continuously generates random C programs using Csmith and YARPGen, compiles them with LLVM, and runs them. Any outcomes that should never occur, like Internal Compiler Errors (ICEs) or runtime behavior mismatches, are recorded and uploaded as artifacts to Github.

## Project Leaders

* Edwin Lu (Rivos)
* Patrick O'Neill (Rivos)

## Value

This project discovers novel bugs that are not caught by the existing LLVM test suite. By continuously generating and running random C programs, it stress-tests the complex interactions between different codepaths and covers newly added behavior. This has two key benefits:

* It exposes undertested areas in the test suite, guiding the creation of new regression tests.
* It finds bugs and edge cases proactively, allowing developers to fix them before they impact users.

## Results

The project is up and running, and has found 56 LLVM bugs so far. A manually-updated (and thus, likely out of date) list of the framework's discovered bugs can be found here: [https://github.com/patrick-rivos/compiler-fuzz-ci?tab=readme-ov-file#bug-trophy-case](https://github.com/patrick-rivos/compiler-fuzz-ci?tab=readme-ov-file#bug-trophy-case)

Results/run stats can be downloaded using:

```
pip install pygithub
./download.py -outdir out -token <Github token>
```

## Future Work

* Dashboard
* Automation for reducing test cases (as opposed to the current, semi-manual process)

## References

* Sources: [https://github.com/patrick-rivos/compiler-fuzz-ci](https://github.com/patrick-rivos/compiler-fuzz-ci)

Results/run stats can be downloaded using:

```
pip install pygithub
./download.py -outdir out -token <Github token>
```
