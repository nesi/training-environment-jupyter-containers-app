# NeSI training environment Containers workshop app

JupyterLab app with [Apptainer](https://apptainer.org/) installed, for running containers workshops
on the NeSI/REANNZ [training environment](https://github.com/nesi/training-environment).

A running session gives learners two buttons:

* **Connect to Terminal** — a full page terminal, which is where Apptainer is actually used
  (`apptainer pull`, `apptainer build`, `apptainer shell`)
* **Connect to JupyterLab** — the usual JupyterLab interface; *File > New > Terminal* gets you the
  same shell

Both connect to the same Jupyter server in the same pod, so files and running work are shared
between them. The workshop examples are waiting in `~/containers-workshop`. The terminal page is served by the `notebook` package at `/terminals/1`, which is why
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

## Workshop examples

The examples come from the [workshop repo](https://github.com/nesi/reannz-containers-workshop),
pinned by commit in *docker/workshop-version.txt*. They are baked into the image at
*/opt/containers-workshop/examples* and rsynced into `~/containers-workshop` at startup with
`--ignore-existing`, so learners' edits survive a restart.

The chapter 9 MPI examples are dropped, since they are written for a Slurm cluster and will not run
here.

The two [chapter 2](https://nesi.github.io/reannz-containers-workshop/setup-containers/#2-the-basics-of-running-containers-on-apptainer)
containers (*hello-world.sif* and *lolcow.sif*) ship prebuilt, so nobody waits for them at the start
of the workshop. They are built by *.github/workflows/build_container.yml* **on the runner**, not in
the docker build, because building a `.sif` needs mount privileges that a `RUN` step does not have.
The workflow drops them into *docker/prebuilt/*, which the Dockerfile moves into the chapter 2
example directory. Building the image by hand leaves that directory empty, which is fine — the
definition files are all still there.

To move to a newer version of the workshop material, change the commit in
*docker/workshop-version.txt* and release a new version of this app. Both the definition files and
the prebuilt images come from that one revision.

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
