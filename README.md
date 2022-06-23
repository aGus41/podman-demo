# Podman demo
- A container is a set of one or more processes that are isolated from the rest of the system.
- Containers provide many of the same benefits as virtual machines, such as security, storage, and network isolation. Containers require far fewer hardware resources and are quick to start and terminate. They also isolate the libraries and the runtime resources (such as CPU and storage) for an application to minimize the impact of any OS update to the host OS.
## Fetching Container Images with Podman
Users can use the
search subcommand to find available images from remote or local registries.

 `$ podman search python`

After finding the image, you can use the pull subcommand to fetch the image and save it locally 
 
 
 `$ podman pull registry.access.redhat.com/ubi8/python-36`

 You can list them with the images subcommand

 `$ podman images`

## Running Containers

The podman run command uses all parameters after the image name as the entry point
command for the container. The following example starts a container from a Red Hat Universal
Base Image. It sets the entry point for this container to the echo "Hello world" command

`$ podman run ubi8/ubi:8.3 echo 'Hello world!'`
        https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image-

To start a container image as a background process, pass the -d option to the podman run
command:

`$ podman run -d -p 44389:8080 registry.redhat.io/rhel8/httpd-24`

`$ podman port -l`

It uses the -p 44389:8080 option to bind HTTP server's port to a local port. Then, it uses the podman port command to
retrieve the local port on which the container listens

`$ curl http://0.0.0.0:44389` 

Most Podman subcommands accept the -l flag (l for latest) as a replacement for
the container id. This flag applies the command to the latest used container in any
Podman command

If the images require that the user interact with the console, then Podman can redirect container
input and output streams to the console. The run subcommand requires the -t and -i flags to enable interactivity.

`$ podman run -it ubi8/ubi:8.3 /bin/bash`

Some containers need or can use external parameters provided at startup. The most common
approach for providing and consuming those parameters is through environment variables.
	
Podman can inject environment variables into containers at startup by adding the -e flag to the
	run subcommand.

`$ podman run -e GREET=Hello -e NAME=RedHat  ubi8/ubi:8.3 printenv GREET NAME`

## Evolution of Container Usage
If you have been running containers for some time, chances are that you are running them as a
privileged user. Historically, tools for container creation required that runtime engines run as root,
and privileged access was necessary to create resources, such as network interfaces.
From a security perspective, providing this level of access is a bad practice. You should always run
software with privileges that are as limited as possible. When a security bug is exploited, either on
the runtime engine or the application itself, the impact is minimized.

### Understanding Rootless Containers
#### User Namespaces
Containers use Linux namespaces to isolate themselves from the host on which they run. In
particular, the User namespace is used to make containers rootless. This namespace maps
user and group IDs so that a process inside the namespace might appear to be running under a
different ID.
Rootless containers use the User namespace to make application code appear to be running as
root. However, from the host's perspective, permissions are limited to those of a regular user.
If an attacker manages to escape the user namespace onto the host, then it will have only the
capabilities of a regular, unprivileged user.
#### Networking
To allow proper networking inside a container, a Virtual Ethernet device is created. This poses a
problem for rootless containers because only a real root user has the privileges to create this and
similar devices.
On a rootless container, networking is usually managed by Slirp. It works by forking into the
container's user and network namespaces and creating a tap device that becomes the default
route. Then, it passes the device's file descriptor to the parent, who runs in the default network
namespace and can now communicate with both the container and the internet.
#### Storage
By default, container engines use a special driver called Overlay2 (or Overlay) to create a layered
file system that is efficient in both capacity and performance. This cannot be done with rootless
containers, because most Linux distributions do not allow mounting overlay file systems in user
namespaces.
For rootless containers, the solution is to create a new storage driver. The FUSE-OverlayFS is a
user-space implementation of Overlay, which is more efficient than the VFS storage driver used
before and can run inside user namespaces.


Run a container as the root user and check the UID of a running process inside it


`$ sudo podman run --rm --name asroot -ti registry.access.redhat.com/ubi8:latest /bin/bash`

- `--rm`: Remove after exit

Start a sleep process inside the container

`$ sleep 1000`


On another terminal, we can run `ps` to search for the process

`$ sudo ps -ef | grep "sleep 1000"`

Start another container from the Red Hat UBI 8 as a regular user.

`$ podman run --rm --name asuser -ti registry.access.redhat.com/ubi8:latest /bin/bash`

## Containerfile

```
FROM ubi8/ubi:8.3
LABEL mantainer=Agus email=aalgorta@redhat.com
LABEL description="A custom Apache container based on UBI 8"
RUN yum install -y httpd && \
yum clean all
RUN echo "Hello from Containerfile" > /var/www/html/index.html
EXPOSE 80
CMD ["httpd", "-D", "FOREGROUND"]
```

`$ podman build -t httpd/apache:2.3 -f <containerfile>`

- `-t: tagged name to apply to the built image`

- Alternatively, you can tag the image later with: 

`$ podman tag <containerID> httpd/apache:2.3`

`$ podman images`

`$ podman run --name lab-apache -d -p 10080:80 httpd/apache`

`$ podman ps`

`$ curl 127.0.0.1:10080`

We can push the image to a specified destination

`$ podman push quay.io/aalgorta/httpd-test`




## Tools to troubleshoot containers

https://www.redhat.com/sysadmin/container-namespaces-nsenter

·runc, nsenter, lsns, buildah

![alt text](https://github.com/aGus41/podman-demo/blob/main/containers.png?raw=true)
- oom: Out of Memory
