!!! warning
    The KWasm found a new home in the [Spinkube](https://www.spinkube.dev/) project, now developed as [runtime-class-manager](https://github.com/spinkube/runtime-class-manager). This new project is KWasms spiritual successor and is going to support use cases and more.

# Introduction

Kwasm is a Kubernetes Operator that adds WebAssembly support to your Kubernetes nodes. It does so
by using a container image that contains binaries and configuration variables needed to run pure
WebAssembly images.

!!! warning
    This project is mean to be used for **development** or **evaluation** purpose. Your nodes may get damaged!

The operator uses the [kwasm-node-installer](https://github.com/KWasm/kwasm-node-installer) project to modify the underlying Kubernetes nodes.

## Supported Kubernetes Distributions

![Support](img/support_matrix.png)
