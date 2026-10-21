# Matlab-Docker-Images
Dockerfiles for MATLAB-based images (from `mathworks/matlab` and `mathworks/matlab-deep-learning`) with additional toolbox sets, like Communications, and Signal Blocks.

Docker images based exclusively on **MATLAB R2024a**.

## Images and tags

Both variants share one Docker Hub repository, for example `yourname/matlab`.
The tag after `:` selects the variant and revision.

| Build directory | MathWorks parent image | Docker tag |
| --- | --- | --- |
| `base-comms-signalblocks/` | `mathworks/matlab:r2024a` | `r2024a-base-comms-signalblocks-v1` |
| `deep-learning-comms-signalblocks/` | `mathworks/matlab-deep-learning:r2024a` | `r2024a-dl-comms-signalblocks-v1` |

Both install these products into the existing MATLAB installation using
[MATLAB Package Manager (MPM)](https://github.com/mathworks-ref-arch/matlab-dockerfile/tree/main/alternates/building-on-matlab-docker-image):

- `Communications_Toolbox`
- `DSP_System_Toolbox` (`Signal_Blocks` is a license feature, not an MPM product name)
- `Signal_Processing_Toolbox`

The deep learning variant also retains all products from its parent image.
Both preserve the parent entrypoint and command and run as user `matlab`.

Tags follow `r2024a-<variant>-v<revision>`. Increase the revision for new
publications to avoid overwriting previous tags. No `latest` tag is published.
MathWorks parent tags can change, so rebuilding may pick up upstream updates.

## Build locally

From the repository root, replace `yourname/matlab` with your Docker Hub repository:

```sh
docker build -t yourname/matlab:r2024a-base-comms-signalblocks-v1 ./base-comms-signalblocks
docker build -t yourname/matlab:r2024a-dl-comms-signalblocks-v1 ./deep-learning-comms-signalblocks
```

## Publish to Docker Hub

1. Create a Docker Hub repository and an access token with write permission.
2. In GitHub **Settings → Secrets and variables → Actions**, configure:
   - Secrets: `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`.
   - Variable: `DOCKERHUB_REPOSITORY`, e.g. `yourname/matlab` (no tag or URL prefix).
3. Once the workflow is on the default branch, open **Actions →
   Publish R2024a images to Docker Hub → Run workflow** and enter a positive
   numeric revision, e.g. `1`.

The manual-only workflow builds and pushes both tags sequentially. No Git tag is
required, and pull requests do not publish images. MATLAB images are large:
allow sufficient disk space and use a larger runner if needed.

## Run and verify

A valid MATLAB license covering the products you use is required at runtime.
Check MathWorks redistribution terms before publishing, especially to a public
repository. Never include licenses, tokens or credentials in images or build arguments.

Pull a specific tag and run with your authorized network license server:

```sh
docker pull yourname/matlab:r2024a-dl-comms-signalblocks-v1
docker run --rm \
  -e MLM_LICENSE_FILE=27000@license-server \
  yourname/matlab:r2024a-dl-comms-signalblocks-v1 \
  -batch "assert(strcmp(version('-release'),'2024a')); ver; assert(~isempty(ver('comm'))); assert(~isempty(ver('dsp'))); assert(~isempty(ver('signal')))"
```

This checks the release and installed products, not license availability.
