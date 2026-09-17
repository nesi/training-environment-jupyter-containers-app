# REANNZ training environment JupyterHub containers app

JupyterLab app with [Apptainer](https://apptainer.org/) installed, for containers workshops on the
NeSI/REANNZ [training environment](https://github.com/nesi/training-environment).

It is a copy of
[training-environment-jupyter-python-app](https://github.com/nesi/training-environment-jupyter-python-app)
with three changes, and nothing else:

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

This image deliberately ships no workshop content. The examples live in the
[workshop repo](https://github.com/nesi/reannz-containers-workshop) and learners can clone them:

```bash
git clone https://github.com/nesi/reannz-containers-workshop.git
cd reannz-containers-workshop/examples/02_basics_of_containers
apptainer build --fakeroot hello-world.sif hello-world.def
```

## Releasing a new version

1. Update the version in `script.native.container.image` in *submit.yml.erb*, commit it
2. `git tag -a v0.3.0 -m "..."` and `git push --tags`
3. Check the *Actions* tab — *.github/workflows/build_container.yml* builds and pushes the image
4. Update `k8s_container` and `version` for this app in *vars/ondemand-config.yml* in the
   training-environment repo
