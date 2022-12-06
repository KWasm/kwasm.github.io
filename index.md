## Supported Kubernetes Distributions 
![support matrix](./assets/images/support_matrix.png)

## KWasm Operator Quickstart
<ul class="tab" data-tab="1e081df9-bdc6-4e81-8552-aa0a915568e9" data-name="install">
  
      <li class="active">
          <a href="#">kind </a>
      </li>
  
      <li>
          <a href="#">MiniKube </a>
      </li>
  
      <li>
          <a href="#">MicroK8s </a>
      </li>
  
      <li>
          <a href="#">Azure AKS </a>
      </li>
  
      <li>
          <a href="#">DO Kubernetes </a>
      </li>
  
      <li>
          <a href="#">GCP GKE </a>
      </li>
  
      <li>
          <a href="#">AWS EKS </a>
      </li>
  
</ul>
<ul class="tab-content" id="1e081df9-bdc6-4e81-8552-aa0a915568e9" data-name="install">
  
      <li class="active">
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kind create cluster
</code></pre></div></div>
</li>
  
      <li>
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>minikube start <span class="nt">--container-runtime</span><span class="o">=</span><span class="s1">'containerd'</span>
</code></pre></div></div>
</li>
  
      <li>
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>microk8s <span class="nb">install
</span>microk8s status <span class="nt">--wait-ready</span>
</code></pre></div></div>
</li>
  
      <li>
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>az group create <span class="nt">--name</span> kwasm <span class="nt">--location</span> eastus
az aks create <span class="se">\</span>
    <span class="nt">--resource-group</span> kwasm <span class="se">\</span>
    <span class="nt">--name</span> kwasm <span class="se">\</span>
    <span class="nt">--node-count</span> 2 <span class="se">\</span>
    <span class="nt">--generate-ssh-keys</span>
</code></pre></div></div>
<p>Or follow the official tutorial: <a href="https://learn.microsoft.com/en-us/azure/aks/tutorial-kubernetes-deploy-cluster">tutorial-kubernetes-deploy-cluster</a></p>
</li>
  
      <li>
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>doctl kubernetes cluster create kwasm
</code></pre></div></div>
</li>
  
      <li>
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code></code></pre></div></div>
</li>
  
      <li>
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>eksctl create cluster <span class="nt">--name</span> wasm-eks <span class="nt">--node-type</span><span class="o">=</span>t3.medium <span class="nt">--without-nodegroup</span> <span class="nt">--version</span><span class="o">=</span>1.23

<span class="c"># ATENTION, you need to choose the right region and ami</span>
<span class="c"># Look up the appropriate ami for your region: https://cloud-images.ubuntu.com/locator/ec2/</span>
<span class="nb">echo</span> <span class="s1">'apiVersion: eksctl.io/v1alpha5
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
      /etc/eks/bootstrap.sh wasm-eks --container-runtime containerd'</span> | eksctl create nodegroup <span class="nt">-f</span> -
</code></pre></div></div>
</li>
  
</ul>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c"># Add HELM repository if not already done</span>
helm repo add kwasm http://kwasm.sh/kwasm-operator/
<span class="c"># Install KWasm operator</span>
helm <span class="nb">install</span> <span class="nt">-n</span> kwasm <span class="nt">--create-namespace</span> kwasm-operator kwasm/kwasm-operator
<span class="c"># Provision Nodes</span>
kubectl annotate node <span class="nt">--all</span> kwasm.sh/kwasm-node<span class="o">=</span><span class="nb">true</span>
</code></pre></div></div>

<ul class="tab" data-tab="38645aeb-b32f-40e5-8b40-5e8065458632" data-name="run">
  
      <li class="active">
          <a href="#">WasmEdge </a>
      </li>
  
      <li>
          <a href="#">Spin </a>
      </li>
  
</ul>
<ul class="tab-content" id="38645aeb-b32f-40e5-8b40-5e8065458632" data-name="run">
  
      <li class="active">
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl apply <span class="nt">-f</span> https://raw.githubusercontent.com/KWasm/kwasm-node-installer/main/example/test-job.yaml
kubectl logs job/wasm-test
</code></pre></div></div>
</li>
  
      <li>
<p>After installing KWasm, start with step 3 of containerd-wasm-shims <a href="https://github.com/deislabs/containerd-wasm-shims#using-a-shim-in-kubernetes">using-a-shim-in-kubernetes</a></p>
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nb">echo</span> <span class="s1">'  apiVersion: node.k8s.io/v1                                           
  kind: RuntimeClass
  metadata:
    name: wasmtime-spin
  handler: spin'</span> | kubectl apply <span class="nt">-f</span> -

<span class="nb">echo</span> <span class="s1">'  apiVersion: apps/v1                                                  
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
'</span> | kubectl apply <span class="nt">-f</span> -

<span class="c"># Finally test the hello spin app 🥳</span>
kubectl port-forward deployment/wasm-spin 8000:80&amp;
curl localhost:8000/hello
</code></pre></div></div>
</li>
  
</ul>
