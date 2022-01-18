[This repository](https://github.com/qbarnes/cucug-wireguard/) contains
files for a Wireguard demostration for [CUCUG](http://cucug.org/).

Files:

```
0-pkginst		Shell script to install necessary packages
1-prep			Shell script to prepare host
cucug-wireguard.pdf	Presentation given on Jan 20, 2022
README.md		This file
```

To recreate the demonstration, copy the two shell script files locally,
make them executable, and run them:

    # curl -LJO https://raw.githubusercontent.com/qbarnes/cucug-wireguard/master/0-pkginst
    # curl -LJO https://raw.githubusercontent.com/qbarnes/cucug-wireguard/master/1-prep
    # chmod +x 0-pkginst 1-prep
    # ./0-pkginst

The script `0-pkginst` handles installing necessary packages on
newer Fedora and Ubuntu hosts.  It may be adapted later to handle
other Linux operating systems.

When running `1-prep`, add an `-s` to use preshared keys and
a number after a `-p` to specify how many other peers will be
participating.  For example:

    # ./1-prep -s -p 6

To activate the hub, run:

    # systemctl enable --now wg-quick@wg0.service

To install necessary packages, on each peer copy `0-pkginst` to `/tmp` and run:

    # /tmp/0-pkginst

Then for each peer that's a Linux host, copy
`/etc/wireguard/wg0_N.conf` to its `/etc/wireguard` directory
(replacing each `N` with a number for that peer) then run:

    # systemctl enable --now wg-quick@wg0_N.service

If a peer is an iOS or Android device, install the Wireguard app,
open it, and tap the '+' sign to load a configuration via a QR code.
To display the QR code for a peer, run:

    # qrencode -t ansiutf8 -r /etc/wireguard/wg0_N.conf
