# RISE GCC post-commit CI

## Overview

The RISE Build Farm hosts a post-merge CI build system for GCC.

The RISE post-merge CI build system monitors the health of the main development branch for GCC (trunk) and release branches.

## Project Leaders

* Edwin Lu (Rivos)
* Patrick O'Neill (Rivos)

## Value

Post-commit CI tightens the feedback cycle time between the point in time when a patch is committed to a branch, and when an issue is discovered with that patch (ideally before the original developer moves on to other work). It catches bugs that the pre-commit CI misses, including integration bugs that only appear once other patches are merged. This helps keep bugs from slipping into releases, and improves developer productivity by keeping branches healthy.

One of the key advantages of post-commit CI is that it's possible to test more targets, since more time-consuming tests can be run. Post-commit CI can also find problems introduced by patches that initially appear to be unrelated to RISC-V (since pre-commit CI only runs on patches that appear to pertain to RISC-V).

Post-commit CI makes identifying regressions much easier. It helps identify when a development branch is broken. Compared to the pre-commit CI, we test more targets with post-commit CI triaging unintended breakages.

## Process

The post-commit CI runs on a regular cadence and performs a full build and test for a variety of targets. Most targets run every six hours, but more obscure configurations are run on a weekly basis. Assuming the build+test finishes without issue, those testsuite results are registered as a new baseline.

Release branches are tested less frequently, since they have less activity than the main release branch.

The post-commit CI focuses on a variety of GCC targets, including application processors, long vector accelerators, and microcontrollers.

An example output from postcommit ci: [https://github.com/patrick-rivos/gcc-postcommit-ci/issues/86](https://github.com/patrick-rivos/gcc-postcommit-ci/issues/86)

## References

* Postcommit CI: [https://github.com/patrick-rivos/gcc-postcommit-ci](https://github.com/patrick-rivos/gcc-postcommit-ci)
* Postcommit CI dashboard: [https://patrick-rivos.github.io/gcc-postcommit-ci/](https://patrick-rivos.github.io/gcc-postcommit-ci/)
