## Introduction to ``wkdev SDK``

NOTE: This documents covers _using_ the SDK. Creating the images is covered in BUILDING.md.

The ``wkdev SDK`` provides a hassle-free environment to perform WebKit Gtk/WPE development.
It is distributed in form of an **OCI image**, a standardized container format that allows
any OCI-compatible container system, such as **Docker** and **podman**, to run the SDK.
The same image can also be used within the WebKit Early Warning System (EWS) to provide
an environment in which tests can be executed in a reliable & reproducible way.

By utilizing the ``wkdev SDK``, a vanilla Linux installation can be turned into a fully
functional WebKit development / debugging environment within minutes. After the initial
setup procedure, the ``wkdev SDK`` user (hereafter: the **developer**) can either directly
run commands within the **wkdev** container or launch one or more interactive shell
sessions, in which you can compile WebKit / run tests / etc.

To run CLI applications within a container, requires no effort: it works out of the box.
Runing graphical applications, that utilize e.g. [Wayland](https://wayland.freedesktop.org)
for screen presentation, need [D-Bus](https://freedesktop.org/wiki/Software/dbus) to communicate
with other system components, or use [SystemD](https://freedesktop.org/wiki/Software/systemd)
APIs to query network / power / etc. information, require a substantial amount of configuration
to allow the containerized GUI application to integrate seamlessly within the host desktop
environment.

To overcome the tedious setup procedure, wrapper tools were created, such as [toolbx](https://containertoolbx.org)
and [distrobox](https://distrobox.privatedns.org), that greatly simplify the setup procedure.

**distrobox** and **toolbx** both allow you to run GUI applications out of the box, the former supports
both **Docker** and **podman** as backends, where **toolbx** is tied to **podman** only. Both also support
to share the current host user and its **\$HOME directory** with the container, replacing any other
\$HOME directory that might reside in the OCI container image. **toolbx** only supports that operation mode,
whereas **distrobox** allows for fine-grained control about what files/directories to share with the container.

However for our specific purposes, we used our own set of wrapper scripts such as `wkdev-create`,
`wkdev-enter`, inspired by **distrobox**, to make it it easy as possible to get stated with WebKit
development.

### Setup procedure

On your **host system** ensure that **podman** is installed.

* [podman](https://podman.io)
  * Fedora: [podman](https://packages.fedoraproject.org/pkgs/podman/podman)
  * Debian (sid): [podman](https://packages.debian.org/sid/podman)
  * Ubuntu (starting from 23.04): [podman](https://packages.ubuntu.com/lunar/podman)
  * macOS: [podman](https://formulae.brew.sh/formula/podman)

That's all you need to install on your host system. Now it's the time to get a fresh WebKit source
checkout, or update/clean an existing one.

```sh
$ cd ~/path/to/home/subdirectory/with/git/checkout/of/
$ git clone https://github.com/WebKit/WebKit.git
```

That can take several hours, depending on your internet connection.
Let it run, and move on to the **Quickstart guide**.

### Quickstart guide

1. Integrate 'wkdev-sdk' with your shell environment.

Add the following to your shell configuration file (e.g. ~/.bashrc, ~/.zprofile, ...)
to ensure that the ${WKDEV_SDK} environment variable points to the correct location
of your 'wkdev-sdk' Git checkout. It also extends the ${PATH} to make the 'wkdev-\*' scripts
provided by this repository accessible without having to specifcy full paths in the shell.

```
# wkdev-sdk integration
pushd /absolute/path/to/your/Git/checkout/of/wkdev-sdk &>/dev/null
source ./register-sdk-on-host.sh
popd &>/dev/null
```

Launch a new shell, or `source` your shell configuration files to verify, ${WKDEV_SDK}
now expects as intented - pointing to your wkdev-sdk checkout.

2. Create a new 'wkdev' container for WebKit development

Execute the following command on your host system:

```wkdev-create --name wkdev --create-home --home ${HOME}/wkdev-home (--verbose)```

This will create a container named 'wkdev', and transparently maps the current hoser user/ID
into the container. Within the container, the ${HOME} directtory is not equal to the host
${HOME} directory: ${HOME}/wkdev-home (from host) is bind-mounted into the container as
"/home/hostuser". This avoids pollution of files in your host ${HOME} directory -- for
convenience it's still exposed in the container, as ${HOST_HOME}.

NOTE: wkdev-create will auto-detect the whole environment: X11, Wayland, PulseAudio, etc.
and eventually needs 'root' permissions on the 'host' system to perform first-time-run-only
initializations (such as allowing GPU profiiling, by modifying root-owned config files, etc.)

3. Enter the new 'wkdev' container

Execute the following command on your host system:

```wkdev-enter --name wkdev```

After a few seconds you enter the container shell.

4. Verify host system integration is working properly

Run the test script in the container, which tests various workloads:

```wkdev-test-host-integration```

5. Compile WPE WebKit

```
cd ${HOST_HOME}/path/to/your/WebKit/checkout
git pull
Tools/Scripts/build-webkit --wpe --release --cmakeargs "-DCMAKE_BUILD_TYPE=RelWithDebInfo -DENABLE_THUNDER=OFF"
```

To run tests / execute MiniBrowser, try;

```
Tools/Scripts/run-webkit-tests --wpe --release fast/css # Full tests take a long time
Tools/Scripts/run-minibrowser --wpe https://browserbench.org/MotionMark1.2/
```

6. READY!
