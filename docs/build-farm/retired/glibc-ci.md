# RISE GLIBC pre-merge CI (inactive)

## Summary

RISE hosted pre-merge CI build systems for the GNU implementation of libc, GLIBC. This project is inactive as of 1Q 2025.

This project was intended to help answer the question of whether a patch meets some of the minimum requirements to be merged into GLIBC, as an aid to maintainers and developers.

## Process

When a new patch with "RISC-V" in the title is submitted to the mailing list, the precommit pulls it from [patchwork](https://patchwork.sourceware.org/project/glibc/list/?series=&submitter=&state=&q=RISC-V&archive=&delegate=). A variety of checks (does the patch apply, linting, build, test) are then run. If the patch applies, the CI will start two runs: one using the current tip of tree and the other using the tip of tree+applied patch. Once the two runs have completed, the testsuite results are diffed and reported.

Results are reported to [patchwork](https://patchwork.sourceware.org/project/glibc/list/?series=&submitter=&state=&q=RISC-V&archive=&delegate=) and may be manually interpreted/directly emailed to upstream maintainers.

An example output from precommit ci: [https://github.com/ewlu/riscv-glibc-ci/issues/112](https://github.com/ewlu/riscv-glibc-ci/issues/112)

Sources: [https://github.com/ewlu/riscv-glibc-ci](https://github.com/ewlu/riscv-glibc-ci)

## Status

Project is inactive as of 1Q 2025.

## Project Sponsors

* Edwin Lu (Rivos)
* Patrick O'Neill (Rivos)
