# Introduction

[This repository](https://github.com/qbarnes/cucug-wireguard/)
contains files for a Wireguard (WG) demonstration using
[DigitalOcean](https://www.digitalocean.com/) (DO) for
[CUCUG](http://cucug.org/).

**This is a completely functional demo and may be used for configuring
a live working Wireguard system for personal use.**

To recreate this demonstration for yourself, you'll need to have a
DigitalOcean account.  If you've never had a DigitalOcean account,
you can set up one for free and the resources used by this demo
will be free for 60 days.

Please keep in mind that by far the most challenging part of this
demo is setting up your DigitalOcean account and the host used for
bootstrapping.  That's about 95% of the work.  To get the Wireguard
network running with your peers' participating, it's just running a
few simple commands.

## Files

```
mkdroplets		Makes DigitalOcean droplets
wgdroplets		Deploys Wireguard to the DO droplets
wggenfiles		Runs on the hub by wgdroplets to create WG config files
userdata-hub		Cloud-init userdata file when initializing DO hub
userdata-spoke		Cloud-init userdata file when initializing DO spokes
cucug-wireguard.pdf	Presentation given on Feb 17, 2022
README.md		This file
```

# Preparing a host to bootstrap the Wireguard network

This Linux host used for bootstrapping can be an existing Linux
system of your own or you can create a temporary DigitalOcean
droplet just long enough to run the commands.  The bootstrap host
is used for creating the Wireguard configuration files and to start
network.

Once you have a Linux host for bootstrapping, you'll need to install
the `doctl` utility on it.  If your bootstrap host is Fedora, to
install `doctl`, you can run:

    $ sudo dnf install -y doctl

If on Ubuntu, run:

    $ sudo snap install doctl

If one of these commands doesn't work for you, you may need to check DO's
[doctl documentation](https://docs.digitalocean.com/reference/doctl/how-to/install/).

Once installed, run:

    $ doctl auth init

The command will prompt for your DO token.  To set up or find your
existing DO account token, see
[your API token tab](https://cloud.digitalocean.com/account/api/tokens).

Make sure when the command completes that you see `Validating
token... OK` to ensure your token was accepted.

The droplet scripts below will use the token from your saved `doctl`
configuration file.

Now you're done invoking `doctl` directly.  However, if you'd like
to know more about it, see the
[doctl documentation](https://docs.digitalocean.com/reference/doctl/reference/).

You will also want to set up on your DO account an SSH key for
authentication with droplets.  See
[adding SSH keys to droplets](https://docs.digitalocean.com/products/droplets/how-to/add-ssh-keys/)
for further information.

Before proceeding, you'll need the files from this git repo
installed on the bootstrap host.  The most straight-forward way is
to clone the repo with `git`.  In case `git` isn't installed on your
host already, run either `sudo dnf install -y git` or `sudo apt-get
update && sudo apt install -y git`, depending on your distro.  If
these don't work, see your distro's documentation for installing
packages.

You may clone the repo with:

    $ git clone https://github.com/qbarnes/cucug-wireguard.git

For running the commands below, enter the repo's directory with `cd
cucug-wireguard`.

# Creating and configuring the DO hosts for Wireguard

The `mkdroplets` command takes a list of OS image names ("slugs")
for the droplets it will create.  You can use `doctl compute
image list-distribution` command to list possible image names.

The first image name given to `mkdroplets` will be used as the
Wireguard "hub".  The hub is the host that all traffic is routed
through to the Internet.  Its existence is mandatory.  The remaining
arguments, if any, are for additional droplets to act as WG "spokes"
or additional "peers".

Before running `mkdroplets`, you may need to install the `jq`
package with either `sudo dnf install -y jq` or `sudo apt-get update
&& apt install -y jq`, depending on your distro.

With `jq` command installed, as an example, if you want to set up
a Wireguard hub running Ubuntu 20.10 with two spokes, one running
Debian 11 and the other Fedora 35, you could run:

    $ ./mkdroplets ubuntu-21-10-x64 debian-11-x64 fedora-35-x64

If you have no need any Wireguard spokes running in DO data centers,
you could specify just the hub's OS image name (in this case,
"ubuntu-21-10-x64") with no following arguments.

If you created spokes on DO that you no longer want, you may destroy
their droplets at any time.

The `wgdroplets` script configures and deploys Wireguard to the
previously created droplet(s) and can create additional Wireguard
configuration files for local spokes.  For example, you could run:

    $ ./wgdroplets -k -r 3

Running this command with the `-k` option says to enable using
pre-shared keys between the peers.  Pre-shared keys add an extra
layer of security.

The `-r 3` option says to provision three more WG spoke
configuration files in the hub's `/etc/wireguard` directory in
addition to the DO spokes (if any).  If you had specified 2
additional spokes to `mkdroplets` and ran `wgdroplets` with `-r 3`,
there would be 6 peer configuration files created, one for the hub
and five (2+3) for the spokes.

These additional WG configuration files can be used for configuring
other spokes running elsewhere such as on iOS or Android devices,
laptops or desktops running MacOS or MS Windows, etc.

You're now all done with the bootstrap host!  If it is a DO droplet
you created earlier, you may now destroy it.  Make sure to only
destroy the bootstrap droplet and leave untouched the other droplets
created by `mkdroplets`.

If you used the `-r` option with `wgdroplets`, you should note and
save which peer number you intend to assign and use with local host.
If you make a mistake and confuse which host uses which peer's
configuration, it would be most unfortunate.

# Using Wireguard on a local computer, iOS, or Android device

To use WG with an iOS or Android device with a camera, login to the
hub host with:

    $ doctl compute ssh root@wg-p1

To display the QR code for the peer configuration you wish to use,
run:

    # sed -e '/^#/d' /etc/wireguard/wg0pN.conf | qrencode -t ansiutf8

Replace the `N` in the command with the peer number you assigned for
that host.  Make sure to assign and track a unique peer number for
each host.

Now install and open the Wireguard app, tap the '+' sign and
specify loading a configuration via a QR code.  Show your phone's
camera the above displayed QR code.

The `sed` filtering isn't strictly necessary.  It just reduces the
size of the generated QR code.

For other local hosts, repeat the above process to generate the
QR code for that peer's configuration.  If a device or computer
doesn't have a camera, use the `cat` command to show the contents
of each peer's `.conf` file then copy-and-paste the values into
the Wireguard app's configuration settings on each device.

You're all set to go!  If you open https://whatismyipaddress.com/
on one of your Wireguard peers, on the page's map you should see
on your location in New York City.
