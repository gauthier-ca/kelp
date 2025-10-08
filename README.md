# kelp

A collection of tiny shell scripts to build and deploy apps to k8s clusters using kustomize manifests. Override or extend the default commands as needed.

## Prerequisites
To start you need an app repository containing a Dockerfile and a kustomize manifest. You also need a kubernetes cluster available, e.g. Docker Desktop, and docker, kubectl and GNU make installed.

This tool uses [`yq`](https://github.com/mikefarah/yq). The docker version of is used by default. This ensures the tool runs everywhere but means slower runs. You can set `yq=yq` in your environment to use a faster local yq is you have it installed.

## Installation

To install clone this repository then add it to your path:

```
$ kelp install
export PATH="/path/to/kelp:$PATH"
export yq=yq
```
Install in the current shell with `eval $(kelp install)` or permanently in your `.bashrc` with `kelp install >>~/.bashrc`.

## Usage
In your own app repository, which must contain a a Dockerfile and a kustomization file, run:

```
kelp [command ...]
```

If no command is specified, `kelp` defaults to `build run`.
`kelp` looks up the commands to run under either `.kelp/` in the current directory, or under the `kelp/commands` repository. When a command is found, it is sourced within the `kelp` script. Commands depend on the kelp context and cannot be executed standalone.

The default `kelp` commands run kustomize. By default, `kelp` runs `kustomize` on the `k8/base` manifests. You can set the `KELP_STAGE` variable to the name of a directory under `k8s/overlays` to use that overlay.

If a `.kelp/kelp.yaml` file exists, `kelp` considers the directory a top-level directory. It then looks up subdirectories in the `.resources` field of `kelp.yaml` and invokes commands on each of them. Furthermore, if other yaml files are present under `.kelp/` they are processed in the same way, and in alphabetical order. This is useful to declare static lists of repositories in cases where `kelp.yaml` is generated.

## Commands

The following commands are available:

<dl>
  <dt>apply</dt>
  <dd>Apply the k8s manifest.</dd>
  <dt>build</dt>
  <dd>Build the image.</dd>
  <dt>clean</dt>
  <dd>Delete the image.</dd>
  <dt>config</dt>
  <dd>Dump the config variables.</dd>
  <dt>delete</dt>
  <dd>Delete all resources in the manifest.</dd>
  <dt>down</dt>
  <dd>Delete non persistent resources. Persistent resources are those marked with '.metadata.labels.reclaimPolicy: Retain'.</dd>
  <dt>login</dt>
  <dd>Login to the registry.</dd>
  <dt>push</dt>
  <dd>Push to the registry.</dd>
  <dt>repo</dt>
  <dd>Create the initial ECR repo (for AWS ECR).</dd>
  <dt>test</dt>
  <dd>Run tests.</dd>
  <dt>up</dt>
  <dd>Run build, apply, then test.</dd>
</dl>

Persistent resources are those marked with `.metadata.labels.kelp/reclaimPolicy: Retain`.

## Variables

The following environment variables can be set:

<dl>
  <dt>overlays</dt>
  <dd>Set the path to the kustomize overlays directory. Defaults to `k8s/overlays`.</dd>
  <dt>stage</dt>
  <dd>Specify a specific overlay. Defaults to dev.</dd>
  <dt>yq</dt>
  <dd>Set the `yq` binary. Use this for faster runs if you have it installed.</dd>
</dl>

## Known issues

The targets currently assume a single image is to be built. This image name is looked up the the first Deployment, Job or CronJob object of the manifest.
