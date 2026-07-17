# RISE Linux Kernel Pre-Commit CI

## Summary

RISE hosts pre-merge CI build systems for the [RISC-V architecture builds in the Linux kernel](https://github.com/linux-riscv/).

This project is intended to help answer the question of whether a patch meets some of the minimum requirements to be merged into the Linux kernel, as an aid to maintainers and developers.

## Process

* Patchwork scrapes the mailing list
* Daemon synchronizes Patchwork with Github
* Builds and tests run on RISE Build Farm dedicated runners
* Feedback is provided to maintainers, reviewers, and contributors

## Status

Project is up and running.

## Project Sponsors

* Björn Töpel (Rivos)
