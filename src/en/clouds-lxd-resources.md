<!--
Todo:
- Test certificate access
-->

# Additional LXD resources

This page offers more in-depth information on LXD itself. To learn how to set
up LXD with Juju see [Using LXD with Juju][clouds-lxd].

The topics presented here are:

 - LXD and images
 - LXD clustering
 - Non-admin user credentials
 - Useful LXD client commands 
 - Using the LXD snap
 - LXD logs
 - Further LXD help and reading

## LXD and images

LXD is image based: all LXD containers come from images and any LXD daemon
instance (also called a "remote") can serve images. When LXD is installed a
locally-running remote is provided (Unix domain socket) and the client is
configured to talk to it (named 'local'). The client is also configured to talk
to several other, non-local, ones (named 'ubuntu', 'ubuntu-daily', and
'images').

An image is identified by its fingerprint (SHA-256 hash), and can be tagged
with multiple aliases.

For any image-related command, an image is specified by its alias or by its
fingerprint. Both are shown in image lists. An image's filename is its *full
fingerprint* while an image list displays its *partial fingerprint*. Either
type of fingerprint can be used to refer to images.

Juju pulls official cloud images from the 'ubuntu' remote
(http://cloud-images.ubuntu.com) and creates the necessary alias. Any
subsequent requests will be satisfied by the LXD cache (`/var/lib/lxd/images`).

Image cache expiration and image synchronization mechanisms are built-in.

## LXD clustering

LXD clustering (v.3 and greater) allows for distributed computing to the extent
that Juju units end up on different cluster nodes (LXD hosts) by default. It
also offers high availability so that the cluster remains functional as long as
more than half of the nodes are up. A downed node will lead to its hosted
containers becoming unavailable.

The following caveats apply:

 - The controller must reside on a cluster node. This will no longer be the
   case starting with v.2.5 of Juju.
 - Each cluster node must have a network bridge that is connected to LXD. This
   is to allow the containers to communicate with the controller.

Clustering is configured by running `sudo lxd init` on each LXD host (a minimum
of three is recommended). The first host that does so will *initialise* the
cluster and any subsequent node will *join* the cluster. Once the cluster is
set up a controller can be created, as normal, on any of the cluster nodes
(e.g. `juju bootstrap localhost lxd-cluster`).

[note=caution]
The cluster-creation process will remove any existing containers. In a Juju
context, this implies that you cannot initialise a cluster *after* having
created a controller.
[/note]

See the upstream documentation on [Clustering][lxd-upstream-clustering].

## Non-admin user credentials

To grant a regular user access to a LXD-based controller a certificate
credential is required. This certificate is generated and shared with the user
who will then use it as a credential. See [Creating users][users-creating] for
details on adding users, registering them, and granting them permissions.

On the LXD host generate the certificate with the `autoload-credentials`
command. Use the below sample session as a guide:

```no-highlight
Looking for cloud and credential information locally...

1. LXD credential "localhost" (new)
Select a credential to save by number, or type Q to quit: 1

Select the cloud it belongs to, or type Q to quit []: localhost

Saved LXD credential "localhost" to cloud localhost

1. LXD credential "localhost" (existing, will overwrite)
Select a credential to save by number, or type Q to quit: Q
```

A certificate credential will have been created. Export it to a file, say
`localhost-credentials.yaml`, by typing the following:

```bash
juju credentials localhost --format=yaml > localhost-credentials.yaml
```

Now transfer the file to the user that requires access to the cloud. Once done,
on that user's system, the credential can be added:

```bash
juju add-credential localhost -f localhost-credentials.yaml
```

See [Cloud credentials][credentials] for more details on how credentials are
used. 

## Useful LXD client commands

There are many client commands available. Some common ones, including those
covered above, are given below.

| client commands                               | meaning                            |
|-----------------------------------------------|------------------------------------|
`lxc launch`					| creates an LXD container
`lxc list`	                             	| lists all LXD containers
`lxc delete`					| deletes an LXD container
`lxc remote list`				| lists remotes
`lxc info`					| displays status of localhost
`lxc info <container>`				| displays status of container
`lxc config show <container>`			| displays configuration of container
`lxc image info <alias or fingerprint>`		| displays status of image
`lxc exec <container> <executable>`		| runs program on container
`lxc exec <container> /bin/bash`		| spawns shell on container
`lxc file pull <container></path/to/file> .`	| copies file from container
`lxc file push </path/to/file> <container>/`  	| copies file to container
`lxc stop <container>`				| stops container
`lxc image list`		                | lists cached images
`lxc image alias delete <alias>`		| deletes image alias
`lxc image alias create <alias> <fingerprint>`	| creates image alias
`lxc cluster list`                              | lists cluster nodes
`lxc cluster show <container>`                  | displays configuration of a cluster node

## Using the LXD snap

The LXD project will soon be moving from the APT installation method (Debian
package) to installing via a snap. Some users may want to opt in early,
*before* building their infrastructure, as moving to the snap entails a
migration of containers. The LXD snap is very well tested (as is the included
migration tool).

First ensure that `snapd` is installed:

```bash
sudo apt install snapd
```

[note=caution]
On Ubuntu 14.04 LTS (Trusty), installing `snapd` will bring in a new kernel
(4.4.0 series) as a dependency. You will then **need to reboot**.
Attempting to install a snap without doing so will result in failure.
[/note]

Now install the LXD snap:

```bash
sudo snap install lxd
```

If LXD is already installed via APT **and there are no existing containers**
under the current installation then simply remove the software:

```bash
sudo apt purge liblxc1 lxcfs lxd lxd-client
```

If containers do exist under the old system the `lxd.migrate` utility should be
used to migrate them to the new system. Once the migration is complete, you
will be prompted to have the old software removed.

Start the migration tool by running:

```bash
sudo lxd.migrate
```

## LXD logs

LXD itself logs to `/var/log/lxd/lxd.log` and Juju machines created via the LXD
local provider log to `/var/log/lxd/juju-UUID-machine-ID`. However, the
standard way to view logs is with the `debug-log` command (see the
[Juju logs][logs] page for details).

[note]
For LXD snap users, the log directory is located at
`/var/snap/lxd/common/lxd/logs`.
[/note]

## Further help and reading

See `lxc --help` for more information on LXD client usage and `lxd --help` for
assistance with the daemon. See upstream documentation for
[LXD configuration][lxd-upstream].


<!-- LINKS -->

[clouds-lxd]: ./clouds-LXD.md
[lxd-upstream]: https://lxd.readthedocs.io/en/latest/configuration/
[lxd-upstream-clustering]: https://lxd.readthedocs.io/en/latest/clustering/
[logs]: ./troubleshooting-logs.md
[credentials]: ./credentials.md
[users-creating]: ./users-creating.md
