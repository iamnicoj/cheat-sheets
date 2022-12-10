# podman

Fom [kubecon2022](https://cloud-native-canada.github.io/k8s_setup_tools/local_cluster/podman/) 

```bash

brew install podman

podman machine init \
--cpus=2 \
--memory=4096 \
--disk-size=200 \
--now

# podman machine start # Not required because of --now option

# if machine is down you can list it and then start it
podman machine list

podman machine start podman-machine-default

# once with a running machie you can pretty much is it like a docker host
podman search busybox

podman run -it docker.io/library/busybox

podman machine stop
```


Make Docker command call Podman, Podman is command-line compatible with Docker:
```bash
mv -f /usr/local/bin/docker /usr/local/bin/docker-orig
ln -s /usr/local/bin/podman /usr/local/bin/docker
```

Point the default Docker socket to the Podman socket. This is needed as some apps use a hardcoded path to Docker:

```bash
# This is needed so every app "hardcoded" for Docker will work
export DOCKER_HOST="unix://$HOME/.local/share/containers/podman/machine/podman-machine-default/podman.sock"

```

After a reboot, `podman` will be disabled. To recover podman and `kind` containers re-run following steps:

``` bash
podman machine start
podman start --all
```
