# RISE Python RISC-V Wheel Builder

## Summary

The RISE Python RISC-V Wheel builder fills a major gap in the Python language ecosystem for RISC-V. The Wheel builder project provides pre-built, tested Python wheels for Python projects such as NumPy, that aren't yet available upstream due to the youth of the RISC-V hardware and software ecosystems. For Python packages that contain code in languages other than Python that must be compiled (such as NumPy, SciPy, Pillow, matplotlib, PANDAS), this project builds and tests binary wheels. This significantly eases the installation and use not only of these packages, but also of many larger packages such as PyTorch, TensorFlow, VLLM, and Jupyter Notebook. All of the packages that this project builds and tests are uploaded to the RISE package repository.

This blog post, [https://riseproject.dev/2025/05/14/easy-installation-of-binary-python-packages-on-riscv64-devices/](https://riseproject.dev/2025/05/14/easy-installation-of-binary-python-packages-on-riscv64-devices/), adds further detail about what this project does and why it's important to the RISC-V software ecosystem.

## Project Leaders

* Mark Ryan (Rivos)

## Value

The Wheel builder project provides pre-built, tested Python wheels for Python projects such as NumPy, that aren't yet available upstream due to the youth of the RISC-V hardware and software ecosystems. For Python packages that contain code in languages other than Python that must be compiled (such as NumPy, SciPy, Pillow, matplotlib, PANDAS), this project builds and tests binary wheels. This significantly eases the installation and use not only of these packages, but also of many larger packages such as PyTorch, TensorFlow, VLLM, and Jupyter Notebook. All of the packages that this project builds and tests are uploaded to the RISE package repository.

An additional benefit of this project is that we unmask hidden build problems with Python packages on RISC-V. Many of the packages that we build initially don't work when built for RISC-V. When a package is initially added to this project, we fix any build bugs and upstream the fixes, along with any non-Python projects that the packages depend on (such as OpenBLAS). So this project also contributes to the overall health and ease of use of the entire RISC-V software ecosystem touched by this work - not only Python packages.

## Results

Our current list of project milestones, automatically updated, is here: [https://gitlab.com/riseproject/python/wheel_builder/-/milestones](https://gitlab.com/riseproject/python/wheel_builder/-/milestones)

Here is a list of packages currently provided by this project: [https://riseproject.gitlab.io/python/wheel_builder/packages/supported_packages.html](https://riseproject.gitlab.io/python/wheel_builder/packages/supported_packages.html)

We've built all of the dependencies for PyTorch and most of the dependencies of VLLM.

## Technical Details

The project uses two builders: one builder on x86-64, and one builder running on a Scaleway EM-RV1 instance. Some tests don't complete on QEMU, while others don't complete on native RISC-V (due to memory constraints with current hardware, or ISA limitations, for example).

When one builds a Python package, typically one has to build at least four copies of it: one for each Python version one wishes to support. And then additionally these four package versions must all be tested. This process can take hours, and so is dependent on high performance machine instances.

**We are actively soliciting for more volunteers from RISE members to add more Python wheels to this project. Please join the RISC-V Developer Infrastructure mailing list if you'd like to help.**

## Future Work

* Dependencies for SGLang and VLLM.
* We also are planning to build and provide the top 30 most popular Python binary wheels.
* In the long term, we want to transfer building and testing of these wheels to the Python community at large, once the RISC-V ecosystem further matures. \[Ludovic Henry (Rivos)\]

## References

* [https://riseproject.dev/2025/05/14/easy-installation-of-binary-python-packages-on-riscv64-devices/](https://riseproject.dev/2025/05/14/easy-installation-of-binary-python-packages-on-riscv64-devices/)
* [The wheel_builder repository](https://gitlab.com/riseproject/python/wheel_builder)
* [The wheel_builder documentation](https://riseproject.gitlab.io/python/wheel_builder/)
* [Towards seamless Python package installation on riscv64](https://fosdem.org/2025/schedule/event/fosdem-2025-5659-towards-seamless-python-package-installation-on-riscv64/) – a FOSDEM 2025 talk on the wheel_builder project and the difficulties of building binary wheels for riscv64.
* [Packaging support for riscv64](https://discuss.python.org/t/packaging-support-for-riscv64/58475) – a discussion of the issues preventing upstream projects from building riscv64 wheels.
