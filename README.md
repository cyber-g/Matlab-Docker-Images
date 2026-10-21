# Matlab-Docker-Images
Dockerfiles for MATLAB-based images (from `mathworks/matlab` and `mathworks/matlab-deep-learning`) with additional toolbox sets, like Communications Toolbox, and Signal Blocks (DSP System Toolbox).

Docker images based exclusively on **MATLAB R2024a**.

## Images and tags

Images are hosted on the same Docker Hub repository. 
Currently, the Docker Hub repository is: 
[drdpham/matlab-images](https://hub.docker.com/repository/docker/drdpham/matlab-images/)

The tag after `:` selects the variant and revision.

| Dockerfile directory | MathWorks parent image | Docker tag |
| --- | --- | --- |
| `base-comms-signalblocks/` | `mathworks/matlab:r2024a` | `r2024a-base-comms-signalblocks-v1` |
| `deep-learning-comms-signalblocks/` | `mathworks/matlab-deep-learning:r2024a` | `r2024a-dl-comms-signalblocks-v1` |

Starting from the base MATLAB images, these Dockerfiles add the following products:

- `Communications_Toolbox`
- `DSP_System_Toolbox` (`Signal_Blocks` is a license feature, not an MPM product name)
- `Signal_Processing_Toolbox`

Both preserve the parent entrypoint and command and run as user `matlab`.

Tags follow `r2024a-<variant>-v<revision>`. Increase the revision for new
publications to avoid overwriting previous tags. No `latest` tag is published.
MathWorks parent tags can change, so rebuilding may pick up upstream updates.

## Build locally

```sh
docker build -t matlab:r2024a-base-comms-signalblocks-v1 ./base-comms-signalblocks
docker build -t matlab:r2024a-dl-comms-signalblocks-v1 ./deep-learning-comms-signalblocks
```

## Publish to Docker Hub

1. In GitHub **Settings → Secrets and variables → Actions**, configure:
   - Secrets: `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`.
   - Variable: `DOCKERHUB_REPOSITORY`.
2. Once pushed, open **Actions →
   Publish R2024a images to Docker Hub → Run workflow** and enter a positive
   numeric revision, e.g. `1`.

The manual-only workflow builds and pushes both tags sequentially. 
(No Git tag is required, and pull requests do not publish images.)

## Run and verify

A valid MATLAB license covering the products in use is required at runtime. When using CI runners, make sure that the runner has access to the network license server or the appropriate license file. 

Example : pull and run with your authorized network license server information:

```sh
docker pull drdpham/matlab-images:r2024a-base-comms-signalblocks-v1
docker run --rm \
   -v $HOME/network.lic:/licenses/license.lic:ro \
   -e MLM_LICENSE_FILE=/licenses/license.lic \
  drdpham/matlab-images:r2024a-base-comms-signalblocks-v1 \
  -batch "ver; assert(~isempty(ver('comm'))); assert(~isempty(ver('dsp'))); assert(~isempty(ver('signal')));"
```

Execute a script in the current working directory: 

```sh
docker run --rm \
   -v $HOME/network.lic:/licenses/license.lic:ro \
   -e MLM_LICENSE_FILE=/licenses/license.lic \
   -v $(pwd):/work --workdir /work \
  drdpham/matlab-images:r2024a-base-comms-signalblocks-v1 \
  -sd /work \
  -batch script
```

Please note that the `startup.m` file changes the startup directory of matlab. The operation of the `-sd /work` option ensures that MATLAB starts in the `/work` directory inside the container, which corresponds to the current working directory on the host. 

If the script requires to write output files, ensure that the container user matches the host user to avoid permission issues: 

```sh
docker run --rm \
   -v $HOME/network.lic:/licenses/license.lic:ro \
   -e MLM_LICENSE_FILE=/licenses/license.lic \
   -v $(pwd):/work --workdir /work \
   --user root -e USER_NAME=$(whoami) -e USER_UID=$(id -u) -e USER_GID=$(id -g) \
  drdpham/matlab-images:r2024a-base-comms-signalblocks-v1 \
  -sd /work \
  -batch script
```