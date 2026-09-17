# NeSI training environment Containers workshop app

JupyterLab app with [Apptainer](https://apptainer.org/) installed, for running containers workshops
on the NeSI/REANNZ [training environment](https://github.com/nesi/training-environment).

A running session gives learners two buttons:

* **Connect to Terminal** — a full page terminal, which is where Apptainer is actually used
  (`apptainer pull`, `apptainer build`, `apptainer shell`)
* **Connect to JupyterLab** — the usual JupyterLab interface; *File > New > Terminal* gets you the
  same shell

Both connect to the same Jupyter server in the same pod, so files and running work are shared
between them. The terminal page is served by the `notebook` package at `/terminals/1`, which is why
*docker/Dockerfile* installs `notebook` alongside `jupyterlab`; the *Connect to Terminal* button
passes `next=` to the login form so it lands there directly.

## Requirements in the training environment

`apptainer build --fakeroot` needs two things from the environment:

* `enable_privileged_pods: true` in *vars/ondemand-config.yml* — `apptainer-suid` will not work
  without it
* `/etc/subuid` and `/etc/subgid` from the worker node, bind mounted into the pod by
  *submit.yml.erb*. The training-environment `container-apps/k8s` role populates these for the
  training and trainer users.

## Apptainer settings

*template/script.sh.erb* exports these before starting Jupyter, so terminals inherit them:

* `APPTAINER_CACHEDIR=$HOME/.apptainer/cache` — pulled images persist across sessions (and count
  against the user's home directory)
* `APPTAINER_TMPDIR=/tmp/apptainer-$USER` — builds use the pod's local disk, not the NFS home
  directory, which is slow and can fail for builds

## Examples

*docker/examples/* is baked into the image at */opt/apptainer-examples* and rsynced into
`~/apptainer-examples` at startup with `--ignore-existing`, so learners' edits survive a restart.

## Releasing a new version

1. Update the version in `script.native.container.image` in *submit.yml.erb*, commit it
2. `git tag -a v0.2.0 -m "..."` and `git push --tags`
3. Check the *Actions* tab — *.github/workflows/build_container.yml* builds and pushes the image to
   ghcr.io
4. Update `k8s_container` and `version` for the `containers` entry in *vars/ondemand-config.yml* in
   the training-environment repo

## Building the image locally

```bash
docker build -t training-environment-jupyter-containers-app:dev docker/
```

The build context is the *docker/* directory, which is why *examples/* lives inside it.
