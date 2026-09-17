# Apptainer examples

These files are copied into `~/apptainer-examples` when the app starts. Existing files are never
overwritten, so anything you change here is kept when you restart the session.

Open a terminal — either the *Connect to Terminal* button on the session card, or
*File > New > Terminal* inside JupyterLab — and try:

```bash
cd ~/apptainer-examples

# check the version
apptainer --version

# pull an existing image from a registry
apptainer pull lolcow.sif docker://ghcr.io/apptainer/lolcow

# run it
apptainer run lolcow.sif

# run a single command inside it
apptainer exec lolcow.sif cowsay "kia ora"

# get an interactive shell inside it
apptainer shell lolcow.sif
```

Building your own image from a definition file:

```bash
# have a look at the definition file first
cat lolcow.def

# build it (needs --fakeroot in this environment)
apptainer build --fakeroot lolcow-custom.sif lolcow.def

# run it
apptainer run lolcow-custom.sif
```

Notes:

* Images pulled from registries are cached in `~/.apptainer/cache`, so they survive a session
  restart. Run `apptainer cache clean` if you run low on space in your home directory.
* Builds use `/tmp` inside the pod (`$APPTAINER_TMPDIR`) rather than your NFS home directory,
  because building on NFS is slow and can fail.
* Your home directory is bind mounted into the container by default, so `$HOME` inside a container
  is the same `$HOME` you see outside it.
