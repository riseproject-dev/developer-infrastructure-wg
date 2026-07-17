# PoC Github Runners on GKE (inactive)

## Foreword

RISE DevInfra used to run on Google Cloud Platform, which offered a number of different services, including Google Kubernetes Engine (GKE). This page formerly described how GKE was used to scale RISE Linux Kernel CI testing. Now, the RISE Linux Kernel CI is running on Hetzner Cloud, and we no longer use GKE or Kubernetes.

## Introduction

GKE is Google's Kubernetes solution. GKE/Autopilot is a variant of GKE where the user does not have to explicitly handle the machines in the cluster. Instead VMs are spun up/down based on the number of running containers, and the users only pay for the resources consumed.

Github provides a project called ARC (actions runner controller), which is a Kubernetes controller that can dynamically create and destroy runners based on the current need. ARC can be configured on a per-org/per-repo basis, and also constraining how many Github Runners that are allowed to run.

For projects based on Github Runners, GKE/Autopilot offers a number of benefits to RISE DevInfra:

* Lower costs: Only pay what you use
* Easier administration: A project is configured once, and no additional VM control plane is required

## Github ARC architecture

Kubernetes is a container orchestration tool. ARC is a Kubernetes controller which, will connect one or more listeners to Github projects/organizations. The listener will spawn a Github runner, when one is requested from the Actions infrastructure. Since the actual workflow is run within a container, the Runner will need an additional container, where the workflow is run.

This mode (Runner spawning an additional container) is, weirdly, called 'containerMode: kubernetes'. There are also other modes, in addition to kubernetes mode:

1. The Runner is co-running with the workflow
2. Docker-in-Docker mode

Co-running the Runner with the workflow makes it difficult to update the workflow container. Docker-in-docker does not work on GKE, hence the "kubernetes" mode was picked for the PoC.

Again, in "kubernetes" mode, the Runner spawns a container to run the workflow. The container build resources (CPU/memory/storage) can be configured.

The Runner communicates to the workflow container using a node.js application, running on a shared NFS drive. Each step in the workflow does a k8s exec, and executes the mounted node.js application to perform a step.

## Setup

### Once

Enable GKE, and create an Autopilot cluster. Enable the Cloud Filestore API.
Install the gcloud CLI: [https://cloud.google.com/sdk/docs/install](https://cloud.google.com/sdk/docs/install)
Install helm (Kubernetes package manager): [https://helm.sh/docs/intro/install/](https://helm.sh/docs/intro/install/)

### Per-cluster

Install the ARC controller using helm

```
helm install arc --namespace arc-systems \
--create-namespace \
oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
```

### Per-project

Create a classic Github token with "repo" and "admin" rights

Create a helm chart for the ARC runner set, e.g.:

```yaml
githubConfigUrl: "https://github.com/bjoto/linux" # Change: The project
githubConfigSecret:
  github_token: "GITHUB token here"

maxRunners: 10 # Change: The maximum amount of spawned runners
minRunners: 0  # Change: The minimum amount of spawned runners

containerMode:
  type: "kubernetes"

template:
  spec:
    securityContext:
      fsGroup: 123
    containers:
      - name: runner
        image: ghcr.io/actions/actions-runner:latest
        command: ["/home/runner/run.sh"]
        env:
          - name: ACTIONS_RUNNER_CONTAINER_HOOK_TEMPLATE
            value: "/home/runner/config/linux-worker-podspec.yaml" # Change
          - name: ACTIONS_RUNNER_USE_KUBE_SCHEDULER
            value: "true"
        volumeMounts:
          - name: worker-podspec-volume
            mountPath: /home/runner/config
            readOnly: true
    volumes:
      - name: worker-podspec-volume
        configMap:
          name: linux-worker-cm
          items:
            - key: linux-worker-podspec.yaml # Change
              path: linux-worker-podspec.yaml # Change
      - name: work
        ephemeral:
          volumeClaimTemplate:
            spec:
              storageClassName: "premium-rwx"
              accessModes: [ "ReadWriteMany" ]
              resources:
                requests:
                  storage: 10Gi # Change: This is where the repo is cloned!
```

Install the runner set with:

```
helm upgrade --install linux-arc-runner-set \
  --namespace "arc-runners" --create-namespace \
  -f your-chart-from-above.yaml \
  oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
```

Note the name, "linux-arc-runner-set" which is important.

Now it's time to specify the resources of the workflow container. This is done via config map yaml:

```yaml
spec:
  nodeSelector:
    cloud.google.com/compute-class: Scale-Out
    kubernetes.io/arch: amd64
  securityContext:
    fsGroup: 123
  containers:
  - name: $job
    resources:
      requests:
        cpu: 32
        memory: 96Gi
    volumeMounts:
        - mountPath: "/build"
          name: build-volume
    env:
    - name: ENV1
      value: "ninjahopp-9"
  volumes:
  - name: build-volume
    ephemeral:
      volumeClaimTemplate:
        metadata:
          labels:
            type: ephemeral
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 1Ti
```

Here we're requesting a large machine with 32 CPUs and 96G of memory, some custom environment, and an ephemeral build drive.
Note that the Runner and the workflow container must run on the same architecture.

Add the configmap:

```
kubectl create configmap linux-worker-cm \
  --from-file=worker-podspec-from-above.yaml \
  --namespace "arc-runners"
```

Now, the runners are ready to use! E.g.:

```yaml
name: Actions Runner Controller Demo

on:
  push:

jobs:
  Explore-GitHub-Actions:
    # You need to use the INSTALLATION_NAME from the previous step
    runs-on: linux-arc-runner-set
    container:
      image: ghcr.io/linux-riscv/pw-runner-multi:latest # Change: your image...
    steps:
    - run: echo "🎉 This job uses runner scale set runners!"
```

## Blocking issues

* Occasionally the workflow steps fail to complete: [https://github.com/actions/runner-container-hooks/issues/124](https://github.com/actions/runner-container-hooks/issues/124)

## Conclusion

ARC/GKE/Autopilot seems to be a good fit. There are still some wrinkles, e.g. requiring a RWX volume via the Cloud Filestore API, and the blocking issue above.

## Debugging

TODO:

* Describe k9s application flows
* Describe kubectl flows
