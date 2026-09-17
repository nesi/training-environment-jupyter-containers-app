# REANNZ training environment JupyterHub containers app

JupyterLab app with [Apptainer](https://apptainer.org/) installed, for containers workshops on the
NeSI/REANNZ [training environment](https://github.com/nesi/training-environment).

It is a copy of
[training-environment-jupyter-python-app](https://github.com/nesi/training-environment-jupyter-python-app)
with these changes:

1. *docker/Dockerfile* installs `apptainer-suid` from `ppa:apptainer/ppa` (plus
   `software-properties-common`, which `add-apt-repository` needs, and `uidmap`) in place of the
   intro-python notebooks and data
2. *submit.yml.erb* bind mounts `/etc/subuid` and `/etc/subgid` from the worker node, which
   `apptainer build --fakeroot` needs
3. the app name, icon and connect button

Everything else — *template/before.sh.erb*, the configmap, the mounts, the workflow — is the python
app's, unchanged, because that app is known to work in this environment.

## Requirements in the training environment

`apptainer build --fakeroot` needs two things:

* `enable_privileged_pods: true` in *vars/ondemand-config.yml* — `apptainer-suid` will not work
  without it
* the `/etc/subuid` and `/etc/subgid` mounts above. The training-environment `container-apps/k8s`
  role populates those files on the worker nodes for the training and trainer users.

## Workshop material

The examples from the [workshop repo](https://github.com/nesi/reannz-containers-workshop) are baked
into the image at */opt/containers-workshop/examples*, pinned by commit in
*docker/workshop-version.txt*, and rsynced into `~/containers-workshop` at startup with
`--ignore-existing`, so learners' edits survive a session restart. The chapter 9 MPI examples are
left out, since they are written for a Slurm cluster.

The two [chapter 2](https://nesi.github.io/reannz-containers-workshop/setup-containers/#2-the-basics-of-running-containers-on-apptainer)
containers ship prebuilt, so nobody waits for them at the start of the workshop:

```bash
cd ~/containers-workshop/examples/02_basics_of_containers
apptainer run hello-world.sif
apptainer run lolcow.sif
```

They are built by *.github/workflows/build_container.yml* **on the runner**, not in the docker
build, because building a `.sif` needs mount privileges that a `RUN` step does not have. The
workflow drops them into *docker/prebuilt/*, which the Dockerfile moves into the chapter 2 example
directory. Building the image by hand leaves that directory empty, which is fine — the definition
files are all still there.

To move to a newer version of the workshop material, change the commit in
*docker/workshop-version.txt*; the definition files and the prebuilt containers both come from it.

## Releasing a new version

1. Update the version in `script.native.container.image` in *submit.yml.erb*, commit it
2. `git tag -a v0.3.1 -m "..."` and `git push --tags`
3. Check the *Actions* tab — *.github/workflows/build_container.yml* builds and pushes the image
4. Update `k8s_container` and `version` for this app in *vars/ondemand-config.yml* in the
   training-environment repo
