# Build CRD

This repository implements `Build` and `BuildTemplate` custom resources
for Kubernetes, and a controller for making them work.

If you are interested in contributing, see [CONTRIBUTING.md](./CONTRIBUTING.md)
and [DEVELOPMENT.md](./DEVELOPMENT.md).

## Objective

Kubernetes is emerging as the predominant (if not de facto) container
orchestration layer.  It is also quickly becoming the foundational layer on top
of which the ecosystem is building higher-level compute abstractions (PaaS,
FaaS).  In order to increase developer velocity, these higher-level compute
abstractions typically operate on source, not just containers, which must be
built.  However, there is no common building block today for bridging these
worlds.

This repository provides an implementation of the Build [CRD](
https://kubernetes.io/docs/concepts/api-extension/custom-resources/) that runs
Builds on-cluster (by default), because that is the lowest common denominator
that we expect users to have available.  It is also possible to write
[pkg/builder](./pkg/builder) that delegates Builds to hosted services (e.g.
Google Container Builder), but these are typically more restrictive.

The aim of this project isn't to be a complete standalone product that folks use
directly (e.g. as a CI/CD replacement), but a building block to facilitate the
expression of Builds as part of larger systems.

## Terminology and Conventions

* [Builds](./builds.md)
* [Build Templates](./build-templates.md)
* [Builders](./builder-contract.md)
* [Authentication](./cmd/creds-init/README.md)

## Getting Started

You can install the latest release of the Build CRD by running:
```shell
kubectl create -f https://storage.googleapis.com/build-crd/latest/release.yaml
```

### Run your first `Build`

Create your first `build.yaml`:

```yaml
apiVersion: build.dev/v1alpha1
kind: Build
metadata:
  name: hello-build
spec:
  steps:
  - name: hello
    image: busybox
    args: ['echo', 'hello', 'build']
```

Run it on your Kubernetes cluster:

```shell
$ kubectl apply -f build.yaml
build "hello-build" created
```

Check that it was created:

```shell
$ kubectl get builds
NAME          AGE
hello-build   4s
```

Get more information about the build:

```shell
$ kubectl get build hello-build -oyaml
apiVersion: build.dev/v1alpha1
kind: Build
...
status:
  builder: Cluster
  cluster:
    namespace: default
    podName: hello-build-jx4ql
  conditions:
  - state: Complete
    status: "True"
  stepStates:
  - terminated:
      reason: Completed
  - terminated:
      reason: Completed
```

This indicates that the build finished successfully, and that it ran on a
pod named `hello-build-jx4ql` -- your build will run on a pod with a
different name. Let's dive into that pod!

```shell
$ kubectl get pod hello-build-jx4ql -oyaml
...lots of stuff!
```

The build's underlying pod also contains the build's logs, in an init
container named after the build step's name. In the case of this example, the
build step's name was `hello`, so the pod will have an init container named
`build-step-hello` which ran the step.

```shell
$ kubectl logs hello-build-jx4ql -c build-step-hello -f
hello build
```

:tada:
