# NeSI training environment Containers workshop app

JupyterLab app with [Apptainer](https://apptainer.org/) installed, for running containers workshops
on the NeSI/REANNZ [training environment](https://github.com/nesi/training-environment).

This repo holds both the **JupyterLab** app and the **image that all three Apptainer apps share**:

| Dashboard tile | Repo | Connects to |
| --- | --- | --- |
| JupyterLab | this one | JupyterLab |
| Terminal | [training-environment-apptainer-terminal-app](https://github.com/nesi/training-environment-apptainer-terminal-app) | the full page terminal at `/terminals/1` |
| VS Code | [training-environment-apptainer-codeserver-app](https://github.com/nesi/training-environment-apptainer-codeserver-app) | code-server |

Each Open OnDemand dashboard tile has to be its own git repo, but all three run
`ghcr.io/nesi/training-environment-jupyter-containers-app`, built here. So there is one image to
build and pre-pull, and the other two repos contain only app definitions. When you release a new
version here, update the image tag in all three.

The workshop examples are waiting in `~/containers-workshop`. The terminal page is served by the
`notebook` package, which is why *docker/Dockerfile* installs `notebook` alongside `jupyterlab`, and
`code-server` is installed for the VS Code app.

## How a session is set up

*template/before.sh.erb* sets the connection details, following the same pattern as
[training-environment-jupyter-python-app](https://github.com/nesi/training-environment-jupyter-python-app):
it sources `find_host_port`, `save_passwd_as_secret` and `create_salt_and_sha1` from */bin* (the ood
k8s utils, baked into the image), exports `host`, `port` and `password` for *view.html.erb*, and
writes the JupyterLab config. There are no init containers.

`base_url` is built from the same `HOST_CFG`/`PORT_CFG` that the connect button uses, so the path
the browser requests always matches the path JupyterLab serves.

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
