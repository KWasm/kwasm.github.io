# Introduction

Kwasm is a Kubernetes Operator that adds WebAssembly support to your Kubernetes nodes. It does so
by using a container image that contains binaries and configuration variables needed to run pure
WebAssembly images.

:warning: Only for development or evaluation purpose. Your nodes may get damaged!

The operator uses the [kwasm-node-installer](https://github.com/KWasm/kwasm-node-installer) project to modify the nodes

## Supported Kubernetes Distributions

![Support](img/support_matrix.png)
