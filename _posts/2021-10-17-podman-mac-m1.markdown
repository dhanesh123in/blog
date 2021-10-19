---
layout: post
title:  "Installing podman on Mac M1"
author: Dhanesh Padmanabhan
date:   2021-10-17
tags: podman apple-silicon M1
---
If you have a Mac M1 and have been struggling to get `podman`, a popular alternative to `docker-desktop` working, this post might be useful to you.

Here are the steps needed:
1. Ensure you have set the architecture on your machine to `arm64`. You can do that using a command `env /usr/bin/arch -arm64 /bin/zsh --login`. Type `arch` again to ensure you get `arm64`
1. Ensure you have brew configured to work for `arm64`. You can refer the section "Alternative Installs" under [homebrew's documentation] [brew-install]. I would recommend using `/opt` as the base folder to clone the homebrew git repo and follow rest of the process
1. You can install podman and its dependencies with the command 

```bash
brew install simnalamburt/x/podman-apple-silicon 
```

After this test your installation with the following commands

```bash
podman machine init
podman machine start
podman run -it --rm docker.io/hello-world
```

If your installation was successful, you should see "Hello from Docker!" followed by some other information.

[brew-install]: https://docs.brew.sh/Installation
