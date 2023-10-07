---
title: "podman 学习之路"
date: 2023-02-11T15:46:08+08:00
draft: false
---

# podman

官网：<https://podman.io/>

安装 podman

```bash
➜ brew install podman


➜ podman machine init
Downloading VM image: fedora-coreos-37.20230303.2.0-qemu.aarch64.qcow2.xz: done
Extracting compressed file
Image resized.
Machine init complete
To start your machine run:

 podman machine start


~ took 2m 38.2s
➜ podman machine start


➜ podman -v
podman version 4.4.2
```

- The `podman ps` command is used to list created and running containers.
- podman ps命令用于列出已创建和正在运行的容器。

**Note**: If you add `-a` to the `podman ps` command, Podman will show all containers (created, exited, running, etc.).

```bash
➜ podman ps
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
```

Podman desktop: <https://podman-desktop.io/docs/Installation>

```bash
~
➜ brew install podman-desktop
fatal: not in a git directory
Warning: No remote 'origin' in /opt/homebrew/Library/Taps/homebrew/homebrew-services, skipping update!
==> Downloading https://github.com/containers/podman-desktop/releases/download/v0.12.0/podman-desktop-0.
==> Downloading from https://objects.githubusercontent.com/github-production-release-asset-2e65be/465844
######################################################################## 100.0%No remot
All formula dependencies satisfied.
==> Installing Cask podman-desktop
==> Moving App 'Podman Desktop.app' to '/Applications/Podman Desktop.app'
🍺  podman-desktop was successfully installed!

~ took 19m 3.6s
➜ No remotNo remot



➜ podman run quay.io/podman/hello
Trying to pull quay.io/podman/hello:latest...
Getting image source signatures
Copying blob sha256:265ed7ff70caa449cf77b1d47bfe235eea5808fc0c9cbd23e3eaa367a0fc8295
Copying config sha256:6b603ca3a0c699dab172c04031854c7601ef210a5660fa3452b93c8d57d8945d
Writing manifest to image destination
Storing signatures
!... Hello Podman World ...!

         .--"--.
       / -     - \
      / (O)   (O) \
   ~~~| -=(,Y,)=- |
    .---. /`  \   |~~
 ~/  o  o \~~~~.----. ~~
  | =(X)= |~  / (O (O) \
   ~~~~~~~  ~| =(Y_)=-  |
  ~~~~    ~~~|   U      |~~

Project:   https://github.com/containers/podman
Website:   https://podman.io
Documents: https://docs.podman.io
Twitter:   @Podman_io

```
