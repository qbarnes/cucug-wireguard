[This repository](https://github.com/qbarnes/cucug-wireguard/)
contains files for a Wireguard (WG) demonstration using
[DigitalOcean](https://www.digitalocean.com/) (DO) for
[CUCUG](http://cucug.org/).

Files:

```
mkdroplets		Makes DigitalOcean droplets
wgdroplets		Deploys wireguard to the DO droplets
wggenfiles		Runs on the hub by wgdroplets to create WG config files
userdata-hub		Cloud-init userdata file when initializing DO hub
userdata-spoke		Cloud-init userdata file when initializing DO spokes
cucug-wireguard.pdf	Presentation given on Feb 17, 2022
README.md		This file
```

To recreate the demonstration, first have a DigitalOcean account,
install the `doctl` utility for your Linux distro, and run `doctl
auth init`.  The scripts below will use the token from your `doctl`
configuration file.  Now run:

    $ ./mkdroplets ubuntu-21-10-x64 debian-11-x64 fedora-35-x64

The first droplet image type is used for the Wireguard "hub".  The
hub is the host that all traffic is routed through to the Internet.
The remaining arguments, if any, are for additional droplets to be
created as "spokes", the peers of the hub.

    $ ./wgdroplets -k -r 3

The `wgdroplets` script configures and deploys Wireguard to the
previously created droplets.

In this example, the `-k` option says to enable using pre-shared
keys between the peers.  Pre-shared keys add an extra layer of
security.  The `-r 3` option says to provision three additional WG
spoke configuration files in the hub's `/etc/wireguard` directory.
These files can be used for configuring other spokes running
elsewhere such as iOS or Android devices or laptops.

To use WG with an iOS or Android device, login to the hub and
display the QR code of the configuration to use with:

    # sed -e '/^#/d' /etc/wireguard/wg0pN.conf | qrencode -t ansiutf8

Replace `N` with the peer number you wish to use.

Now install and open the Wireguard app, tap the '+' sign and
specify loading a configuration via a QR code.  Show your phone's
camera the above displayed QR code.

The `sed` filtering isn't strictly necessary.  It just reduces the
size of the generated QR code.
