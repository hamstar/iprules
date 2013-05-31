IPrules
---

Simple rules management for people who are scared of IP Tables

## What is this?

This package is a set of scripts to manage iptables rules.  It is enspired by the method apache2 uses to manage sites on Debian systems.

That is, one folder with available rules, that get symlinked into a folder of enabled rules.

## What is required?

Root access and iptables.

## How does it work?

In `/etc/iptables/rules-available` you have all the available rules with names you can understand and remember easily.  You then run the `ipenrule` command with the rule name as an argument and it will symlink the rule from the `rules-available` folder into `/etc/iptables/rules-enabled`.  You then need to run `iprules reload` to build the iptables script (in `/etc/iptables.rules`) and load it into iptables.  You can use `ipdisrule` in the same manner to disable a rule.

It also uses a policy file (found in `/etc/iptables/policy.rules`) to drop or accept by default (you likely want the former).

## How to install

I have built a Debian package you can get from [here (v1.1)](http://hamstar.github.io/iprules/downloads/iprules_1.1.deb).

Otherwise you will have to clone this repo and install it manually. It's pretty straight forward, just copy all the folders (except for DEBIAN) to `/` and then run the `DEBIAN/postinst` script.

## Examples

Here is how you would give access to your webserver and allow it to be pinged:

```sh
ipenrule http-in
ipenrule ping-in
iprules reload
```

Piece of pie.

Drop everything except for incoming SSH packets:

```sh
ipenrule ssh-in
ippol drop
iprules reload
```

Easy as cake.

You will probably want to do this by default:

```sh
ipenrule loopback http-out https-out dns-out ping-out ssh-out ssh-in
ippol drop
iprules reload
```

You can view the iptables rules before you reload them:

```sh
iprules show
```

Disable access to your webserver:

```sh
ipdisrule http-in https-in
iprules reload
```

I have ommitted the script output for brevity above but it will let you know stuff:

```sh
$ ipenrule http-out hsdsd ssh-in dns-out
Must be root.
$ sudo ipenrule http-out hsdsd ssh-in dns-out
http-out rules enabled
ERROR: No such rule called: hsdsd
ssh-in rules enabled
dns-out rules enabled
Remember to run 'iprules reload' to activate the configuration.
$ sudo ippol drop
WARNING: be sure remote access is allowed (if needed) before reloading
Remember to run 'iprules reload' to activate the configuration.
$ sudo iprules reload
Rebuilt rules file.
Reloaded rules.
```

Check out what rules are available...

```sh
$ sudo iprules av[ail]
dns-out
http-in
http-out
loopback
synflood-protect
```

And whats enabled...

```sh
$ sudo iprules en[abled]
dns-out
http-out
loopback
```

## What rules come with it?

You can see the list of rules in the [share folder in the source](https://github.com/hamstar/iprules/blob/master/usr/share/iprules/rules/).  If you have ideas for new ones, or see errors in the existing ones submit a patch or pull request and I will add them in.

As above, you can also run `iprules avail` to see what rules are installed.

## Can I write my own rules?

You most certainly can.  IPrules makes it easy to manage your iptables rules... if you know the iptables syntax... but you know how to use google right?

Just make your own file in `/etc/iptables/rules-available` (as root) and then you can use `ipenrule` and `ipdisrule` on it.  If you change it when it's already enabled, simply run `iprules reload` again.

If you make an error in the syntax, iptables won't accept it and will fail to reload.

If you put comments in the file, they will be printed out when the rule is enabled:

```sh
$ ipenrule synflood-protect

Notes from synflood-protect rules:
* need to set net.ipv4.tcp_syncookies=1 in /etc/sysctl.conf
* need to set net.netfilter.nf_conntrack_tcp_timeout_syn_recv=30 in /etc/sysctl.conf

synflood-protect rules enabled
Remember to run 'iprules reload' to activate the configuration.
```

## Enable the rules on boot

If you have the package **iptables-persistent** installed on Debian, it will already do this.  RPM based distro's should do this out of the box but may use the file `/etc/sysconfig/iptables` instead.  So delete that file and make a symlink to the rules file (`ln -s /etc/iptables.rules /etc/sysconfig/iptables`.

If neither of these is the case, you can just add this line to `/etc/rc.local`:

```sh
`which iptables-restore` < /etc/iptables.rules;
```

## WARNING

Be very careful using the default drop policy (`ippol drop`) with remote systems.  If you have not allowed SSH in then you will lock yourself out!

## Todo

* add a `/etc/iptables/envvars` file so you can specify variables to use in the rule files.
* specify priorities in the enable script e.g. `ipenrule last-rule:999`
* add more rules!
* some kind of port forwarding command maybe...?
