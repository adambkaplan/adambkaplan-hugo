---
title: "CRC with Fedora Silverblue"
date: 2021-12-30T19:18:00-05:00
draft: false
---

Using CodeReady Containers with Fedora Silverblue and Toolboxes.

----

<!--more-->

I use [Fedora Silverblue](https://silverblue.fedoraproject.org/) as my daily driver at work, and lately I have started using [CodeReady Containers](https://developers.redhat.com/products/codeready-containers/overview) (CRC) for development and testing.
I recently discovered a few tricks to get the `crc` command line working within a toolbox.

This post is an update from the work [@RyanJ](https://twitter.com/ryanj) did in his gist/demo of [CRC on Silverblue](https://gist.github.com/ryanj/01b8592a04e40837ccd07b76716dbe73) earlier this year.

## Setup CRC on Silverblue

To begin with CRC on Silverblue, first install the `libvirt`, `qemu`, and `virt-manager` packages using `rpm-ostree`:

```sh
$ rpm-ostree install libvirt qemu virt-manager
```

After reboot, download CodeReady Containers and place the `crc` binary in your `$PATH`.
I personally prefer to install `crc` in `~/.local/bin`, alongside other binaries that I know need host access:

```sh
$ export CRC_VERSION=1.37.0
$ mkdir -p /tmp/crc && cd /tmp/crc
$ wget https://developers.redhat.com/content-gateway/file/pub/openshift-v4/clients/crc/${CRC_VERSION}/crc-linux-amd64.tar.xz
$ tar -xvf crc-linux-amd64.tar.xz
$ mv crc-linux-${CRC_VERSION}-amd64/crc ~/.local/bin/
$ rm -rf /tmp/crc
```

Finally, run `crc setup` to get your CodeReady Containers cluster initialized.
Note that you may need to grant the setup process `sudo` permissions:

```sh
$ crc setup
```

## Starting and Accessing the Cluster

Once the cluster has been set up, use the start command to launch the control plane, console, and other essential services.
Currently this can only be done on the host - attempting to do this within a toolbox will fail:

```sh
$ crc start
```

You can then use `eval $(crc oc-env)` to access the cluster.

## Using `crc` Inside a Toolbox

Most Silverblue users are familiar with [Toolbox](https://docs.fedoraproject.org/en-US/fedora-silverblue/toolbox/), which allows you to run libraries and applications within its own containerized environment.
The `crc` utility can be used (mostly) inside a toolbox, however setting this up requires a bit more work than just invoking `crc` commands directly.

### Extend the Toolbox Image

To access the cluster inside a toolbox, `crc` requires the `libvirt-libs` package to be present.
Unfortunately this can't be done within a running toolbox container, because toolbox bind mounts many of the libvirt directories if they are present on the host.
To get around this, you can create a toolbox from a container image that extends the base toolbox for the OS you are running.
Below is a `Containerfile` which installs `libvirt-libs` on top of the Fedora 35 toolbox image:

```Dockerfile
FROM registry.fedoraproject.org/fedora-toolbox:35
RUN sudo dnf install -y libvirt-libs
```

Use your container engine of choice to build the extended toolbox image, then create a toolbox from your image:

```sh
$ podman build -t crc-toolbox:35 -f Containerfile .
...
$ toolbox create --container crc-toolbox --image crc-toolbox:35
```

### Run crc

Once the CRC cluster is created and started on the host, you can use most other `crc` commands inside the extended toolbox.
This includes `crc oc-env`, which provides an instance of `oc` pre-configured to connect to the CRC cluster.

```sh
$ toolbox enter --container crc-toolbox
[toolbox]$ eval $(crc oc-env)
```

Note that for this to work, `crc` needs to be in the `$PATH` for both the toolbox and the host.
This is the case for binaries in `~/.local/bin` - toolboxes by default have acess to your home directory plus many other things that you can typically access in a terminal.

## Wrapping it Up

With this setup, I'm able to run CodeReady Containers on my laptop while keeping the host operating system immutable.
Only the `libvirt` related packages need to be added on top of Siverblue's core OS - everything else I need is either in a containerized toolbox or runs as user-local applications.
In a future post, I hope to showcase what else I do with Fedora Silverblue - the good, the bad, and the not so pretty.
