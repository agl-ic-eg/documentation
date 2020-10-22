---
title: Downloading AGL Software
---

Once you have determined the build host can build an AGL image,
you need to download the AGL source files.
The AGL source files, which includes the Yocto Project layers, are
maintained on the AGL Gerrit server.
For information on how to create accounts for Gerrit, see the
[Getting Started with AGL](https://wiki.automotivelinux.org/start/getting-started)
wiki page.

**NOTE:** Further information about Download and Build AGL Source Code available at [AGL wiki](https://wiki.automotivelinux.org/agl-distro/source-code).

The remainder of this section provides steps on how to download the AGL source files:

1. **Define Your Top-Level Directory:**
   You can define an environment variable as your top-level AGL workspace folder.
   Following is an example that defines the `$HOME/workspace_agl` folder using
   an environment variable named "AGL_TOP":

      ```sh
      $ export AGL_TOP=$HOME/AGL
      $ echo 'export AGL_TOP=$HOME/AGL' >> $HOME/.bashrc
      $ mkdir -p $AGL_TOP
      ```

2. **Download the `repo` Tool and Set Permissions:**
   AGL Uses the `repo` tool for managing repositories.
   Use the following commands to download the tool and then set its
   permissions to allow for execution:

      ```sh
      $ mkdir -p $HOME/bin
      $ export PATH=$HOME/bin:$PATH
      $ echo 'export PATH=$HOME/bin:$PATH' >> $HOME/.bashrc
      $ curl https://storage.googleapis.com/git-repo-downloads/repo > $HOME/bin/repo
      $ chmod a+x $HOME/bin/repo
      ```

   **NOTE:** See the
   "[Repo Command Reference](https://source.android.com/setup/develop/repo)"
   for more information on the `repo` tool.

3. **Download the AGL Source Files:**
   Depending on your development goals, you can either download the
   latest stable AGL release branch, or the "cutting-edge" (i.e. "master"
   branch) files.

   * **Stable Release:**
     Using the latest stable release gives you a solid snapshot of the
     latest know release.
     The release is static, tested, and known to work.
     To download the latest stable release branch (i.e. Jellyfish), use
     the following commands:

     ```sh
     $ cd $AGL_TOP
     $ mkdir jellyfish
     $ cd jellyfish
     $ repo init -b jellyfish -u https://gerrit.automotivelinux.org/gerrit/AGL/AGL-repo
     $ repo sync
     ```

   * **Cutting-Edge Files:**
     Using the "cutting-edge" AGL files gives you a snapshot of the
     "master" branch.
     The resulting local repository you download is dynamic and changes frequently depending on community contributions.
     The advantage of using "cutting-edge" AGL files is that you have the
     absolute latest features, which are often under development, for AGL.

     To download the "cutting-edge" AGL files, use the following commands:

     ```sh
     $ cd $AGL_TOP
     $ mkdir master
     $ cd master
     $ repo init -u https://gerrit.automotivelinux.org/gerrit/AGL/AGL-repo
     $ repo sync
     ```

   Once you `sync` the repository, you have the AGL files in the form of
   "layers" (e.g. `meta-*` folders).
   You also have the `poky` repository in your AGL workspace.

   Listing out the resulting directory structure appears as follows:

   ```sh
   $ tree -L 1
    .
    ├── bsp
    ├── external
    ├── meta-agl
    ├── meta-agl-cluster-demo
    ├── meta-agl-demo
    ├── meta-agl-devel
    ├── meta-agl-extra
    └── meta-agl-telematics-demo
   ```

