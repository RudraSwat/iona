# Iona

An image builder for Ubuntu remixes (currently supports building ISOs, squashfses and tarballs).

## Requirements

You must run Debian, Ubuntu, or a derivative of either distribution. Alternatively, you could install Ubuntu on WSL2 if you're on Windows by following the [guide here](https://canonical-ubuntu-wsl.readthedocs-hosted.com/en/latest/guides/install-ubuntu-wsl2/).

Install the required packages by running the following commands:

```bash
sudo apt update
sudo apt install mmdebstrap systemd-container arch-install-scripts cd-boot-images-amd64
```

## Usage

```sh
sudo ./iona <PATH_TO_YAML_FILE>
```

## Configuration format

Taking the example of an installable Ubuntu 24.04 remix with GNOME, `openssh-server`, and `nginx`:

```yaml
name: 'Ubuntu Sample' # Name of remix

codename: 'noble' # Ubuntu release codename

repos: # Repositories
    - url: 'http://archive.ubuntu.com/ubuntu' # Repository URL
      components: # Repository components
        - 'main'
        - 'universe'
        - 'multiverse'
        - 'restricted'
      suites: # Repository suits
        - 'noble'
        - 'noble-updates'
      deb-src: false # true if this were a deb-src repo entry (for pulling source packages with APT)

    - url: 'http://security.ubuntu.com/ubuntu'
      components:
        - 'main'
        - 'universe'
        - 'multiverse'
        - 'restricted'
      suites:
        - 'noble-security'
      deb-src: false

pre-workarounds:
    - name: 'command' # to run commands before the packages in the next section are installed
      command: 'apt-get update; apt-get install -y wget; wget -q https://packages.mozilla.org/apt/repo-signing-key.gpg -O- | sudo tee /etc/apt/keyrings/packages.mozilla.org.asc'

    - name: 'command'
      command: 'echo "deb [signed-by=/etc/apt/keyrings/packages.mozilla.org.asc] https://packages.mozilla.org/apt mozilla main" | sudo tee -a /etc/apt/sources.list.d/mozilla.list'

    - name: 'command'
      command: 'echo -e "Package: *\nPin: origin packages.mozilla.org\nPin-Priority: 1000" | sudo tee /etc/apt/preferences.d/mozilla'

packages:
    - type: 'deb'
      list: # list of packages
        - 'gnome-core'
        - 'nginx'
        - 'openssh-server'
        - 'ubiquity-casper' # installer package (Ubiquity)
        - 'ubiquity-frontend-gtk' # installer package (Ubiquity)
        - 'ubiquity-slideshow-ubuntu' # installer package (Ubiquity)

post-workarounds: # similar to pre-workarounds, you can add commands here too (like in pre-workarounds)
    - name: 'ubuntu-network' # to fix certain networking issues
    - name: 'ubiquity-remove' # to remove Ubiquity (installer) packages after the system is installed

artifacts:
    - type: 'iso' # replace with 'squashfs' to generate a squashfs or 'tarball' for a tarball
      name: 'ubuntu-gnome.iso' # name of ISO/squashfs/tarball
```
