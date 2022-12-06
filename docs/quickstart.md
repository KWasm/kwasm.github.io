# Quickstart

## Installation

A Helm chart is available to easy install the operator

```bash
# Add HELM repository if not already done
helm repo add kwasm http://kwasm.sh/kwasm-operator/
# Install KWasm operator
helm install -n kwasm --create-namespace kwasm-operator kwasm/kwasm-operator
# Provision Nodes
kubectl annotate node --all kwasm.sh/kwasm-node=true
```

## Platform based configuration

Depending on which platform you're experimenting with the Kwasm Operator certain configuration needs to be done to ensure proper functionality.

The following configuration include the instruction to create a fresh cluster in case you haven't done that already.

=== "Kind"
    ```bash
    kind create cluster
    ```

=== "Minikube"

    ```bash
    minikube start --container-runtime='containerd'
    ```

=== "MicroK8s"

    ```bash
    microk8s install
    microk8s status --wait-ready
    ```

=== "Azure AKS"
    ```bash
    az group create --name kwasm --location eastus
    az aks create \
    --resource-group kwasm \
    --name kwasm \
    --node-count 2 \
    --generate-ssh-keys
    ```

    Or follow the official [Azure guide](https://learn.microsoft.com/en-us/azure/aks/tutorial-kubernetes-deploy-cluster?tabs=azure-cli)

=== "AWS EKS"
    ```bash
    eksctl create cluster --name wasm-eks --node-type=t3.medium --without-nodegroup --version=1.23

    # ATENTION, you need to choose the right region and ami
    # Look up the appropriate ami for your region: https://cloud-images.ubuntu.com/locator/ec2/
    echo 'apiVersion: eksctl.io/v1alpha5
    kind: ClusterConfig
    metadata:
      name: wasm-eks
      region: us-west-2
    managedNodeGroups:
      - name: ng-wasm-eks
        ami: ami-0809c110f957a5eb8
        instanceType: t3.medium
        minSize: 1
        maxSize: 2
        overrideBootstrapCommand: |
          #!/bin/bash
          /etc/eks/bootstrap.sh wasm-eks --container-runtime containerd' | eksctl create nodegroup -f -
    ```

=== "GCP GKE"
    ```bash
    ```

=== "DO Kubernetes"
    ```bash
    doctl kubernetes cluster create kwasm
    ```

## WASM Runtime configuration

Depending on the wasm runtime you want to use the following configuration needs to be done.

=== "WASMEDGE"
    The following definition can be applied to create a test wasm workload
    ```yaml
      apiVersion: node.k8s.io/v1
      kind: RuntimeClass
      metadata:
        name: crun
      handler: crun
      ---
      apiVersion: batch/v1
      kind: Job
      metadata:
        creationTimestamp: null
        name: wasm-test
      spec:
        template:
          metadata:
            annotations:
              module.wasm.image/variant: compat-smart
            creationTimestamp: null
          spec:
            containers:
            - image: wasmedge/example-wasi:latest
              name: wasm-test
              resources: {}
            restartPolicy: Never
            runtimeClassName: crun
        backoffLimit: 1

    ```

    or apply the file from source
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/KWasm/kwasm-node-installer/main/example/test-job.yaml
    kubectl logs job/wasm-test
    ``` 

=== "Spin"
    After installing KWasm, start with step 3 of containerd-wasm-shims [using-a-shim-in-kubernetes](https://github.com/deislabs/containerd-wasm-shims#using-a-shim-in-kubernetes)

    ```bash
    echo 'apiVersion: node.k8s.io/v1                                           
      kind: RuntimeClass
      metadata:
        name: wasmtime-spin
      handler: spin' | kubectl apply -f -

    echo 'apiVersion: apps/v1                                                  
      kind: Deployment
      metadata:
        name: wasm-spin
      spec:
        replicas: 1
        selector:
          matchLabels:
            app: wasm-spin
        template:
          metadata:
            labels:
              app: wasm-spin
          spec:
            runtimeClassName: wasmtime-spin
            containers:
            - name: spin-hello
              image: ghcr.io/deislabs/containerd-wasm-shims/examples/spin-rust-hello:latest
              command: ["/"]
    ' | kubectl apply -f -

    # Finally test the hello spin app 🥳
    kubectl port-forward deployment/wasm-spin 8000:80
    curl localhost:8000/hello
    ```
