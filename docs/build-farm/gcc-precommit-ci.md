# RISE GCC pre-commit CI

## Overview

The RISE Build Farm hosts a pre-merge CI build system for GCC.

RISE's pre-merge CI build system helps answer the question of whether a proposed patch meets the minimum requirements to be merged into GCC, as an aid to maintainers and developers.

## Project Leaders

* Edwin Lu (Rivos)
* Patrick O'Neill (Rivos)

## Value

This automated pre-commit CI process saves developers hours of what would be manual testing time, accelerating the development of GCC and increasing codebase quality with consistent testing. It helps reviewers understand whether a patch is suitable for inclusion in the codebase.

## Reports

* The activity of the RISE GCC pre-merge CI system can be monitored here: [https://github.com/ewlu/gcc-precommit-ci/issues](https://github.com/ewlu/gcc-precommit-ci/issues)

## Process

When a new patch with "RISC-V" in the title is submitted to the mailing list the precommit pulls it from [patchworks](https://patchwork.sourceware.org/project/gcc/list/?series=&submitter=&state=&q=RISC-V&archive=&delegate=). A variety of checks are then run using the post-commit CI as a baseline and source-of-truth for expected failures. The checks include:

* Does the patch apply?
* Does the patch pass lint checks?
* Does the patch build, after applying?
* Does the patched compiler work?

The build/test logic is as follows:

```
if patch_applies_to_baseline:
    perform build+test, comparing testsuite results with the baseline testsuite results
if patch_does_not_apply_to_baseline and patch_applies_to_trunk:
    perform build
if patch_does_not_apply_to_baseline_or_trunk:
    report a warning
```

Results are reported to [patchworks](https://patchwork.sourceware.org/project/gcc/list/?series=&submitter=&state=&q=RISC-V&archive=&delegate=) and may be manually interpreted/directly emailed to upstream maintainers.

The pre-commit CI focuses on a variety of GCC targets, including application processors, long vector accelerators, and microcontrollers.

## Future Work

* Can we add a dashboard / set of charts to depict progress over time?

## References

* GCC Pre-commit CI Documentation: [https://gcc.gnu.org/wiki/PreCommitCI](https://gcc.gnu.org/wiki/PreCommitCI)
* Precommit CI: [https://github.com/ewlu/gcc-precommit-ci](https://github.com/ewlu/gcc-precommit-ci)
