# lima-vm-docker

Lima VM Guest with rootfull Docker support.

Tested on:
- macOS 11.6
- Docker Client 20.10.8
- lima 0.6.4

## Installation

First step is: **REMOVE DOCKER DESKTOP IF INSTALLED**

Using brew:

```bash
$ brew install lima docker
[...]
$ mkdir ~/docker
```

The "docker" directory will be accessible from the lima VM

## Configuration

Create a file named "docker.yaml" with [this content](docker.yaml) or clone this repo.

This file describes a VM with this features:
- 4 cpu cores
- 4GiB memory
- 50GiB dynamically allocated disk space
- access to host directories "~/docker" and "/tmp/lima" with read/write permissions
- docker automatic installation

Create and start the VM:

```bash
$ limactl start ~/lima-vm-docker/docker.yaml
? Creating an instance "docker" Proceed with the default configuration
[...]
INFO[0062] READY. Run `limactl shell docker` to open the shell.
```

Check installation completion:

```bash
$ LIMA_INSTANCE=docker lima docker version
Client: Docker Engine - Community
 Version:           20.10.8
 API version:       1.41
 Go version:        go1.16.6
 Git commit:        3967b7d
 Built:             Fri Jul 30 19:53:57 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true
 
Server: Docker Engine - Community
 Engine:
  Version:          20.10.8
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.16.6
  Git commit:       75249d8
  Built:            Fri Jul 30 19:52:06 2021
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.9
  GitCommit:        e25210fe30a0a703442421b0f60afac609f950a3
 runc:
  Version:          1.0.1
  GitCommit:        v1.0.1-0-g4144b63
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

Configure the Docker Client on the Mac to access the Lima Docker TCP socket:

```bash
$ docker context create lima-tcp --description "Connect to Lima-VM" --docker "host=tcp://127.0.0.1:2375"
context "lima-tcp" created
 
$ docker context use lima-tcp
lima-tcp
Current context is now "lima-tcp"
```

Check host configuration:

```bash
$ docker version
Client: Docker Engine - Community
 Version:           20.10.8
 API version:       1.41
 Go version:        go1.16.6
 Git commit:        3967b7d28e
 Built:             Thu Jul 29 13:55:47 2021
 OS/Arch:           darwin/amd64
 Context:           lima-tcp
 Experimental:      true
 
Server: Docker Engine - Community
 Engine:
  Version:          20.10.8
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.16.6
  Git commit:       75249d8
  Built:            Fri Jul 30 19:52:06 2021
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.9
  GitCommit:        e25210fe30a0a703442421b0f60afac609f950a3
 runc:
  Version:          1.0.1
  GitCommit:        v1.0.1-0-g4144b63
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

Check network access to container exposed ports:

```bash
$ docker run --name network-test --rm -d -p 8080:80 nginx:alpine
Unable to find image 'nginx:alpine' locally
alpine: Pulling from library/nginx
[...]
Digest: sha256:686aac2769fd6e7bab67663fd38750c135b72d993d0bb0a942ab02ef647fc9c3
Status: Downloaded newer image for nginx:alpine
15c3b1ee913408895e26f45bb005495b036b2e99c9677bde1c8484fe307e64e3
 
$ curl http://127.0.0.1:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
[...]
 
$ docker stop network-test
network-test
```

Check filesystem access:

```bash
$ echo "This is a test" > ~/docker/test-file
 
$ docker run --rm -it -v ~/docker/test-file:/test-file bash cat /test-file
Unable to find image 'bash:latest' locally
latest: Pulling from library/bash
[...]
Digest: sha256:377439330211c232e90d460bac26829aacd33f407b525f271295e0460a586b52
Status: Downloaded newer image for bash:latest
This is a test
```

## Start and stop

```bash
$ limactl stop docker
INFO[0000] Sending SIGINT to hostagent process 24075
INFO[0000] Waiting for the host agent and the qemu processes to shut down
INFO[0000] [hostagent] Received SIGINT, shutting down the host agent
INFO[0000] [hostagent] Shutting down the host agent
INFO[0000] [hostagent] Unmounting "/tmp/lima"
WARN[0000] [hostagent] connection to the guest agent was closed unexpectedly
INFO[0000] [hostagent] Shutting down QEMU with ACPI
INFO[0000] [hostagent] Sending QMP system_powerdown command
INFO[0001] [hostagent] QEMU has exited
 
$ limactl start docker
INFO[0000] Using the existing instance "docker"
INFO[0000] [hostagent] Starting QEMU (hint: to watch the boot progress, see "~/.lima/docker/serial.log")
INFO[0000] SSH Local Port: 60022
INFO[0000] [hostagent] Waiting for the essential requirement 1 of 4: "ssh"
[...]
INFO[0031] READY. Run `limactl shell docker` to open the shell.
```

## Known Limitations

- Writeable volumes is still experimental
- Privileged ports are not forwarded
- Work on ARM Mac (M1) but is untested

## Security Warning

The socket is unauthenticated and the Docker Engine is running rootfull mode.

## Bonus

You can always enter the lima VM and customize docker daemon configuration using "`limactl shell docker`" command (i.e. to edit the "`/etc/docker/daemon.json`" file).
