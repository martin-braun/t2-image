# Vanilla OS MacBook T2 (2016-2019) Image

Containerfile for building a Vanilla OS Desktop T2 image.

This image is based on top of [`vanillaos/desktop`](https://github.com/Vanilla-OS/desktop-image/pkgs/container/desktop) and offers the necessary [kernel patches, drivers and tweaks](https://wiki.t2linux.org/distributions/debian/installation/#adding-t2-support) for any MacBook with a T2 chip.

## Development

### Preparation

- Install `go` via `brew install go`
- Install `vib` (`git clone git@github.com:Vanilla-OS/Vib.git`, or `vs add git@github.com:Vanilla-OS/Vib.git -b=v0.3.3` with `https://gist.github.com/martin-braun/e10b7cafdd55feea44bc2d06d5c2c972#file-vs` and `alias vib='"$(vs dir github.com/Vanilla-OS/Vib -b=v0.3.3)/vib"'`)
- Build `vib` by `cd Vib` or `vs sh github.com/Vanilla-OS/Vib/v0.3.3` and then `go build && chmod +x vib`
- Install `podman` via `brew install podman`
- Run `podman machine init` to initialize the host VM for `podman`

### Build

This build step is not necessary to use the image, but it is useful to verify the image can be built. Comment the `fsguard` section in `recipe.yml` and then:

```bash
podman machine start; vib build recipe.yml && podman image build -t vanilla-os/t2 . && podman save -o vanilla-os-t2-desktop.tar localhost/vanilla-os/t2
```

Don't forget to uncomment the `fsguard` section in `recipe.yml` again.

> Note: You can track the image via `podman inspect localhost/vanilla-os/t2`.
> You can also clean up via `podman rmi localhost/vanilla-os/t2 && podman rmi 0179950f34c7`

## Usage

### Preparation

This image cannot contain any proprietary driver software for your MacBook, so a few manual steps are required:

- Make sure your Bluetooth and WiFi firmware is available on your EFI partition: https://wiki.t2linux.org/guides/wifi-bluetooth/#on-macos

### Install

- Use balenaEtcher to setup an USB drive with the VanillaOS tar image
- Boot into recovery, disable SIP and mount your USB drive
- Install and boot into VanillaOS
- Connect to the internet using Ethernet/USB tethering/external Wi-Fi adapter
- Run `host-shell pkexec vim /etc/abroot/abroot.json` in your local installation:
- Change the "name" entry from something like `vanilla-os/desktop` to `martin-braun/t2-image`.
<!-- - Remove the differ URL line. -->
- Run `abroot upgrade` to switch to this image.
- Perform upgrade via `pkexec abroot upgrade`
- Finalize your setup by installing your Bluetooth and WiFi firmware under VanillaOS: https://wiki.t2linux.org/guides/wifi-bluetooth/#on-linux

## Development hints

- If we want to install .deb files, we can just put them in `includes.container/deb-pkgs`
- Optionally, we can add out modules to the `modules` directory and add them to the package-modules `includes` in `recipe.yml`.
- We can check the Actions tab in GitHub to see the build progress of our image.

> [!NOTE]
> It is suggested to add a `vib-image` tag to your repository for your image to be easily discoverable.

## Project structure

- `.github/workflows/vib-build.yml`: This file contains the GitHub Actions workflow to check for updates to the base image and build the Vib image.
  - It uses the `vib` action to build the recipe and upload it as an artifact. The generated artifact is then built using Docker and pushed to GHCR.
  - The action runs automatically on a schedule checking updates to the base image using [Differ](https://github.com/Vanilla-OS/Differ).
- `.github/workflows/vib-pr.yml`: This file contains the GitHub Actions workflow to build the Vib image on pull requests.
  - It uses the `vib` action to build the recipe and upload it as an artifact. You can download and view the artifact to verify that your changes are performed as expected.
- `.github/dependabot.yml`: This file contains the configuration for GitHub's Dependabot to check for updates to the GitHub actions used in the workflow files monthly and when it finds a new version it creates a PR in your repository.
- `includes.container`: The files included in this directory are added by default to your image to the specified location. (It also contains ABRoot's configuration file)
- `modules`: This directory contains the modules that are used to customize the image. You can add your modules to this directory.
- `recipe.yml`: This file contains the recipe for the image. It specifies the base image, modules and other fields to be present in the custom image.
