# Introduction

Kwasm is a Kubernetes Operator that adds WebAssembly support to your Kubernetes nodes. It does so
by using a container image that contains binaries and configuration variables needed to run pure
WebAssembly images.

!!! warning
    This project is mean to be used for **development** or **evaluation** purpose. Your nodes may get damaged!

The operator uses the [kwasm-node-installer](https://github.com/KWasm/kwasm-node-installer) project to modify the underlying Kubernetes nodes.

## How it works
Nigel Poulton describes in his [blog article](https://nigelpoulton.com/webassembly-and-containerd-how-it-works/) in detail what is needed to use containerd Wasm shims with Kubernetes.


## Supported Kubernetes Distributions

![Support](img/support_matrix.png)
