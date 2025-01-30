#Podman + Docker-compose + Apple Silicon GPU (MPS) Support

Commands to get up & running with Podman + docker-compose + GPU on Macbook pro (Apple silicon)

## Context
This is compiled from multiple sources (cited below) and many hours of googling.

### Why not DMG installer?
I did start with podman's mac installer (.dmg). It did not work for me as expected. 
Firstly, the positives about the dmg:
1. It gives you a graphical option to choose VM type (applehv or libkrun).
2. Lays out nicely what's running and what extensions are there.

Beyond that, dmg installer seemed completely disconnected from its CLI counterpart. Some commands worked through graphical interface, some through CLI, some on none. So much so, that even after uninstalling podman, the CLI command was still very much functional. I did a bunch of manual jugglary to remove and clean up my system. If you're unlucky enough to be in the same situation, refer to appendix for safe removal or podman. Therefore, on with the homebrew route.

### Steps
For GPU support, install krunkit first

```
brew tap slp/krunkit
brew install krunkit
brew install podman
```

Then, use this to point podman to a non-default VM type:
```
export CONTAINERS_MACHINE_PROVIDER="libkrun"
```

Then with the usual podman setup commands
```
podman machine init --cpus 8 --memory 16384 --disk-size 64 --username <you>
podman machine start
```

At this point (if all is well), you should have a VM running on mac with GPU support. To check, use this command to SSH into the running machine
```
podman machine ssh
```

Then run this command in the terminal that opened
```
ls /dev/dri
```

If its not empty, you're set.

Time to setup docker compose (Docker compose needs to be installed, obviously). Doesn't need full-fledged docker running. The command below will do (since `podman compose` is just a wrapper on top of docker-compose command).
```
brew install docker-compose
```

Use the following command to note URI and Identity columns 
```
podman system connection ls
```

It'll put out something like this:

```
Name                         URI                                                         Identity                                                    Default     ReadWrite
podman-machine-default       ssh://core@127.0.0.1:49241/run/user/501/podman/podman.sock  /Users/you/.local/share/containers/podman/machine/machine  true        true
podman-machine-default-root  ssh://root@127.0.0.1:49241/run/podman/podman.sock           /Users/you/.local/share/containers/podman/machine/machine  false       true
```

From this info, compose this command:
```
ssh -fnNT -L/tmp/podman.sock:/run/user/501/podman/podman.sock -i /Users/you/.local/share/containers/podman/machine/machine ssh://core@127.0.0.1:49241 -o StreamLocalBindUnlink=yes
export DOCKER_HOST='unix:///tmp/podman.sock'
```

Unset SSH auth flag (I had to after much googling, and its temporary so might as well)
```
unset SSH_AUTH_SOCK
```

Run podman compose (at long last!)
```
podman compose -f docker-compose.yml up <your_service>
```

Should work.

### To the giants, thank you! 
This document stands on the shouders of great research by following gentlemen:
1. Andrea Kunar: https://medium.com/@andreask_75652/gpu-accelerated-containers-for-m1-m2-m3-macs-237556e5fe0b
2. Sergio Lopez: https://sinrega.org/2024-03-06-enabling-containers-gpu-macos/
3. Kasper Aaquist: https://gist.github.com/kaaquist/dab64aeb52a815b935b11c86202761a3

### Appendix 1

Complete removal of podman:
```
sudo find / -name podman 2>/dev/null 
sudo /opt/podman/bin/podman-mac-helper uninstall 
sudo rm -f /etc/paths.d/podman-pkg
sudo rm -rfv /opt/podman
sudo rm -rf /private/var/folders/5h/d1l6vlzn1rj53p1pglygcv6w0000gn/T/podman /Users/you/.config/containers/podman /Users/you/.local/share/containers/podman
sudo rm -rf /usr/local/podman
```
