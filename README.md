# Docker container for MakeMKV
[![Docker Image Size](https://img.shields.io/docker/image-size/jlesage/makemkv/latest)](https://hub.docker.com/r/jlesage/makemkv/tags) [![Build Status](https://drone.le-sage.com/api/badges/jlesage/docker-makemkv/status.svg)](https://drone.le-sage.com/jlesage/docker-makemkv) [![GitHub Release](https://img.shields.io/github/release/jlesage/docker-makemkv.svg)](https://github.com/jlesage/docker-makemkv/releases/latest) [![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://paypal.me/JocelynLeSage/0usd)

This is a Docker container for [MakeMKV](http://www.makemkv.com/).

The GUI of the application is accessed through a modern web browser (no installation or configuration needed on the client side) or via any VNC client.

A fully automated mode is also available: insert a DVD or Blu-ray disc into an optical drive and let MakeMKV rips it without any user interaction.

---

[![MakeMKV logo](https://images.weserv.nl/?url=raw.githubusercontent.com/jlesage/docker-templates/master/jlesage/images/makemkv-icon.png&w=200)](http://www.makemkv.com/)[![MakeMKV](https://dummyimage.com/400x110/ffffff/575757&text=MakeMKV)](http://www.makemkv.com/)

MakeMKV is your one-click solution to convert video that you own into free and
patents-unencumbered format that can be played everywhere. MakeMKV is a format
converter, otherwise called "transcoder". It converts the video clips from
proprietary (and usually encrypted) disc into a set of MKV files, preserving
most information but not changing it in any way. The MKV format can store
multiple video/audio tracks with all meta-information and preserve chapters.

---

## Table of Content

   * [Docker container for MakeMKV](#docker-container-for-makemkv)
      * [Table of Content](#table-of-content)
      * [Quick Start](#quick-start)
      * [Usage](#usage)
         * [Environment Variables](#environment-variables)
         * [Data Volumes](#data-volumes)
         * [Ports](#ports)
         * [Changing Parameters of a Running Container](#changing-parameters-of-a-running-container)
      * [Docker Compose File](#docker-compose-file)
      * [Docker Image Update](#docker-image-update)
         * [Synology](#synology)
         * [unRAID](#unraid)
      * [User/Group IDs](#usergroup-ids)
      * [Accessing the GUI](#accessing-the-gui)
      * [Security](#security)
         * [SSVNC](#ssvnc)
         * [Certificates](#certificates)
         * [VNC Password](#vnc-password)
      * [Reverse Proxy](#reverse-proxy)
         * [Routing Based on Hostname](#routing-based-on-hostname)
         * [Routing Based on URL Path](#routing-based-on-url-path)
      * [Shell Access](#shell-access)
      * [Access to Optical Drive(s)](#access-to-optical-drives)
      * [Automatic Disc Ripper](#automatic-disc-ripper)
      * [Troubleshooting](#troubleshooting)
         * [Expired Beta Key](#expired-beta-key)
      * [Support or Contact](#support-or-contact)

## Quick Start

**NOTE**: The Docker command provided in this quick start is given as an example
and parameters should be adjusted to your need.

Launch the MakeMKV docker container with the following command:
```
docker run -d \
    --name=makemkv \
    -p 5800:5800 \
    -v /docker/appdata/makemkv:/config:rw \
    -v $HOME:/storage:ro \
    -v $HOME/MakeMKV/output:/output:rw \
    --device /dev/sr0 \
    --device /dev/sg2 \
    jlesage/makemkv
```

Where:
  - `/docker/appdata/makemkv`: This is where the application stores its configuration, log and any files needing persistency.
  - `$HOME`: This location contains files from your host that need to be accessible by the application.
  - `$HOME/MakeMKV/output`: This is where extracted videos are written.
  - `/dev/sr0`: This is the first Linux device file representing the optical drive.
  - `/dev/sg2`: This is the second Linux device file representing the optical drive.

Browse to `http://your-host-ip:5800` to access the MakeMKV GUI.
Files from the host appear under the `/storage` folder in the container.

## Usage

```
docker run [-d] \
    --name=makemkv \
    [-e <VARIABLE_NAME>=<VALUE>]... \
    [-v <HOST_DIR>:<CONTAINER_DIR>[:PERMISSIONS]]... \
    [-p <HOST_PORT>:<CONTAINER_PORT>]... \
    jlesage/makemkv
```
| Parameter | Description |
|-----------|-------------|
| -d        | Run the container in the background.  If not set, the container runs in the foreground. |
| -e        | Pass an environment variable to the container.  See the [Environment Variables](#environment-variables) section for more details. |
| -v        | Set a volume mapping (allows to share a folder/file between the host and the container).  See the [Data Volumes](#data-volumes) section for more details. |
| -p        | Set a network port mapping (exposes an internal container port to the host).  See the [Ports](#ports) section for more details. |

### Environment Variables

To customize some properties of the container, the following environment
variables can be passed via the `-e` parameter (one for each variable).  Value
of this parameter has the format `<VARIABLE_NAME>=<VALUE>`.

| Variable       | Description                                  | Default |
|----------------|----------------------------------------------|---------|
|`USER_ID`| ID of the user the application runs as.  See [User/Group IDs](#usergroup-ids) to better understand when this should be set. | `1000` |
|`GROUP_ID`| ID of the group the application runs as.  See [User/Group IDs](#usergroup-ids) to better understand when this should be set. | `1000` |
|`SUP_GROUP_IDS`| Comma-separated list of supplementary group IDs of the application. | (unset) |
|`UMASK`| Mask that controls how file permissions are set for newly created files. The value of the mask is in octal notation.  By default, this variable is not set and the default umask of `022` is used, meaning that newly created files are readable by everyone, but only writable by the owner. See the following online umask calculator: http://wintelguy.com/umask-calc.pl | (unset) |
|`TZ`| [TimeZone] of the container.  Timezone can also be set by mapping `/etc/localtime` between the host and the container. | `Etc/UTC` |
|`KEEP_APP_RUNNING`| When set to `1`, the application will be automatically restarted if it crashes or if a user quits it. | `0` |
|`APP_NICENESS`| Priority at which the application should run.  A niceness value of -20 is the highest priority and 19 is the lowest priority.  By default, niceness is not set, meaning that the default niceness of 0 is used.  **NOTE**: A negative niceness (priority increase) requires additional permissions.  In this case, the container should be run with the docker option `--cap-add=SYS_NICE`. | (unset) |
|`CLEAN_TMP_DIR`| When set to `1`, all files in the `/tmp` directory are deleted during the container startup. | `1` |
|`DISPLAY_WIDTH`| Width (in pixels) of the application's window. | `1280` |
|`DISPLAY_HEIGHT`| Height (in pixels) of the application's window. | `768` |
|`SECURE_CONNECTION`| When set to `1`, an encrypted connection is used to access the application's GUI (either via a web browser or VNC client).  See the [Security](#security) section for more details. | `0` |
|`VNC_PASSWORD`| Password needed to connect to the application's GUI.  See the [VNC Password](#vnc-password) section for more details. | (unset) |
|`X11VNC_EXTRA_OPTS`| Extra options to pass to the x11vnc server running in the Docker container.  **WARNING**: For advanced users. Do not use unless you know what you are doing. | (unset) |
|`ENABLE_CJK_FONT`| When set to `1`, open-source computer font `WenQuanYi Zen Hei` is installed.  This font contains a large range of Chinese/Japanese/Korean characters. | `0` |
|`MAKEMKV_KEY`| MakeMKV registration key to use.  The key is written to the configuration file during container startup.  When set to `BETA`, the latest beta key is automatically used.  When set to `UNSET`, no key is automatically written to the configuration file. | `BETA` |
|`AUTO_DISC_RIPPER`| When set to `1`, the automatic disc ripper is enabled. | `0` |
|`AUTO_DISC_RIPPER_MAKEMKV_PROFILE`| Filename of the custom MakeMKV profile the automatic disc ripper should use.  The profile is expected to be found under the `/config` folder of the container, unless an absolute path is specified. | (unset) |
|`AUTO_DISC_RIPPER_EJECT`| When set to `1`, disc is ejected from the drive when ripping is terminated. | `0` |
|`AUTO_DISC_RIPPER_PARALLEL_RIP`| When set to `1`, discs from all available optical drives are ripped in parallel.  Else, each disc from optical drives is ripped one at time. | `0` |
|`AUTO_DISC_RIPPER_INTERVAL`| Interval, in seconds, the automatic disc ripper checks for the presence of a DVD/Blu-ray discs. | `5` |
|`AUTO_DISC_RIPPER_MIN_TITLE_LENGTH`| Titles with a length less than this value are ignored.  Length is in seconds.  By default, no value is set, meaning that value from MakeMKV's configuration file is taken. | (unset) |
|`AUTO_DISC_RIPPER_BD_MODE`| Rip mode of Blu-ray discs.  `mkv` is the default mode, where a set of MKV files are produced.  When set to `backup`, a copy of the (decrypted) file system is created instead. **NOTE**: This applies to Blu-ray discs only.  For DVD discs, MKV files are always produced. | `mkv` |
|`AUTO_DISC_RIPPER_FORCE_UNIQUE_OUTPUT_DIR`| When set to `0`, files are written to `/output/DISC_LABEL/`, where `DISC_LABEL` is the label/name of the disc.  If this directory exists, then files are written to `/output/DISC_LABEL-XXXXXX`, where `XXXXXX` are random readable characters.  When set to `1`, the `/output/DISC_LABEL-XXXXXX` pattern is always used. | `0` |
|`AUTO_DISC_RIPPER_NO_GUI_PROGRESS`| When set to `1`, progress of discs ripped by the automatic disc ripper is not shown in the MakeMKV GUI. | `0` |

### Data Volumes

The following table describes data volumes used by the container.  The mappings
are set via the `-v` parameter.  Each mapping is specified with the following
format: `<HOST_DIR>:<CONTAINER_DIR>[:PERMISSIONS]`.

| Container path  | Permissions | Description |
|-----------------|-------------|-------------|
|`/config`| rw | This is where the application stores its configuration, log and any files needing persistency. |
|`/storage`| ro | This location contains files from your host that need to be accessible by the application. |
|`/output`| rw | This is where extracted videos are written. |

### Ports

Here is the list of ports used by the container.  They can be mapped to the host
via the `-p` parameter (one per port mapping).  Each mapping is defined in the
following format: `<HOST_PORT>:<CONTAINER_PORT>`.  The port number inside the
container cannot be changed, but you are free to use any port on the host side.

| Port | Mapping to host | Description |
|------|-----------------|-------------|
| 5800 | Mandatory | Port used to access the application's GUI via the web interface. |
| 5900 | Optional | Port used to access the application's GUI via the VNC protocol.  Optional if no VNC client is used. |

### Changing Parameters of a Running Container

As can be seen, environment variables, volume and port mappings are all specified
while creating the container.

The following steps describe the method used to add, remove or update
parameter(s) of an existing container.  The general idea is to destroy and
re-create the container:

  1. Stop the container (if it is running):
```
docker stop makemkv
```
  2. Remove the container:
```
docker rm makemkv
```
  3. Create/start the container using the `docker run` command, by adjusting
     parameters as needed.

**NOTE**: Since all application's data is saved under the `/config` container
folder, destroying and re-creating a container is not a problem: nothing is lost
and the application comes back with the same state (as long as the mapping of
the `/config` folder remains the same).

## Docker Compose File

Here is an example of a `docker-compose.yml` file that can be used with
[Docker Compose](https://docs.docker.com/compose/overview/).

Make sure to adjust according to your needs.  Note that only mandatory network
ports are part of the example.

```yaml
version: '3'
services:
  makemkv:
    image: jlesage/makemkv
    ports:
      - "5800:5800"
    volumes:
      - "/docker/appdata/makemkv:/config:rw"
      - "$HOME:/storage:ro"
      - "$HOME/MakeMKV/output:/output:rw"
    devices:
      - "/dev/sr0:/dev/sr0"
      - "/dev/sg2:/dev/sg2"
```

## Docker Image Update

Because features are added, issues are fixed, or simply because a new version
of the containerized application is integrated, the Docker image is regularly
updated.  Different methods can be used to update the Docker image.

The system used to run the container may have a built-in way to update
containers.  If so, this could be your primary way to update Docker images.

An other way is to have the image be automatically updated with [Watchtower].
Watchtower is a container-based solution for automating Docker image updates.
This is a "set and forget" type of solution: once a new image is available,
Watchtower will seamlessly perform the necessary steps to update the container.

Finally, the Docker image can be manually updated with these steps:

  1. Fetch the latest image:
```
docker pull jlesage/makemkv
```
  2. Stop the container:
```
docker stop makemkv
```
  3. Remove the container:
```
docker rm makemkv
```
  4. Create and start the container using the `docker run` command, with the
the same parameters that were used when it was deployed initially.

[Watchtower]: https://github.com/containrrr/watchtower

### Synology

For owners of a Synology NAS, the following steps can be used to update a
container image.

  1.  Open the *Docker* application.
  2.  Click on *Registry* in the left pane.
  3.  In the search bar, type the name of the container (`jlesage/makemkv`).
  4.  Select the image, click *Download* and then choose the `latest` tag.
  5.  Wait for the download to complete.  A  notification will appear once done.
  6.  Click on *Container* in the left pane.
  7.  Select your MakeMKV container.
  8.  Stop it by clicking *Action*->*Stop*.
  9.  Clear the container by clicking *Action*->*Reset* (or *Action*->*Clear* if
      you don't have the latest *Docker* application).  This removes the
      container while keeping its configuration.
  10. Start the container again by clicking *Action*->*Start*. **NOTE**:  The
      container may temporarily disappear from the list while it is re-created.

### unRAID

For unRAID, a container image can be updated by following these steps:

  1. Select the *Docker* tab.
  2. Click the *Check for Updates* button at the bottom of the page.
  3. Click the *update ready* link of the container to be updated.

## User/Group IDs

When using data volumes (`-v` flags), permissions issues can occur between the
host and the container.  For example, the user within the container may not
exist on the host.  This could prevent the host from properly accessing files
and folders on the shared volume.

To avoid any problem, you can specify the user the application should run as.

This is done by passing the user ID and group ID to the container via the
`USER_ID` and `GROUP_ID` environment variables.

To find the right IDs to use, issue the following command on the host, with the
user owning the data volume on the host:

    id <username>

Which gives an output like this one:
```
uid=1000(myuser) gid=1000(myuser) groups=1000(myuser),4(adm),24(cdrom),27(sudo),46(plugdev),113(lpadmin)
```

The value of `uid` (user ID) and `gid` (group ID) are the ones that you should
be given the container.

## Accessing the GUI

Assuming that container's ports are mapped to the same host's ports, the
graphical interface of the application can be accessed via:

  * A web browser:
```
http://<HOST IP ADDR>:5800
```

  * Any VNC client:
```
<HOST IP ADDR>:5900
```

## Security

By default, access to the application's GUI is done over an unencrypted
connection (HTTP or VNC).

Secure connection can be enabled via the `SECURE_CONNECTION` environment
variable.  See the [Environment Variables](#environment-variables) section for
more details on how to set an environment variable.

When enabled, application's GUI is performed over an HTTPs connection when
accessed with a browser.  All HTTP accesses are automatically redirected to
HTTPs.

When using a VNC client, the VNC connection is performed over SSL.  Note that
few VNC clients support this method.  [SSVNC] is one of them.

[SSVNC]: http://www.karlrunge.com/x11vnc/ssvnc.html

### SSVNC

[SSVNC] is a VNC viewer that adds encryption security to VNC connections.

While the Linux version of [SSVNC] works well, the Windows version has some
issues.  At the time of writing, the latest version `1.0.30` is not functional,
as a connection fails with the following error:
```
ReadExact: Socket error while reading
```
However, for your convenience, an unofficial and working version is provided
here:

https://github.com/jlesage/docker-baseimage-gui/raw/master/tools/ssvnc_windows_only-1.0.30-r1.zip

The only difference with the official package is that the bundled version of
`stunnel` has been upgraded to version `5.49`, which fixes the connection
problems.

### Certificates

Here are the certificate files needed by the container.  By default, when they
are missing, self-signed certificates are generated and used.  All files have
PEM encoded, x509 certificates.

| Container Path                  | Purpose                    | Content |
|---------------------------------|----------------------------|---------|
|`/config/certs/vnc-server.pem`   |VNC connection encryption.  |VNC server's private key and certificate, bundled with any root and intermediate certificates.|
|`/config/certs/web-privkey.pem`  |HTTPs connection encryption.|Web server's private key.|
|`/config/certs/web-fullchain.pem`|HTTPs connection encryption.|Web server's certificate, bundled with any root and intermediate certificates.|

**NOTE**: To prevent any certificate validity warnings/errors from the browser
or VNC client, make sure to supply your own valid certificates.

**NOTE**: Certificate files are monitored and relevant daemons are automatically
restarted when changes are detected.

### VNC Password

To restrict access to your application, a password can be specified.  This can
be done via two methods:
  * By using the `VNC_PASSWORD` environment variable.
  * By creating a `.vncpass_clear` file at the root of the `/config` volume.
    This file should contain the password in clear-text.  During the container
    startup, content of the file is obfuscated and moved to `.vncpass`.

The level of security provided by the VNC password depends on two things:
  * The type of communication channel (encrypted/unencrypted).
  * How secure the access to the host is.

When using a VNC password, it is highly desirable to enable the secure
connection to prevent sending the password in clear over an unencrypted channel.

**ATTENTION**: Password is limited to 8 characters.  This limitation comes from
the Remote Framebuffer Protocol [RFC](https://tools.ietf.org/html/rfc6143) (see
section [7.2.2](https://tools.ietf.org/html/rfc6143#section-7.2.2)).  Any
characters beyond the limit are ignored.

## Reverse Proxy

The following sections contain NGINX configurations that need to be added in
order to reverse proxy to this container.

A reverse proxy server can route HTTP requests based on the hostname or the URL
path.

### Routing Based on Hostname

In this scenario, each hostname is routed to a different application/container.

For example, let's say the reverse proxy server is running on the same machine
as this container.  The server would proxy all HTTP requests sent to
`makemkv.domain.tld` to the container at `127.0.0.1:5800`.

Here are the relevant configuration elements that would be added to the NGINX
configuration:

```
map $http_upgrade $connection_upgrade {
	default upgrade;
	''      close;
}

upstream docker-makemkv {
	# If the reverse proxy server is not running on the same machine as the
	# Docker container, use the IP of the Docker host here.
	# Make sure to adjust the port according to how port 5800 of the
	# container has been mapped on the host.
	server 127.0.0.1:5800;
}

server {
	[...]

	server_name makemkv.domain.tld;

	location / {
	        proxy_pass http://docker-makemkv;
	}

	location /websockify {
		proxy_pass http://docker-makemkv;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection $connection_upgrade;
		proxy_read_timeout 86400;
	}
}

```

### Routing Based on URL Path

In this scenario, the hostname is the same, but different URL paths are used to
route to different applications/containers.

For example, let's say the reverse proxy server is running on the same machine
as this container.  The server would proxy all HTTP requests for
`server.domain.tld/makemkv` to the container at `127.0.0.1:5800`.

Here are the relevant configuration elements that would be added to the NGINX
configuration:

```
map $http_upgrade $connection_upgrade {
	default upgrade;
	''      close;
}

upstream docker-makemkv {
	# If the reverse proxy server is not running on the same machine as the
	# Docker container, use the IP of the Docker host here.
	# Make sure to adjust the port according to how port 5800 of the
	# container has been mapped on the host.
	server 127.0.0.1:5800;
}

server {
	[...]

	location = /makemkv {return 301 $scheme://$http_host/makemkv/;}
	location /makemkv/ {
		proxy_pass http://docker-makemkv/;
		location /makemkv/websockify {
			proxy_pass http://docker-makemkv/websockify/;
			proxy_http_version 1.1;
			proxy_set_header Upgrade $http_upgrade;
			proxy_set_header Connection $connection_upgrade;
			proxy_read_timeout 86400;
		}
	}
}

```
## Shell Access

To get shell access to the running container, execute the following command:

```
docker exec -ti CONTAINER sh
```

Where `CONTAINER` is the ID or the name of the container used during its
creation (e.g. `crashplan-pro`).

## Access to Optical Drive(s)

By default, a Docker container doesn't have access to host's devices.  However,
access to one or more devices can be granted with the `--device DEV` parameter.

In the Linux world, an optical drive is represented by two different device
files: `/dev/srX` and `/dev/sgY`, where `X` and `Y` are numbers.

For best performance, it is recommended to expose both these devices to the
container.  For example, for an optical drive represented by `/dev/sr0` and
`/dev/sg1`, the following parameters would be added to the `docker run`
command:
```
--device /dev/sr0 --device /dev/sg1
```

**NOTE**: For an optical drive to be detected by MakeMKV, it is mandatory to
expose `/dev/sgY` to the container.  `/dev/srX` is optional, but performance
could be affected.

The easiest way to determine the right Linux devices to expose is to run the
container (without `--device` parameter) and look at its log: during the
startup, messages similar to these ones are outputed:
```
[cont-init.d] 95-check-optical-drive.sh: executing...
[cont-init.d] 95-check-optical-drive.sh: looking for usable optical drives...
[cont-init.d] 95-check-optical-drive.sh: found optical drive [/dev/sr0, /dev/sg3], but it is not usable because:
[cont-init.d] 95-check-optical-drive.sh:   --> the host device /dev/sr0 is not exposed to the container.
[cont-init.d] 95-check-optical-drive.sh:   --> the host device /dev/sg3 is not exposed to the container.
[cont-init.d] 95-check-optical-drive.sh: no usable optical drive found.
[cont-init.d] 95-check-optical-drive.sh: exited 0.
```

In this case, it's clearly indicated that `/dev/sr0` and `/dev/sg3` needs to be
exposed to the container.

## Automatic Disc Ripper

This container has an automatic disc ripper built-in.  When enabled, any DVD or
Blu-ray video disc inserted into an optical drive is automatically ripped.  In
other words, MakeMKV decrypts and extracts all titles (such as the main movie,
bonus features, etc) from the disc to MKV files.

To enable the automatic disc ripper, set the environment variable
`AUTO_DISC_RIPPER` to `1`.

To eject the disc from the drive when ripping is terminated, set the environment
variable `AUTO_DISC_RIPPER_EJECT` to `1`.

If multiple drives are available, discs can be ripped simultaneously by
setting the environment variable `AUTO_DISC_RIPPER_PARALLEL_RIP` to `1`.

See the [Environment Variables](#environment-variables) section for details
about setting environment variables.

**NOTE**: All titles, audio tracks, chapters, subtitles, etc are
        extracted/preserved.

**NOTE**: Titles and audio tracks are kept in their original format.  They are
        not transcoded or converted to other formats or into smaller sizes.

**NOTE**: Ripped Blu-ray discs can take a large amount of disc space (~40GB).

**NOTE**: MKV Files are written to the `/output` folder of the container.

**NOTE**: The automatic disc ripper processes all available optical drives.

## Troubleshooting

### Expired Beta Key

If the beta key is expired, just restart the container to automatically fetch
and install the latest one.

**NOTE**: Once beta key expires, it can take few days before a new key is made
available by the author of MakeMKV.  During this time, the application is not
functional.

**NOTE**: For this solution to work, the `MAKEMKV_KEY` environment variable must
be set to `BETA`.  See the [Environment Variables](#environment-variables)
section for more details.

[TimeZone]: http://en.wikipedia.org/wiki/List_of_tz_database_time_zones

## Support or Contact

Having troubles with the container or have questions?  Please
[create a new issue].

For other great Dockerized applications, see https://jlesage.github.io/docker-apps.

[create a new issue]: https://github.com/jlesage/docker-makemkv/issues
